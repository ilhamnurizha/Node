<p align="center">
  <img height="150" height="auto" src="https://user-images.githubusercontent.com/38981255/187036471-e23ab080-2e03-46b7-8513-23e1f6612b4a.png">
</p>

# [Islamiccoin](https://islamiccoin.net/) Testedge Haqq Node Validator Network


## Prerequisites

Minimum Hardware Requirement :

| Hardware |	Chunk-Only Producer Specification |
| -------- | ----------------------------------   |
| CPU      | 4 or more physical CPU cores         |
| RAM      | At least 32GB of memory (RAM)        |
| Storage  | At least 500GB of SSD disk storage   |
| Network  | At least 100mbps network bandwidth   |

Official Docs: [Haqq](https://docs.haqq.network/guides/validators/setup.html) </br>
Official Contact: [Discord](https://discord.gg/hG9J6zsY) [Twitter](https://twitter.com/Islamic_Coin) [Telegram](https://t.me/islamiccoin_community) 

## Setting up vars
```
NODENAME=moniker_name
```
moniker_name filled with your Name

```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export HAQQ_CHAIN_ID=haqq_53211-1" >> $HOME/.bash_profile
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
git clone https://github.com/haqq-network/haqq.git && cd haqq
make install

```

## Config Node

```
haqqd config chain-id $HAQQ_CHAIN_ID
haqqd config keyring-backend test
```

## Init App

```
haqqd init $NODENAME --chain-id $HAQQ_CHAIN_ID
```

## Download genesis and addrbook

```
curl -OL https://storage.googleapis.com/haqq-testedge-snapshots/genesis.json
mv genesis.json $HOME/.haqqd/config/genesis.json
haqqd validate-genesis
```

## State Sync
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0aISLM\"/" $HOME/.haqqd/config/app.toml
curl -OL https://raw.githubusercontent.com/haqq-network/testnets/main/TestEdge/state_sync.sh
chmod +x state_sync.sh && ./state_sync.sh
```

## Create Service

```
sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqq
After=network-online.target

[Service]
User=$USER
ExecStart=$(which haqqd) start --home $HOME/.haqqd
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
sudo systemctl enable haqqd
sudo systemctl restart haqqd
```
## Reload Variable

```
source $HOME/.bash_profile
```

## Check your Node Sync

```
haqqd status 2>&1 | jq .SyncInfo
```

Wait until "catching_up": false

## Check your log

```
sudo journalctl -u haqqd -f -o cat
```

## Create your Funded Wallet Address

```
haqqd keys add $WALLET
```
Save your mnemonic to safe place, don't give it to another, mnemonic used for import to your metamask later


Or you can recover your wallet

```
haqqd keys add $WALLET --recover
```

Check your Funded wallet address 

```
haqqd keys list
```

## Save your wallet info

```
HAQQ_WALLET_ADDRESS=$(haqqd keys show $WALLET -a)
HAQQ_VALOPER_ADDRESS=$(haqqd keys show $WALLET --bech val -a)
echo 'export HAQQ_WALLET_ADDRESS='${HAQQ_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export HAQQ_VALOPER_ADDRESS='${HAQQ_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Get your faucet Haqq Token

- Import your wallet into private key:
```
haqqd keys unsafe-export-eth-key $WALLET --keyring-backend file
```

- Import your private key into metamask
- visit page https://testedge.haqq.network/
- Connect and sign with metamask wallet 
- Switch to TestNow, approve and switch to Haqq Testnet Network 
- Login with your Gihtub Account
- Request Token

```
Network Title: Haqq Network Testnet
RPC URL: https://rpc.eth.testedge.haqq.network/
Chain ID: 53211
SYMBOL: ISLM
```

## Check your balance

```
haqqd query bank balances $HAQQ_WALLET_ADDRESS
```

## Create yuor validator
```
haqqd tx staking create-validator \
  --amount 1000000aISLM \
  --pubkey $(haqqd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $HAQQ_CHAIN_ID \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas-prices="0.025aISLM" \
  --from $WALLET \
  --node https://rpc.tm.testedge.haqq.network:443
  --keyring-backend file
```

## Self Delegated or to Other Validator
```
haqqd tx staking delegate YOUR_VALOPER_ADDRESS 10000000aISLM --from=$WALLET --chain-id $HAQQ_CHAIN_ID --gas-prices=0.025aISLM
```

## Check Active Validator

```
haqqd query tendermint-validator-set | grep "$(haqqd tendermint show-address)"
```

## Check Active Validator

```
haqqd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
