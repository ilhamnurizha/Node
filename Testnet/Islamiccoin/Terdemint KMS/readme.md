# [Islamiccoin](https://docs.haqq.network/guides/kms/kms.html) Create High Availability with Tendermint KMS

[Terndermint KMS](https://github.com/iqlusioninc/tmkms#about) is Key Management System for Tendermint applications such as Cosmos Validators.
This repository contains tmkms, a key management service intended to be deployed in conjunction with Tendermint applications (ideally on separate physical hosts) which provides the following:

- High-availability access to validator signing keys
- Double-signing prevention even in the event the validator process is compromised
- Hardware security module storage for validator keys which can survive host compromise

So lets start to begin.

# [Run Your Full Node and Create Validator First](https://github.com/ilhamnurizha/Node/edit/main/Testnet/Islamiccoin/readme.md)

## Prerequisites

We recommended you to run KMS service in a separate machine. Becasue this service work for just in case if your another validators down, you have a backup validator.  </br>

| Hardware |	Chunk-Only Producer Specification |
| -------- | ----------------------------------   |
| OS       | Linux Ubuntu 20.04         |
| CPU Architectures      | X86_64        |

## Install All Dependencies

Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

GCC
```
sudo apt update -y
sudo apt install git build-essential ufw curl jq snapd --y
```

Libusb
```
apt install libusb-1.0-0-dev
export RUSTFLAGS=-Ctarget-feature=+aes,+ssse3
```

## Install & Setup TMKMS

```
cd $HOME
git clone https://github.com/iqlusioninc/tmkms.git
cd $HOME/tmkms
cargo install tmkms --features=softsign
tmkms init config
tmkms softsign keygen ./config/secrets/secret_connection_key
```

## Copy your validator keys from Your Node Server

```
rsync -Pavz root@(your_node_ip):~/.haqqd/config/priv_validator_key.json ~/tmkms/config/secrets
```
  Change your_node_ip with your Node Public IP Address.
  
## import validator key into tmkms 

```
tmkms softsign import $HOME/tmkms/config/secrets/priv_validator_key.json $HOME/tmkms/config/secrets/priv_validator_key
```

## Config tmkms
```
nano $HOME/tmkms/config/tmkms.toml
```
```
# Tendermint KMS configuration file

## Chain Configuration

### Cosmos Hub Network

[[chain]]
id = "haqq_53211-1"
key_format = { type = "cosmos-json", account_key_prefix = "haqqpub", consensus_key_prefix = "haqqvalconspub" }
state_file = "/root/tmkms/config/state/priv_validator_state.json"

## Signing Provider Configuration

### Software-based Signer Configuration

[[providers.softsign]]
chain_ids = ["haqq_53211-1"]
key_type = "consensus"
path = "/root/tmkms/config/secrets/priv_validator_key"

## Validator Configuration

[[validator]]
chain_id = "haqq_53211-1"
addr = "tcp://123.456.32.123:688" # your validator node ip and port
secret_key = "/root/tmkms/config/secrets/secret_connection_key"
protocol_version = "v0.34"
reconnect = true
```

