<p align="right">
    <a href="https://t.me/genznodes">
    <img width="auto" height="50" src="https://user-images.githubusercontent.com/94878333/204091299-78a00a6b-a288-4db5-883f-1ef5106020e4.jpg">
    SUPPORT
    </a>
</p>


<p align="center">
    <img height="100" width="auto" src="https://user-images.githubusercontent.com/94878333/204089761-19c1cebf-ce7f-4d81-bade-46072cc99dd0.jpg">
</p>

# Guide how to join mainnet mises

- Hardware requirements

The following requirements are recommended for running Mises:

At least 300 mbps of network bandwidth

- 4 core or higher CPU
- 32GB RAM
- 2TB NVME storage
- At least 300mbps network bandwidth

## Install dependences

```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## Install go

```
ver="1.19"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## Install and build node

```
git clone https://github.com/mises-id/mises-tm/
cd mises-tm/
git checkout main
make install
```

Verify that you've installed misestmd successfully

```
misestmd version
```

## Init

```
misestmd init <moniker> --chain-id mainnet
```

## Get genesis and addrbook

```
curl https://e1.mises.site:443/genesis | jq .result.genesis > ~/.misestm/config/genesis.json
wget -O $HOME/.misestm/config/addrbook.json https://raw.githubusercontent.com/Genz22/mainnet-node/main/mises/addrbook.json
```

## set peers

```
PERSISTENT_PEERS="40a8318fa18fa9d900f4b0d967df7b1020689fa0@e1.mises.site:26656"
sed -i.bak -E "s|^(persistent_peers[[:space:]]+=[[:space:]]+).*$|\1\"$PERSISTENT_PEERS\"|"  ~/.misestm/config/config.toml
```

## Create key

```
misestmd keys add wallet
```

or recover key with :

```
misestmd keys add wallet --recover
```

and fund your wallet

## create service file and start node

```
sudo tee /etc/systemd/system/misestmd.service > /dev/null <<EOF
[Unit]
Description=Misestmd
After=network-online.target

[Service]
Type=simple
User=root
ExecStart=$(which misestmd) start  
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable misestmd
sudo systemctl restart misestmd
journalctl -fu misestmd -o cat
```

close with `CTRL + C`

## Statesync (optional)


```
sudo systemctl stop misestmd

cp $HOME/.misestm/data/priv_validator_state.json $HOME/.misestm/priv_validator_state.json.backup
misestmd tendermint unsafe-reset-all --home $HOME/.misestm --keep-addr-book

SNAP_RPC="https://human-rpc.genznodes.dev:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="563b2b4535a36a58d2366aab7de73ddf8f23a28b@85.190.246.191:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.misestm/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.misestm/config/config.toml

mv $HOME/.misestm/priv_validator_state.json.backup $HOME/.misestm/data/priv_validator_state.json

sudo systemctl restart misestmd
sudo journalctl -fu misestmd -o cat
```

## Create validator

- check sync info

```
misestmd status 2>1 | jq .SyncInfo
```

make sure that "catching_up": false

- create validator

```
misestmd tx staking create-validator \
  --amount 1000000umis \
  --from wallet \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey $(misestmd tendermint show-validator) \
  --moniker "<moniker>" \
  --chain-id "mainnet" \
  --gas auto -y
```

Explore :

**OFFICIAL** https://gw.mises.site/validators

or https://explorer.genznodes.dev/mises

## Useful commands

- check node

```
misestmd status |& jq
```

- check logs

```
journalctl -fu misestmd -o cat
```

- check key

```
misestmd keys list
```

- check balances 

```
misestmd query bank balances <address>
```

❤ Stake with genznodes ❤

https://portal.mises.site/validator/misesvaloper1s9y4vkmsu30mc74qkhn083mhj6tgdvvecxdcss
