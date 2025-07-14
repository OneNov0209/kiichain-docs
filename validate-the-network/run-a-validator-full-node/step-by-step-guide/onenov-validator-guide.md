<!-- kiichain Logo -->
<p align="center">
  <img src="https://pbs.twimg.com/profile_images/1800553180083666944/zZe128CW.jpg" alt="kiichain Logo" width="150"/>
</p>

# OneNov Kiichain Validator Guide

**Complete guide to deploying and managing a high-performance Kiichain validator node with enterprise-grade reliability.**

> ![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange)
> ![Node Status](https://img.shields.io/badge/Node%20Status-Active-brightgreen)

---

## 🌐 Useful Links

- 🔍 [All Validators](https://explorer.onenov.xyz/kiichain-test/staking/)
- 🧭 [Explorer](https://explorer.onenov.xyz/kiichain-testnet)
- 📘 [Official Docs](https://docs.kiiglobal.io/docs/validate-the-network/run-a-validator-full-node/step-by-step-guide)
- 🛠️ [API Access](https://lcd.uno.sentry.testnet.v3.kiivalidator.com/)
- 🛠️ [RPC Access](https://rpc.uno.sentry.testnet.v3.kiivalidator.com/)

---

## 📦 System Requirements

| Component | Minimum        | Recommended     |
|----------|----------------|-----------------|
| OS       | Ubuntu 20.04   | Ubuntu 22.04 LTS|
| CPU      | 4 Cores        | 8+ Cores        |
| RAM      | 8GB            | 16GB+           |
| Storage  | 500GB SSD/NVMe | 1TB SSD/NVMe    |
| Network  | 10Mbps         | 100Mbps+        |

---

## 🛠️ Manual Installation

### Step 1: Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```
### Step 2: Install Go
```
rm -rf $HOME/go
sudo rm -rf /usr/local/go
curl https://dl.google.com/go/go1.23.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```
### Step 3: Install Kiichain Node
```
cd $HOME
rm -rf kiichain
git clone https://github.com/KiiChain/kiichain.git
cd kiichain
make install
kiichaind version
```
### Step 4: Initialize Node
```
kiichaind init YOUR_NODE_NAME --chain-id oro_1336-1
```
### Step 5: Configure Peers
```
PEERS="5b6aa55124c0fd28e47d7da091a69973964a9fe1@uno.sentry.testnet.v3.kiivalidator.com:26656,5e6b283c8879e8d1b0866bda20949f9886aff967@dos.sentry.testnet.v3.kiivalidator.com:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kiichain/config/config.toml
```
# Set minimum gas price
```
sed -i 's|^minimum-gas-prices =.*|minimum-gas-prices = "1000000000akii"|' $HOME/.kiichain/config/app.toml
```
---
### 🪙 Wallet & Validator
**Create Wallet**
```
kiichaind keys add wallet --keyring-backend test --coin-type 118 --key-type secp256k1
```
**Recover Wallet**
```
kiichaind keys add wallet --keyring-backend test --recover --coin-type 118 --key-type secp256k1
```
### Create Validator
```
kiichaind tx staking create-validator \
--amount=1000000000000000000000akii \
--moniker="$Your_Validator_Name" \
--identity="" \
--details="" \
--website="" \
--from wallet \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(kiichaind tendermint show-validator) \
--chain-id oro_1336-1 \
--fees=1000000000akii \
-y
```
---

### ⚙️ Setup Systemd Service
```
sudo tee /etc/systemd/system/kiichaind.service <<'EOF' > /dev/null
[Unit]
Description=Kiichain Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kiichaind) start --home $HOME/.kiichain
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Reload & enable service
```
sudo systemctl daemon-reload
sudo systemctl enable kiichaind
sudo systemctl start kiichaind
```
### Check Status
```
sudo systemctl status kiichaind
```
---

### 🔁 Manage Service
**Stop**
```
sudo systemctl stop kiichaind
```
**Restart**
```
sudo systemctl restart kiichaind
```
**View Logs**
```
journalctl -u kiichaind -f -o cat
```
---

### 🔍 Essential Commands

**Check Sync Status**
```
kiichaind status 2>&1 | jq
```
**Check Validator Info**
```
kiichaind query staking validator $(kiichaind keys show wallet --bech val -a)
```
**Monitor Logs**
```
journalctl -fu kiichaind -o cat
```

---

> Created by [OneNov](https://onenov.xyz) for the Kiichain Community 💙
