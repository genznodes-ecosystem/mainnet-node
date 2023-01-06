<p align="right">
    <a href="https://t.me/genznodes">
    <img width="auto" height="50" src="https://user-images.githubusercontent.com/94878333/204091299-78a00a6b-a288-4db5-883f-1ef5106020e4.jpg">
    SUPPORT
    </a>
</p>


<p align="center">
    <img height="100" width="auto" src="https://user-images.githubusercontent.com/94878333/211021651-2a062102-6b16-4169-a537-4aa6606ffe57.jpg">
</p>

# Guide how to join mainnet Planq

- Hardware requirements

The following requirements are recommended for running planq:

4 or more physical CPU cores
At least 500GB of SSD disk storage
At least 32GB of memory (RAM)
At least 100mbps network bandwidth

## setting vars

`<YOUR_MONIKER>` change with your moniker / name of your validator in explorer

```
NODENAME=<YOUR_MONIKER>
```

save and import variable

```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export NOLUS_CHAIN_ID=nolus-rila" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

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

## Install binary

```
git clone https://github.com/planq-network/planq.git
cd planq
git fetch
git checkout v1.0.2
make install
```

Verify that you've installed planq successfully

```
planqd version
```

## Config app

```
planqd config chain-id planq_7070-2
planqd config keyring-backend file
```

## Initialize Node

```
planqd init $NODENAME --chain-id planq_7070-2
```

## Get genesis and addrbook

- genesis

```
wget https://raw.githubusercontent.com/planq-network/networks/main/mainnet/genesis.json
mv genesis.json ~/.planqd/config/
```

- addrbook

```
curl -o addrbook.json https://raw.githubusercontent.com/Genz22/mainnet-node/main/planq/addrbook.json > $HOME/.planqd/config/addrbook.json
```

## Set peers and seeds

```
SEEDS=`curl -sL https://raw.githubusercontent.com/planq-network/networks/main/mainnet/seeds.txt | awk '{print $1}' | paste -s -d, -`
PEERS=`curl -sL https://raw.githubusercontent.com/planq-network/networks/main/mainnet/peers.txt | sort -R | head -n 10 | awk '{print $1}' | paste -s -d, -`

sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.planqd/config/config.toml
```

## Statesync (optional)

```
STATE_SYNC_RPC=https://planq-rpc.genznodes.dev:443
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 2000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.planqd/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
  $HOME/.planqd/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
  $HOME/.planqd/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
  $HOME/.planqd/config/config.toml
```

## create service file and start node

- create

```
sudo tee /etc/systemd/system/planqd.service > /dev/null <<EOF
[Unit]
Description=planqd
After=network-online.target

[Service]
User=root
ExecStart=$(which planqd) start --home $HOME/.planqd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

- enable service and start

```
sudo systemctl daemon-reload
sudo systemctl enable planqd
sudo systemctl restart planqd
sudo journalctl -fu planqd -o cat
```

## add keys or import keys

- create

```
planqd keys add wallet
```

- import

```
planqd keys add wallet --recover
```

## Create validator

- create validator

```
planqd tx staking create-validator \
  --amount 1000000000000aplanq \
  --pubkey $(planqd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id planq_7070-2 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation "1" \
  --gas-prices="30000000000aplanq" \
  --gas-adjustment="1.15" \
  --from wallet -y
```

- check your validator 

https://explorer.genznodes.dev/planq

# Usefull commands

N/A
