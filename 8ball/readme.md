<p align="right">
    <a href="https://t.me/genznodes">
    <img width="auto" height="50" src="https://user-images.githubusercontent.com/94878333/204091299-78a00a6b-a288-4db5-883f-1ef5106020e4.jpg">
    Support
    </a>
</p>


<p align="center">
    <img height="100" width="auto" src="https://user-images.githubusercontent.com/94878333/219876313-4db96124-91f7-457c-89a0-b58134b1b097.png">
</p>

## Hardware requirements

The following requirements are recommended :

- 4 or more physical CPU cores
- At least 500GB of SSD disk storage
- At least 32GB of memory (RAM)
- At least 100mbps network bandwidth

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
curl -L "https://8ball.info/8ball.tar.gz" > 8ball.tar.gz && \
    tar -C ./ -vxzf 8ball.tar.gz && \
    rm -f 8ball.tar.gz  && \
    sudo mv ./8ball /usr/local/bin/
```

Verify that you've installed successfully

```
8ball version
```

## Config app

```
8ball config chain-id eightball-1
8ball config keyring-backend file
```

## Initialize Node

```
planqd init $NODENAME --chain-id eightball-1
```

## Get genesis and addrbook

- genesis

```
curl -L "https://8ball.info/8ball-genesis.json" > genesis.json
mv genesis.json ~/.8ball/config/
```

- addrbook

```
NA
```

## Set peers and seeds

```
SEEDS=
PEERS=fca96d0a1d7357afb226a49c4c7d9126118c37e9@one.8ball.info:26656,aa918e17c8066cd3b031f490f0019c1a95afe7e3@two.8ball.info:26656,98b49fea92b266ed8cfb0154028c79f81d16a825@three.8ball.info:26656

sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.8ball/config/config.toml
```

## create service file and start node

- create

```
sudo tee /etc/systemd/system/8ball.service > /dev/null <<EOF
[Unit]
Description=8ball
After=network-online.target

[Service]
User=$USER
ExecStart=$(which 8ball) start --home $HOME/.8ball
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
sudo systemctl enable 8ball
sudo systemctl restart 8ball
sudo journalctl -fu 8ball -o cat
```
