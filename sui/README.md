<p style="font-size:14px" align="right">
<a href="https://hetzner.cloud/?ref=jQhYwCyb1jPf" target="_blank">Deploy your VPS using our referral link to get 20€ bonus</a>
</p>

# <img width="25" height="auto" src="https://user-images.githubusercontent.com/50621007/174559198-c1f612e5-bba2-4817-95a8-8a3c3659a2aa.png"> Sui node setup
Sui is a decentralized, high-bandwidth, low-latency smart contract platform using the Move language.

## Information
- Official manual: https://docs.sui.io/build/fullnode
- Node health: https://node.sui.zvalid.com
- Explorer: https://explorer.devnet.sui.io

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
sudo apt install wget jq git libclang-dev pkg-confi ca-certificates build-essential libssl-dev libudev-dev clang pkg-config cmake -y
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

## Operations with objects

### 1. Create an NFT example
```
sui client create-example-nft
```
You should see output like this:
```
Successfully created an ExampleNFT:

----- Move Object (***[1]) -----
Owner: Account Address ( (***[ )
Version: 1
Storage Rebate: 25
Previous Transaction: tv/MWuMQJgfgdPhX1HBTaOKbTTItZFgDDmSEqgt3B2E=
----- Data -----
type: 0x2::devnet_nft::DevNetNFT
description: An NFT created by the Sui Command Line Tool
id: (***[
name: Example NFT
url: ipfs://bafkreibngqhl3gaa7daob4i2vccziay2jjlp435cf66vhono7nrvww53ty
```

### 2. Create an NFT with specific image
```
sui client create-example-nft --name "<YOUR_NFT_NAME>" --url "<URL_TO_IMAGE_YOU_LIKE>"
```
You should see output similar output. 

### 3. Merge two objects into one
```
JSON=$(sui client gas --json | jq -r)
FIRST_OBJECT_ID=$(echo $JSON | jq -r .[0].id.id)
SECOND_OBJECT_ID=$(echo $JSON | jq -r .[1].id.id)
sui client merge-coin --primary-coin ${FIRST_OBJECT_ID} --coin-to-merge ${SECOND_OBJECT_ID} --gas-budget 1000
```

You should see output like this:
```
----- Certificate ----
Transaction Hash: VFCb0bz8OXB8mFjjsJLsUCVnZSC8dIV++2OjKLNu//c=
Transaction Signature: AA==@B54xgjHBvjm/Q0bVs07jJaT2+K8hs4Vx2e1ZYm1aTsLK6XGPzwaqhlnNB1uyg3uK/HkOuB3WRGtyy2o/l4ElCg==@RQaPWLzW3kbtq/Fp0Y7/xX9pAfvCQ0JhINZoE71dPcU=
Signed Authorities Bitmap: RoaringBitmap<[0, 1, 3]>
Transaction Kind : Call
Package ID : 0x2
Module : coin
Function : join
Arguments : ["0x7dd750192ccdb481c3d58cb868cd9445bab547b", "0x30a49d2b5f02fdb2aeaa702bb013abe2a12a2706"]
Type Arguments : ["0x2::sui::SUI"]
----- Merge Coin Results ----
Updated Coin : Coin { id: 0x07dd750192ccdb481c3d58cb868cd9445bab547b, value: 176513 }
Updated Gas : Coin { id: 0x497761cd29b921166daf138145b9a76edb403d1c, value: 49553 }
```

You can see that two first objects are now merged into one and gas has been payed by third object:
```
sui client gas
```

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
