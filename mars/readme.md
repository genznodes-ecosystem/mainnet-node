<p align="right">
    <a href="https://t.me/genznodes">
    <img width="auto" height="50" src="https://user-images.githubusercontent.com/94878333/204091299-78a00a6b-a288-4db5-883f-1ef5106020e4.jpg">
    Support
    </a>
</p>


<p align="center">
    <img height="100" width="auto" src="https://user-images.githubusercontent.com/94878333/218276596-8867703b-740e-49a7-bbf1-eef62df874da.jpg">
</p>

## Hardware Requirements

A production-ready server typically requires:

- 8-core x86 CPU. Cosmos apps do compile on ARM chips (e.g. Apple's M1 processor) but the reliability is not battle-tested. Notably, chains that incorporate the CosmWasm module won't even compile on ARM servers.

- 64 GB RAM. Cosmos apps typically use less than 32 GB under normal conditions, but during events such as chain upgrades, up to 64 GB is usually needed.

- 4 TB NVME SSD. Hard drive I/O speed is crucial! Validators who run on HDD or SATA SSD often find themselves missing blocks. Requirements on disk space depends on the chain and your pruning settings but generally at least 2 TB is recommended. 

- Linux operating system. 

# Install and update dependencies

```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

# Install Go

```
ver="1.19"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```

check golang

```
go version
```

# Install app

```
git clone https://github.com/mars-protocol/hub.git
cd hub
git checkout v1.0.0
make install
```

check if already marsd

```
marsd version --long
```

# init

```
marsd init <Moniker> --chain-id mars-1
```

## get genesis and addrbook

- genesis

```
wget -O $HOME/.mars/config/genesis.json https://snap.genznodes.dev/genesis/mars/genesis.json
```

- addrbook

```
wget -O $HOME/.mars/config/addrbook.json https://snap.genznodes.dev/addrbook/mars/addrbook.json
```

## create service file and start node

- create 

```
sudo tee /etc/systemd/system/marsd.service > /dev/null <<EOF
[Unit]
Description=marsd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which marsd) start --home $HOME/.mars
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

- start

```
sudo systemctl daemon-reload
sudo systemctl enable marsd
sudo systemctl restart marsd
sudo journalctl -fu marsd -o cat
```

