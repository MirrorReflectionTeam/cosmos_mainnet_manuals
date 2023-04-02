<div align="center">
  <h2> Instructions </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

<div align="center">
  <h2> Snapshot </h2>
</div>

```
sudo systemctl stop palomad

cp $HOME/.paloma/data/priv_validator_state.json $HOME/.paloma/priv_validator_state.json.backup

palomad tendermint unsafe-reset-all --home $HOME/.paloma --keep-addr-book
curl -L https://snapshots1-cosmos.mirror-reflection.com/cosmos-mainnet/paloma-mainnet/messenger_latest.tar | tar -xf - -C $HOME/.paloma/data

mv $HOME/.paloma/priv_validator_state.json.backup $HOME/.paloma/data/priv_validator_state.json

sudo systemctl start palomad
sudo journalctl -u palomad -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop palomad

cp $HOME/.paloma/data/priv_validator_state.json $HOME/.paloma/priv_validator_state.json.backup
palomad tendermint unsafe-reset-all --home $HOME/.paloma --keep-addr-book

SNAP_RPC="https://rpc.paloma-mainnet.mirror-reflection.com:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="ef1cd7da8319351b51ec930924929d03a5b76dc3@rpc.paloma-mainnet.mirror-reflection.com:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.paloma/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.paloma/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.paloma/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.paloma/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.paloma/config/config.toml

mv $HOME/.paloma/priv_validator_state.json.backup $HOME/.paloma/data/priv_validator_state.json

sudo systemctl restart palomad
sudo journalctl -u palomad -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="ef1cd7da8319351b51ec930924929d03a5b76dc3@rpc.paloma-mainnet.mirror-reflection.com:26656,f4c43099e04b721c54a454dad85f61da49be90bc@65.108.199.222:28656,dfa0d66a3713bf6b49bc509a2a4fc75bee042a30@23.88.77.188:20009,99c890c97afc8abfdfeff662d539af5c504a0baf@88.99.67.234:26656,874ccf9df2e4c678a18a1fb45a1d3bb703f87fa0@65.109.172.249:26656,317141e329bc214a76ba92201f6818574ebe5323@135.181.114.98:36656,b92c94f00b46500a5ff8920acd438c0873c2f9da@50.116.13.101:26656,2c6772b11c1f9eff2a923eb2bf808543cdd501c5@79.143.179.196:26656,4569193b58dfc6d9ca9acd4e2bcabf596e5b6b3c@65.21.7.251:10656,8af8dfa817359036f55f6793b0ed4bcce8884027@85.14.245.70:26656,b3ba407aef9e18e16e8e9a3b523a1b026dabeab3@84.46.248.174:26656,810bea15ec11d510dd33170851ee2ab74c48b6de@81.0.221.57:26656,b244dfc19293103040d4bdad359534d0990a9070@45.140.185.181:26656,ab6875bd52d6493f39612eb5dff57ced1e3a5ad6@95.217.229.18:10656,e4b7cdd48c39c355e9a3480f4f4d5afab8fb0e08@46.0.203.78:26637,e833844c00b8ce60ce6826f170becfa18e6172c2@46.4.27.59:26656,06e9c9d5c07755d36241249a568b51ec8476fe65@135.181.220.168:26656,741bdc1ffdd93b7de09d8417bc0ada0d2285affa@65.109.57.67:27656,ca92abdc4599dd91dd63e689c64c468df5425f2c@95.216.100.99:10656,124cbe860f1eaa8084444587928db17c78ebd8f3@149.90.94.145:26658,19165f3248f358ded53c3f51cf97a22123560b86@65.109.69.154:38656,d9bfa29e0cf9c4ce0cc9c26d98e5d97228f93b0b@65.109.88.38:10656,45a3b59139f6db435c9a803c8f78eb2feda82c7c@135.181.144.214:26656"
sed -i 's|^persistent_peers =.|persistent_peers = "'$PEERS'"|' $HOME/.paloma/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -Ls https://snapshots1-cosmos.mirror-reflection.com/cosmos-mainnet/paloma-mainnet/addrbook.json > $HOME/.paloma/config/addrbook.json
```

<div align="center">
  <h2> Wasm </h2>
</div>

```
curl https://snapshots1-cosmos.mirror-reflection.com/cosmos-mainnet/paloma-mainnet/wasm_messenger_.tar | tar -xf - -C $HOME/.paloma/data
```
