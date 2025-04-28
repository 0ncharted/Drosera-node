# Drosera-node
A protocol that automates live smart contract audit, by monitoring state changes, unique approach to on-chain security: Reward potential

---
# 🛠️ Complete Drosera Operator VPS Setup Guide

## 🔥 Step 0: Prerequisites
- Ubuntu 22.04 (or newer) VPS.
- You have **basic terminal skills** and **root** or **sudo** access.
- You have (or will generate) a **new ECDSA private key** funded with Holesky ETH.
- You have an RPC URL for Holesky (e.g., from [publicnode.com](https://ethereum-holesky-rpc.publicnode.com)).

---

## 1. Install System Dependencies
```bash
sudo apt update
sudo apt install -y curl clang libssl-dev tar ufw
```

✅ Done installing system packages.

---

## 2. Install `droseraup` Tool
```bash
curl -L https://app.drosera.io/install | bash
```

✅ When it finishes, **run this to activate it**:
```bash
source ~/.bashrc
```

(If you forget this step, `droseraup` won't work.)

---

## 3. Install Drosera Operator CLI
```bash
droseraup
```

✅ This downloads the **drosera** and **drosera-operator** binaries to `~/.drosera/bin/`.

(You can also install a specific version like `droseraup -v v1.16.2`, but default is fine.)

---

## 4. Register Your Operator Node
- **Replace** `YOUR_ETH_PRIVATE_KEY_HERE` with your actual private key (without `0x` prefix).
- **Example command**:

```bash
drosera-operator register \
  --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com \
  --eth-private-key YOUR_ETH_PRIVATE_KEY_HERE
```

✅ Registration complete.

(‼️ If it fails, make sure your Holesky ETH balance is sufficient.)

---

## 5. Create the Database Directory
```bash
sudo mkdir -p /var/lib/drosera-data
sudo chown -R root:root /var/lib/drosera-data
sudo chmod -R 700 /var/lib/drosera-data
```

✅ This is where Drosera will store persistent data.

---

## 6. Set Up Systemd Service for Auto-Start

**Create the service file:**
```bash
sudo nano /etc/systemd/system/drosera-operator.service
```

**Paste this config inside:**
```ini
[Unit]
Description=Drosera Operator Node
After=network.target

[Service]
User=root
Restart=always
RestartSec=5
Environment="DRO_DB_FILE_PATH=/var/lib/drosera-data/db.sqlite"
Environment="DRO__DROSERA_ADDRESS=0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8"
Environment="DRO__LISTEN_ADDRESS=0.0.0.0"
Environment="DRO__ETH__CHAIN_ID=17000"
Environment="DRO__ETH__RPC_URL=https://ethereum-holesky-rpc.publicnode.com"
Environment="DRO__ETH__PRIVATE_KEY=YOUR_ETH_PRIVATE_KEY_HERE"
Environment="DRO__NETWORK__P2P_PORT=32325"
ExecStart=/root/.drosera/bin/drosera-operator node

[Install]
WantedBy=multi-user.target
```

✅ Save and exit (`CTRL+O`, `ENTER`, `CTRL+X`).

Now reload systemd and enable the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera-operator
sudo systemctl start drosera-operator
```

✅ Your node should now be running!

**To check logs:**
```bash
sudo journalctl -u drosera-operator -f
```

---

## 7. Configure the UFW Firewall (Optional but Recommended)
```bash
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 32325/tcp # P2P port
sudo ufw allow 32324/tcp # HTTP liveness port
sudo ufw enable
```

✅ Firewall configured.

---

# 🧩 OPTIONAL: Set Up Delegation Client (Highly Recommended)

If you are whitelisted for automatic Trap delegation:

---

## 8. Install Drosera Delegation Client
```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.0.2/drosera-delegation-client-v1.0.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-delegation-client-v1.0.2-x86_64-unknown-linux-gnu.tar.gz
```

✅ Delegation client ready.

---

## 9. Run the Delegation Client

**Start it with:**
```bash
./drosera-delegation-client \
  --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com \
  --eth-private-key YOUR_ETH_PRIVATE_KEY_HERE \
  --delegation-server-url https://delegation-server.testnet.drosera.io
```

✅ It will automatically opt you into Traps.

---

# 🎯 Quick Full Command Summary
```bash
# Install dependencies
sudo apt update && sudo apt install -y curl clang libssl-dev tar ufw

# Install droseraup
curl -L https://app.drosera.io/install | bash
source ~/.bashrc

# Install drosera binaries
droseraup

# Register your operator
drosera-operator register --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key YOUR_ETH_PRIVATE_KEY_HERE

# Create DB directory
sudo mkdir -p /var/lib/drosera-data
sudo chown -R root:root /var/lib/drosera-data
sudo chmod -R 700 /var/lib/drosera-data

# Create and configure systemd service
sudo nano /etc/systemd/system/drosera-operator.service
# (Paste systemd config from above)

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable drosera-operator
sudo systemctl start drosera-operator

# Install and run Delegation Client
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.0.2/drosera-delegation-client-v1.0.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-delegation-client-v1.0.2-x86_64-unknown-linux-gnu.tar.gz
./drosera-delegation-client --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key YOUR_ETH_PRIVATE_KEY_HERE --delegation-server-url https://delegation-server.testnet.drosera.io
```

---

# ✅ VPS Drosera Operator Setup — DONE

If you follow this exactly, you **should not hit any errors**.
If anything fails during the service run, just check:
```bash
sudo journalctl -u drosera-operator -f
```

---

