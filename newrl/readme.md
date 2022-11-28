<p align="right">
    <a href="https://t.me/genznodes">
    <img width="auto" height="50" src="https://user-images.githubusercontent.com/94878333/204091299-78a00a6b-a288-4db5-883f-1ef5106020e4.jpg">
    SUPPORT
    </a>
</p>

<p align="center">
    <img height="100" width="auto" src="https://user-images.githubusercontent.com/94878333/204346560-59540b94-deb2-4b2b-bde8-0bd967d004d0.png">
</p>

# Guide how to join mainnet mises

Requiretments 

- open port 8456

```
ufw allow 8456
ufw allow 22
ufw enable
```

## update packages and install dependences 

```
sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential libssl-dev libffi-dev git curl screen python3.10 python3.10-venv -y
```

## Get pip

```
curl -o get-pip.py https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py
```

## Clone repo and setup Environment

```
git clone https://github.com/newrlfoundation/newrl.git
cd newrl
python3 -m venv newrl-venv
source newrl-venv/bin/activate
```

## install node

```
scripts/install.sh mainnet
```

flags =

- mainnet
- testnet
- devnet

## Wallet

- check wallet 

```
cat $HOME/newrl/data_mainnet/.auth.json
```

import your wallet : https://wallet.newrl.net/

## Import existing wallet

```
python3 scripts/import_wallet.py 
Enter environment[mainnet/testnet/devnet]: mainnet
```

## Start node

```
screen -R newrl
scripts/start.sh mainnet --fullnode
```

## update node 

```
screen -rd newrl
```

and stop node with : `CTRL + C`

```
source newrl-venv/bin/activate
scripts/install.sh mainnet
scripts/start.sh mainnet --fullnode
```

