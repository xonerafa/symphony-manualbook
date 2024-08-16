---
icon: hand-point-right
---

# Installation guide

Instalation Guide

### Install prequirement

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### Install go

```
ver="1.21.4"
```

```
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
```

```
sudo rm -rf /usr/local/go
```

```
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
```

```
rm "go$ver.linux-amd64.tar.gz"
```

```
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
go version
```

### Download binary

```
cd $HOME
rm -rf symphony
git clone https://github.com/Orchestra-Labs/symphony symphony
cd symphony
git checkout 0.3.0
make install
```

### Config Node

* Init Node

```
symphonyd init x1node --chain-id symphony-testnet-3
symphonyd config chain-id symphony-testnet-3
symphonyd config keyring-backend test
```

* Download Genesis && AddrBook

```
curl -Ls https://nuaink.x1node.xyz/genesis.json > $HOME/.symphonyd/config/genesis.json
curl -Ls https://nuaink.x1node.xyz/addrbook.json > $HOME/.symphonyd/config/addrbook.json

```

* Add Peers or Seed

```
SEEDS="1dabe13c78472344711ccd9c326bc422a1bda5a8@152.53.66.24:26656"
PEERS="8ba2e8c2964a58e69f02af175e473ce232d544dc@109.123.238.47:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.symphonyd/config/config.toml

```

* Setup minimum GasFee

```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0note\"/" $HOME/.symphonyd/config/app.toml

```

* Setup pruning

```
sed -i \
-e 's|^pruning *=.*|pruning = "custom"|' \
-e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
-e 's|^pruning-keep-every *=.*|pruning-keep-every = ""|' \
-e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
$HOME/.symphonyd/config/app.toml
```

* Create service

```
sudo tee /etc/systemd/system/symphonyd.service > /dev/null <<EOF
[Unit]
Description=symphony
After=network-online.target

[Service]
User=$USER
ExecStart=$(which symphonyd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Launch Node

```
sudo systemctl daemon-reload 
```

```
sudo systemctl enable symphonyd
```

```
sudo systemctl restart symphonyd
```

### Check log node

```
journalctl -fu symphonyd -o cat
```

### Wallet configuration

* add wallet

```
symphonyd keys add wallet
```

* recover wallet

```
symphonyd keys add wallet --recover
```

* list wallet

```
symphonyd keys list
```

* delete wallet

```
symphonyd keys delete wallet
```

* check balances

```
symphonyd q bank balances $(symphonyd keys show wallet -a)
```

### Validator Management

* create validator

```
symphonyd tx staking create-validator \
--amount="1000000000note" \
--pubkey=$(symphonyd tendermint show-validator) \
--moniker=NodeName \
--identity="F57A71944DDA8C4B" \
--website="https://dwentz.xyz" \
--details=details node validator \
--chain-id=symphony-testnet-2 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000 \
--gas-adjustment=1.2 \
--gas-prices="0.5note" \
--gas=auto \
--from=wallet
```

* edit validator

```
symphonyd tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=symphony-testnet-2 \
--gas-adjustment=1.2 \
--gas-prices="0.5note" \
--gas=auto \
--from=wallet
```

* unjail validator

```
symphonyd tx slashing unjail --from wallet --chain-id symphony-testnet-2 --gas-prices 0.5note --gas-adjustment 1.2 --gas auto
```

* check jailed reason

```
symphonyd query slashing signing-info $(symphonyd tendermint show-validator)
```

### Token management

* withdraw rewards

```
symphonyd tx distribution withdraw-all-rewards --from wallet --chain-id symphony-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5note --gas auto -y
```

* withdraw rewards with comission

```
symphonyd tx distribution withdraw-rewards $(symphonyd keys show wallet --bech val -a) --commission --from wallet --chain-id symphony-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5note --gas auto -y
```

* delegate token to your own validator

```
symphonyd tx staking delegate $(symphonyd keys show wallet --bech val -a) 1000000note --from wallet --chain-id symphony-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5note --gas auto -y
```

### Node Info

* node id

```
symphonyd status 2>&1 | jq .NodeInfo
```

* validator info

```
symphonyd status 2>&1 | jq .ValidatorInfo
```

### Delete Node

```
sudo systemctl stop symphonyd &&
sudo systemctl disable symphonyd &&
sudo rm /etc/systemd/system/symphonyd.service &&
sudo systemctl daemon-reload &&
rm -f $(which symphonyd) &&
rm -rf .symphonyd
```
