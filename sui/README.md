
# <img width="25" height="auto" src="https://user-images.githubusercontent.com/50621007/174559198-c1f612e5-bba2-4817-95a8-8a3c3659a2aa.png"> Sui node setup

## Information
- Official manual: https://docs.sui.io/build/fullnode
- Node health: https://node.sui.zvalid.com/

## Minimum hardware requirements
- CPU: 2 CPU
- Memory: 4 GB RAM
- Disk: 50 GB SSD Storage

## Set up Sui node

### 1. Update packages
```
sudo apt update && sudo apt upgrade -y
```

### 2. Install dependencies
```
sudo apt install wget jq git libclang-dev ca-certificates build-essential libssl-dev pkg-config libclang-dev cmake -y
```

### 3. Install Rust
```
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
```

### 4. Download and compile binaries
```
cd $HOME
git clone https://github.com/MystenLabs/sui.git
cd sui
git remote add upstream https://github.com/MystenLabs/sui
git fetch upstream
git checkout --track upstream/devnet
cargo build --release
mv $HOME/sui/target/release/{sui,sui-node,sui-faucet} /usr/local/bin/
```

### 5. Create configs
```
mkdir -p $HOME/.sui
wget -qO $HOME/.sui/fullnode.yaml https://github.com/MystenLabs/sui/raw/main/crates/sui-config/data/fullnode-template.yaml
wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob
yq -i ".db-path = \"$HOME/.sui/db\"" $HOME/.sui/fullnode.yaml
yq -i '.metrics-address = "0.0.0.0:9184"' $HOME/.sui/fullnode.yaml
yq -i '.json-rpc-address = "0.0.0.0:9000"' $HOME/.sui/fullnode.yaml
yq -i ".genesis.genesis-file-location = \"$HOME/.sui/genesis.blob\"" $HOME/.sui/fullnode.yaml
```

### 6. Create sui service
```
sudo tee /etc/systemd/system/suid.service > /dev/null <<EOF
[Unit]
Description=Sui node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sui-node) --config-path $HOME/.sui/fullnode.yaml
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 7. Start sui node
```
sudo systemctl daemon-reload
sudo systemctl enable suid
sudo systemctl restart suid
```

### 8. Check status of your node
```
curl -s -X POST http://127.0.0.1:9000 -H 'Content-Type: application/json' -d '{ "jsonrpc":"2.0", "method":"rpc.discover","id":1}' | jq .result.info
```
You should see something similar in the output:
```json
{
  "title": "Sui JSON-RPC",
  "description": "Sui JSON-RPC API for interaction with the Sui network gateway.",
  "contact": {
    "name": "Mysten Labs",
    "url": "https://mystenlabs.com",
    "email": "build@mystenlabs.com"
  },
  "license": {
    "name": "Apache-2.0",
    "url": "https://raw.githubusercontent.com/MystenLabs/sui/main/LICENSE"
  },
  "version": "0.1.0"
}
```

### 9. Check your node health status
Enter your node IP into https://node.sui.zvalid.com/
Healthy node should look like this:
<p align="center">
  <img width="800" height="auto" src="https://github.com/hivefive-space/guides/blob/a0dc6b22a21b789e338600f918963d8d895e1735/sui/img/healthcheck.png">
</p>

## Connect wallet
Sui Wallet is an open-sourced wallet for SUI network, allowing users the ability to create an address, view and manage assets on the Sui network, and interact with dApps.

### 1. Chrome extension
Download the Chrome extension from the store using [this link](https://chrome.google.com/webstore/detail/sui-wallet/opcgpfmipidbgpenhmajoajpbobppdil).
<p align="center">
  <img width="800" height="auto" src="https://github.com/hivefive-space/guides/blob/a0dc6b22a21b789e338600f918963d8d895e1735/sui/img/chrome_extension.png">
</p>

### 2. Create an account
Create a new account by clicking on the Application -> Get started -> Create new account.
<p align="center">
  <img width="600" height="auto" src="https://github.com/hivefive-space/guides/blob/a0dc6b22a21b789e338600f918963d8d895e1735/sui/img/create_account.png">
</p>

### 3. Seed phrase
You will be prompted with a Seed phrase, which is the only way to recover your account.
Save this and find your wallet address in the extension window.
<p align="center">
  <img width="600" height="auto" src="https://github.com/hivefive-space/guides/blob/db9d06176315e6f71f66a06d1dd7f5412c4e06ad/sui/img/mnemonic.png">
</p>

### 4. Connect wallet to Sui node
```
sui keytool import "<YOUR_MNEMONIC_PHRASE>" ed25519
```

### 5. Get your wallet address
```
sui client active-address
```

### 6. Top up your wallet
Navigate to [Sui Discord](https://discord.gg/sui) `#devnet-faucet` channel and top up your wallet
```
!faucet <YOUR_WALLET_ADDRESS>
```

### 7. Check balance
You can check your balance at `https://explorer.devnet.sui.io/addresses/<YOUR_WALLET_ADDRESS>`.

## Useful commands
Check status
```
systemctl status suid
```

Check logs
```
journalctl -fu suid -o cat
```

Check version
```
sui --version
```

## Update Sui node
```
sudo systemctl stop suid
rm -rf $HOME/.sui/db
wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob
source $HOME/.cargo/env
cd $HOME/sui
git remote add upstream https://github.com/MystenLabs/sui
git fetch upstream
git stash
git checkout --track upstream/devnet
cargo build --release
mv $HOME/sui/target/release/{sui,sui-node,sui-faucet} /usr/local/bin/
sudo systemctl restart suid
```

## Delete Sui node
```
sudo systemctl stop suid
sudo systemctl disable suid
sudo rm -rf $HOME/{sui,.sui} 
sudo rm -rf /usr/bin/{sui,sui-node,sui-faucet}
sudo rm -rf /etc/systemd/system/suid.service
sudo systemctl daemon-reload
```
