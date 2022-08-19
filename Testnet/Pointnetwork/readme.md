<p align="center">
  <img height="150" height="auto" src="https://user-images.githubusercontent.com/38981255/185550018-bf5220fa-7858-4353-905c-9bbd5b256c30.jpg">
</p>

# #Point Network Testnet Incentivized


## Prerequisites

Minimum Hardware Requirement based on https://docs.evmos.org/validators/overview.html#hardware

| Hardware |	Chunk-Only Producer Specification |
| -------- | ---------------------------------- |
| CPU      | 4-Core CPU with AVX support        |
| RAM      |	8GB DDR4                          |
| Storage	 |  500GB SSD                         |

Official Docs: https://github.com/pointnetwork/point-chain/blob/xnet-triton/VALIDATORS.md#prerequisites

## Setting up vars
```
NODENAME=moniker_name
```
moniker_name filled with your Name

```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export EVMOS_CHAIN_ID=point_10721-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages

```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies

```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Install Go

```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries

```
cd $HOME
git clone https://github.com/pointnetwork/point-chain && cd point-chain
git checkout xnet-triton
make install
```

## Install Node

```
evmosd config chain-id $EVMOS_CHAIN_ID
evmosd config keyring-backend file
```

## Init App

```
evmosd init $NODENAME --chain-id $EVMOS_CHAIN_ID
```

## Download genesis and addrbook

```
wget https://raw.githubusercontent.com/pointnetwork/point-chain-config/main/testnet-xNet-Triton-1/config.toml
wget https://raw.githubusercontent.com/pointnetwork/point-chain-config/main/testnet-xNet-Triton-1/genesis.json
mv config.toml genesis.json ~/.evmosd/config/
```

### Validate:

```
evmosd validate-genesis
```

## Create Service

```
sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
[Unit]
Description=evmos
After=network-online.target

[Service]
User=$USER
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service

```
sudo systemctl daemon-reload
sudo systemctl enable evmosd
sudo systemctl restart evmosd && sudo journalctl -u evmosd -f -o cat
```

## Check your Node Sync

```
evmosd status 2>&1 | jq .SyncInfo
```

Wait until "catching_up": false


## Create your Funded Wallet Address

```
evmosd keys add validatorkey --keyring-backend file
```
Save your mnemonic to safe place, don't give it to another, mnemonic used for import to your metamask later


Or you can recover your wallet

```
evmosd keys add validatorkey --keyring-backend file --recover
```

Check your Funded wallet address 

```
evmosd keys list
```
address must evmosxxxxxxxxxx


## Import your Funded Wallet Address

Open your metamask and import your wallet with your mnemonic, or you can import via private keys with this command

```
evmosd keys unsafe-export-eth-key validatorkey --keyring-backend file
```


## Get your faucet Point Token

- Filled this form : https://pointnetwork.io/testnet-form 
- Filled with metamask address (it must 0x address) 
- Check with https://evmos.me/utils/tools for converting address info between evmos address (evmosxxxxx) and metamask address (0x)
- Wait faucet land, Point Network team will check who filled the form within 24 hours 
- Add RPC Point Network in Metamask

```
Network Title: Point XNet Triton
RPC URL: https://xnet-triton-1.point.space/
Chain ID: 10721
SYMBOL: XPOINT
```

## Check your balance

```
evmosd query bank balances $(evmosd keys show validatorkey | grep address: | cut -d ':' --complement -f 1)
```
when the faucet token land the balance shows 1024 XPOINT


## Check Your Validator Address

```
evmosd tendermint show-address
```

## Stake XPOINT and Create Validator

Make sure again SyncInfo status False and Balance available (1024 XPOINT)

```
evmosd tx staking create-validator \
--amount=1000000000000000000000apoint \
--pubkey=$(evmosd tendermint show-validator) \
--moniker=$NODENAME \
--chain-id=point_10721-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1000000000000000000000" \
--gas="400000" \
--gas-prices="0.025apoint" \
--from=validatorkey \
--keyring-backend file
```

## Check Your Validator Active

```
evmosd query tendermint-validator-set | grep "$(evmosd tendermint show-address)"
```

## Make sure you are not in jailled

```
evmosd query slashing signing-info $(evmosd tendermint show-validator)
```
