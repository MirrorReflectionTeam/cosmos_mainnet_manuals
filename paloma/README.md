<div align="center">
  <h1> Manual Installation </h1>
</div>

### Paloma

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/paloma.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://www.palomachain.com/)** | **[Explorer](https://paloma.explorers.guru/)**

### Public endpoints

- **https://rpc.paloma-mainnet.mirror-reflection.com**

- **https://grpc.paloma-mainnet.mirror-reflection.com**

- **https://api.paloma-mainnet.mirror-reflection.com**

### Hardware Requirements

4CPU 8RAM 160GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

### Install dependencies

#### Install update, build tools and GO if you needed

```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.19.7.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

### Setup Validator Name

Replace YOUR_MONIKER_NAME with your node name

```
NODE_MONIKER="YOUR_MONIKER_NAME"
```

### Download and build binaries

```
cd || return
curl -L https://github.com/palomachain/paloma/releases/download/v0.11.6/paloma_Linux_x86_64.tar.gz > paloma.tar.gz
tar -xvzf paloma.tar.gz
rm -rf paloma.tar.gz
sudo mv -f palomad /usr/local/bin/palomad
palomad version
```

```
curl -L https://github.com/palomachain/pigeon/releases/download/v0.11.5/pigeon_Linux_x86_64.tar.gz > pigeon.tar.gz
tar -xvzf pigeon.tar.gz
rm -rf pigeon.tar.gz
sudo mv -f pigeon /usr/local/bin/pigeon
pigeon version
```

```
curl -L https://github.com/CosmWasm/wasmvm/raw/main/internal/api/libwasmvm.x86_64.so > libwasmvm.x86_64.so
sudo mv -f libwasmvm.x86_64.so /usr/lib/libwasmvm.x86_64.so
```

### Initialize the node

```
palomad config chain-id messenger
palomad init "$NODE_MONIKER" --chain-id messenger
```

### Download genesis and addrbook

```
curl -Ls https://rpc.paloma-mainnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.paloma/config/genesis.json
curl -Ls https://snapshots1-cosmos.mirror-reflection.com/cosmos-mainnet/paloma-mainnet/addrbook.json > $HOME/.paloma/config/addrbook.json
```

### Add seeds

```
SEEDS="ef1cd7da8319351b51ec930924929d03a5b76dc3@rpc.paloma-mainnet.mirror-reflection.com:26656,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@paloma.rpc.kjnodes.com:10659"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.paloma/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.paloma/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0ugrain"|g' $HOME/.paloma/config/app.toml
```

### Resetting data

```
palomad tendermint unsafe-reset-all --home $HOME/.paloma --keep-addr-book
```

### Prepare and create pigeon

```
echo "export PIGEON_HEALTHCHECK_PORT=5757" >> $HOME/.bash_profile
source .bash_profile
```

```
mkdir -p $HOME/.pigeon
```

```
sudo tee $HOME/.pigeon/config.yaml > /dev/null << EOF
loop-timeout: 5s
health-check-port: 5757

paloma:
  chain-id: messenger
  call-timeout: 20s
  keyring-dir: ~/.paloma
  keyring-pass-env-name: "PALOMA_PASSWORD"
  keyring-type: os
  signing-key: wallet
  base-rpc-url: http://localhost:26657
  gas-adjustment: 1.5
  gas-prices: 0ugrain
  account-prefix: paloma

evm:
  eth-main:
    chain-id: 1
    base-rpc-url: \${ETH_RPC_URL}
    keyring-pass-env-name: "ETH_PASSWORD"
    signing-key: \${ETH_SIGNING_KEY}
    keyring-dir: ~/.pigeon/keys/evm/eth-main
    gas-adjustment: 1.9
    tx-type: 2
EOF
```

### Create a pigeon wallet and keys

```
palomad keys add wallet
```

```
pigeon evm keys generate-new $HOME/.pigeon/keys/evm/eth-main
```

```
PALOMA_PASSWORD=<YOUR_PALOMA_PASSWORD>
ETH_PASSWORD=<YOUR_ETH_PASSWORD>
ETH_SIGNING_KEY=0x$(cat $HOME/.pigeon/keys/evm/eth-main/*  | jq -r .address | head -n 1)
```

```
sudo tee $HOME/.pigeon/env.sh > /dev/null << EOF
PALOMA_PASSWORD=$PALOMA_PASSWORD
ETH_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/9bYS5h99MVmaa7f0fYztaHwN31k2EBvZ
ETH_PASSWORD=$ETH_PASSWORD
ETH_SIGNING_KEY=$ETH_SIGNING_KEY
EOF
```

### Create a service

```
sudo tee /etc/systemd/system/palomad.service > /dev/null << EOF
[Unit]
Description=Paloma Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which palomad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

```
sudo tee /etc/systemd/system/pigeond.service > /dev/null << EOF
[Unit]
Description=Pigeon daemon
After=network-online.target
ConditionPathExists=$(which pigeon)

[Service]
Type=simple
Restart=always
RestartSec=5
User=$USER
WorkingDirectory=$HOME
EnvironmentFile=$HOME/.pigeon/env.sh
ExecStart=$(which pigeon) start
ExecReload=

[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl -L https://snapshots1-cosmos.mirror-reflection.com/cosmos-mainnet/paloma-mainnet/messenger_latest.tar | tar -xf - -C $HOME/.paloma/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable palomad
sudo systemctl enable pigeond
sudo systemctl start pigeond
sudo systemctl start palomad
```

```
sudo journalctl -u pigeond -f --no-hostname -o cat
```

```
sudo journalctl -u palomad -f --no-hostname -o cat
```
