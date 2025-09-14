## OneNov's Step-by-Step Guide for Kiichain Validator Installation

<img src="https://pbs.twimg.com/profile_images/1800553180083666944/zZe128CW.jpg" alt="Kiichain Logo" width="450"/>

This is a community-created guide for setting up a Kiichain validator node.

System Requirements

These requirements are for a mainnet validator node with default pruning settings. Archive nodes will require significantly more storage.

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| OS | Ubuntu 20.04 | Ubuntu 22.04 LTS |
| CPU | 4 Cores | 8+ Cores |
| RAM | 8GB | 16GB+ |
| Storage | 500GB SSD/NVMe | 1TB SSD/NVMe |
| Network | 10Mbps | 100Mbps+ |
| Pruning | Default | Default/Everything |
| Indexer | Off | Off |

Note: NVMe SSD is strongly recommended for better I/O performance. Ensure accurate time synchronization using chrony for consensus correctness.

---

Step 1: Update System and Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl build-essential make jq gcc chrony lz4 tmux unzip bc -y
```

Step 2: Install Go (Verified)

```bash
GO_VERSION="1.22.3"
cd $HOME

# Hapus previous install
rm -rf $HOME/go
sudo rm -rf /usr/local/go

# Download dan verify checksum
curl -fsSLO "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
curl -fsSLO "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz.sha256"
sha256sum -c "go${GO_VERSION}.linux-amd64.tar.gz.sha256" || exit 1

# Install
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"

# Setup environment (only if there isn't one yet)
if ! grep -q '/usr/local/go/bin' $HOME/.profile; then
    cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
fi

# Apply dan test
source $HOME/.profile
go version

# Cleanup
rm "go${GO_VERSION}.linux-amd64.tar.gz" "go${GO_VERSION}.linux-amd64.tar.gz.sha256"
```

Step 3: Build Kiichain

```bash
cd $HOME
rm -rf kiichain
git clone https://github.com/KiiChain/kiichain.git
cd kiichain
# Checkout the latest stable release - replace with actual version
git checkout vX.Y.Z
make install
kiichaind version
```

Step 4: Initialize Node and Download Genesis

```bash
kiichaind init YOUR_NODE_NAME --chain-id oro_1336-1
```

# Download official genesis file
```
curl -fsSL https://raw.githubusercontent.com/KiiChain/testnets/refs/heads/main/testnet_oro/genesis.json > $HOME/.kiichain/config/genesis.json
```

Step 5: Configure Peers and Seeds

```bash
# Set persistent peers
PEERS="5b6aa55124c0fd28e47d7da091a69973964a9fe1@uno.sentry.testnet.v3.kiivalidator.com:26656,5e6b283c8879e8d1b0866bda20949f9886aff967@dos.sentry.testnet.v3.kiivalidator.com:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kiichain/config/config.toml

# Add seeds (fallback discovery) - replace with official seeds
SEEDS="id1@seed1.example.org:26656,id2@seed2.example.org:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" $HOME/.kiichain/config/config.toml

# Enable peer exchange and set reasonable limits
sed -i 's|^pex *=.*|pex = true|' $HOME/.kiichain/config/config.toml
sed -i 's|^max_num_inbound_peers *=.*|max_num_inbound_peers = 100|' $HOME/.kiichain/config/config.toml
sed -i 's|^max_num_outbound_peers *=.*|max_num_outbound_peers = 50|' $HOME/.kiichain/config/config.toml
```

Step 6: Set Minimum Gas Price

```bash
# Use recommended minimum gas price from chain documentation
sed -i 's|^minimum-gas-prices =.*|minimum-gas-prices = "0.0025akii"|' $HOME/.kiichain/config/app.toml
```

Step 7: Wallet Setup

Create Wallet (Secure)

```bash
kiichaind keys add wallet --keyring-backend os --coin-type 118 --key-type secp256k1
```

Recover Wallet

```bash
kiichaind keys add wallet --keyring-backend os --recover --coin-type 118 --key-type secp256k1
```

Important: For mainnet validators, consider using a hardware wallet for maximum security.

Step 8: Create Validator

```bash
kiichaind tx staking create-validator \
--amount=1000000akii \
--moniker="YOUR_VALIDATOR_NAME" \
--identity="" \
--details="" \
--website="" \
--from wallet \
--commission-rate 0.10 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(kiichaind tendermint show-validator) \
--chain-id oro_1336-1 \
--gas auto --gas-adjustment 1.3 --gas-prices "0.0025akii" \
-y
```

Step 9: Configure Firewall

```bash
sudo ufw allow 26656/tcp    # P2P port
sudo ufw allow 22/tcp       # SSH
sudo ufw enable
sudo ufw status
```

Step 10: Setup Systemd Service

First, create a dedicated system user:

```bash
sudo useradd -r -s /usr/sbin/nologin -m -d /var/lib/kiichain kiichain
sudo rsync -av $HOME/.kiichain/ /var/lib/kiichain/
sudo chown -R kiichain:kiichain /var/lib/kiichain
```

Create the systemd service:

```bash
sudo tee /etc/systemd/system/kiichaind.service > /dev/null <<EOF
[Unit]
Description=Kiichain Node
After=network-online.target
Wants=network-online.target

[Service]
User=kiichain
Group=kiichain
ExecStart=/usr/local/bin/kiichaind start --home /var/lib/kiichain
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=/var/lib/kiichain"
WorkingDirectory=/var/lib/kiichain
ProtectSystem=full
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF
```

Start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now kiichaind
journalctl -u kiichaind -f -o cat
```

---

Helpful Commands

Stop

```bash
sudo systemctl stop kiichaind
```

Restart

```bash
sudo systemctl restart kiichaind
```

View Logs

```bash
journalctl -u kiichaind -f -o cat
```

Check Sync Status

```bash
kiichaind status 2>&1 | jq .SyncInfo
```

Check Validator Info

```bash
# If supported:
kiichaind query staking validator $(kiichaind keys show wallet --bech val -a)
# Otherwise:
OP=$(kiichaind keys show wallet -a | sed 's/^kii/kiivaloper/') && kiichaind query staking validator "$OP"
```

Monitor Logs

```bash
journalctl -u kiichaind -f
```

---

Optional: State Sync (Fast Sync)

To reduce synchronization time, you can use state sync. Replace with actual trust height and hash from a trusted RPC:

```bash
TRUST_HEIGHT=1000000
TRUST_HASH="ABCDEF123456..."
SYNC_RPC="https://rpc.kiichain.com:443"

sed -i 's|^enable *=.*|enable = true|' $HOME/.kiichain/config/config.toml
sed -i "s|^trust_height *=.*|trust_height = $TRUST_HEIGHT|" $HOME/.kiichain/config/config.toml
sed -i "s|^trust_hash *=.*|trust_hash = \"$TRUST_HASH\"|" $HOME/.kiichain/config/config.toml
sed -i "s|^rpc_servers *=.*|rpc_servers = \"$SYNC_RPC,$SYNC_RPC\"|" $HOME/.kiichain/config/config.toml
```

Note: State sync must be configured before starting the node for the first time.

---

Credits

Guide created by [OneNov](https://www.onenov.xyz/validator/Kiichain). Always verify all commands and URLs with official Kiichain documentation.

---
