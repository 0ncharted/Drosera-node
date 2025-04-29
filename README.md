# Drosera Testnet Node & Trap Setup Guide

> **This guide is a streamlined and complete setup instruction based on a fork of [Moei's guide](https://github.com/0xmoei/Drosera-Network).**

## ✅ System Requirements
- **2 CPU Cores**
- **4 GB RAM**
- **20 GB Disk Space**

👉 Suggested VPS: You can start with a $5/month VPS
👉 RPC Requirement: Create a Holesky Ethereum RPC using [Alchemy](https://alchemy.com/) or [QuickNode](https://www.quicknode.com/)

---

## 1. Install Dependencies
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano \
  automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev \
  tar clang bsdmainutils ncdu unzip -y
```

---

## 2. Install Docker
```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Test Docker
sudo docker run hello-world
```

---

## 3. Install Required CLIs
```bash
# Drosera CLI
curl -L https://app.drosera.io/install | bash
source ~/.bashrc
droseraup

# Foundry CLI
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup

# Bun
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc
```

---

## 4. Deploy Contract & Trap
```bash
mkdir my-drosera-trap && cd my-drosera-trap

# Replace below with your GitHub credentials
git config --global user.email "Github_Email"
git config --global user.name "Github_Username"

# Initialize trap
forge init -t drosera-network/trap-foundry-template
bun install
forge build

# Deploy trap (Replace xxx with your private key)
DROSERA_PRIVATE_KEY=xxx drosera apply
# Use --eth-rpc-url if you face RPC errors:
DROSERA_PRIVATE_KEY=xxx drosera apply --eth-rpc-url YOUR_RPC_URL
```

---

## 5. Dashboard & Bloom Boost
1. Go to: https://app.drosera.io
2. Connect your wallet and view traps under "Traps Owned"
3. Click your trap, press **"Send Bloom Boost"**, and deposit Holesky ETH.

---

## 6. Fetch Blocks (Optional)
```bash
drosera dryrun
```

---

## 7. Whitelist Operator
```bash
cd my-drosera-trap
nano drosera.toml

# Add to the bottom:
private_trap = true
whitelist = ["YOUR_OPERATOR_ADDRESS"]

# Save and reapply config:
DROSERA_PRIVATE_KEY=xxx drosera apply
# Or with RPC:
DROSERA_PRIVATE_KEY=xxx drosera apply --eth-rpc-url YOUR_RPC_URL
```

---

## 8. Operator CLI Setup
```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.16.2/drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-operator-*.tar.gz
sudo cp drosera-operator /usr/bin
drosera-operator --version
```

---

## 9. Pull Operator Docker Image
```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
```

---

## 10. Register Operator
```bash
drosera-operator register --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key YOUR_OPERATOR_PRIVATE_KEY
```

---

## 11. Open Ports
```bash
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
sudo ufw enable
```

---

## 12. Run Operator (Choose One Method)

### Method 1: Docker
```bash
git clone https://github.com/0xmoei/Drosera-Network
cd Drosera-Network
cp .env.example .env
nano .env     # Add your private key and VPS IP
nano docker-compose.yaml    # Add your private Holesky RPC

docker compose up -d

# Check logs
docker logs -f drosera-node
```

### Method 2: SystemD
```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node --db-file-path \$HOME/.drosera.db \
  --network-p2p-port 31313 --server-port 31314 \
  --eth-rpc-url YOUR_RPC \
  --eth-backup-rpc-url https://1rpc.io/holesky \
  --drosera-address YOUR_TRAP_ADDRESS \
  --eth-private-key YOUR_PRIVATE_KEY \
  --listen-address 0.0.0.0 \
  --network-external-p2p-address YOUR_VPS_IP \
  --disable-dnr-confirmation true

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl start drosera

# Monitor
journalctl -u drosera.service -f
```

---

## 13. Opt-In Operator to Trap
- Visit dashboard
- Open your trap
- Click **"Opt-in"**

---

## 14. Node Liveness
- Check for **green blocks** in the dashboard

---

## 15. Run a 2nd Operator (Docker)
1. Stop SystemD operator:
```bash
sudo systemctl stop drosera
sudo systemctl disable drosera
```
2. Create new EVM wallet, get Holesky ETH
3. Update whitelist:
```bash
cd ~/my-drosera-trap
nano drosera.toml
# Add both operator addresses:
whitelist = ["Operator1_Address", "Operator2_Address"]
DROSERA_PRIVATE_KEY=xxx drosera apply
```
4. Register 2nd operator:
```bash
drosera-operator register --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key 2nd_Operator_Privatekey
```
5. Open extra ports:
```bash
sudo ufw allow 31315/tcp
sudo ufw allow 31316/tcp
```
6. Edit docker-compose.yaml for 2nd operator:
```yaml
version: '3'
services:
  drosera2:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node2
    ports:
      - "31315:31313"
      - "31316:31314"
    volumes:
      - drosera_data2:/data
    command: node --db-file-path /data/drosera2.db --network-p2p-port 31313 --server-port 31314 \
      --eth-rpc-url YOUR_RPC --drosera-address YOUR_TRAP_ADDRESS \
      --eth-private-key 2nd_Operator_Privatekey --listen-address 0.0.0.0 \
      --network-external-p2p-address YOUR_VPS_IP --disable-dnr-confirmation true

volumes:
  drosera_data2:
```
7. Run:
```bash
docker compose up -d
```

---

## ✅ You're Done
- Visit https://app.drosera.io
- Verify both operators show under your trap
- Monitor health & continue exploring Drosera

---

For updates, troubleshooting or deeper integrations, refer to [Drosera Docs](https://app.drosera.io/) or the original [Moei Guide](https://github.com/0xmoei/Drosera-Network).

