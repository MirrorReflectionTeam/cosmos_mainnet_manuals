## KEY

#### Add new key

```
palomad keys add wallet
```

#### Recover Existing Key

```
palomad keys add wallet --recover
```

#### Query Wallet Balance

```
palomad q bank balances $(palomad keys show wallet -a)
```

## Validator


#### Create new validator
```
palomad tx staking create-validator \
--amount=1000000ugrain \
--pubkey=$(palomad tendermint show-validator)
--moniker="YOUR_MONIKER_NAME" \
--chain-id=messenger \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0ugrain \
-y
```

#### Edit existing validator

```
palomad tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL"
--chain-id=messenger \
--commission-rate=0.07 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0ugrain \
-y
```

#### Unjail validator

```
palomad tx slashing unjail --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y  
```

#### View validator details

```
palomad q staking validator $(palomad keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
palomad tx distribution withdraw-rewards $(palomad keys show wallet --bech val -a) --commission --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

#### Delegate tokens to yourself

```
palomad tx staking delegate $(palomad keys show wallet --bech val -a) 1000000ugrain --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

#### Delegate tokens to validator

```
palomad tx staking delegate <TO_VALOPER_ADDRESS> 1000000ugrain --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y 
```

#### Redelegate tokens to validator
```
palomad tx staking redelegate $(palomad keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000ugrain --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

#### Unbond tokens from your validator

```
palomad tx staking unbond $(palomad keys show wallet --bech val -a) 1000000ugrain --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

#### Send tokens to wallet

```
palomad tx bank send wallet <TO_WALLET_ADDRESS> 1000000ugrain --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y 
```

## Governance

#### List all proposals

```
palomad query gov proposals
```

#### View proposal by id

```
palomad query gov proposal 1
```

#### Vote 'Yes'

```
palomad tx gov vote 1 yes --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y  
```

#### Vote 'No'

```
palomad tx gov vote 1 no --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

#### Vote 'Abstain'

```
palomad tx gov vote 1 abstain --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

#### Vote 'No With Veto'

```
palomad tx gov vote 1 NoWithVeto --from wallet --chain-id messenger --gas-adjustment 1.4 --gas auto --gas-prices 0ugrain -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.paloma/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.paloma/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.paloma/config/app.toml
```

#### Get validator info

```
palomad status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
palomad status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(palomad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.paloma/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop palomad && sudo systemctl disable palomad && sudo rm /etc/systemd/system/palomad.service && sudo systemctl daemon-reload && rm -rf $HOME/.paloma && rm -rf $HOME/paloma && sudo rm $(which palomad) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable palomad
```

#### Disable service

```
sudo systemctl disable palomad
```

#### Start service

```
sudo systemctl start palomad
```

#### Stop service

```
sudo systemctl stop palomad
```

#### Restart service

```
sudo systemctl restart palomad
```

#### Check service status

```
sudo systemctl status palomad
```

#### Check service logs

```
sudo journalctl -u palomad -f --no-hostname -o cat
```
