# GURUFIN Full Node Installation Guide

GURUFIN chain full node is a node that validates blocks and transactions. <br>
This document guides the installation and operation of a full node of the GURUFIN chain.

## Install gaiad & gaiacli
 Describes how to install 'gaiad' and 'gaiacli' for node operation on your system.

### Install Golang
Follow the official [Go document](https://golang.org/doc/install) to install 'go'.<br>
Follow the instructions below to set up the environment of '$GOPATH' and '$PATH'.
```bash
mkdir -p $HOME/go/bin
echo "export GOPATH=$HOME/go" >> ~/.bash_profile
echo "export PATH=\$PATH:\$GOPATH/bin" >> ~/.bash_profile
echo "export GO111MODULE=on" >> ~/.bash_profile
source ~/.bash_profile
```
### Install Binaries
```bash
$ git clone https://github.com/gurufin2021/guru.git
$ git clone https://github.com/gurufin2021/cosmos-sdk.git
$ cd guru && vi go.mod
```
Open the go.mod file and modify the part below to the path of the downloaded cosmos-sdk.<br>
<go.mod>
```
replace github.com/cosmos/cosmos-sdk v0.37.15 => /cosmos_sdk_download_path
```
```bash
$ make install
```
If you follow the above procedure, the binary 'gaiad' and 'gaiacli' will be installed $GOPATH/bin. <br>
Make sure that the installation is successful.

```bash
$ gaiad version
$ gaiacli version
```
## Set up a full node
Create a home directory (node_home_directory) to operate a full node.<br>
You also create an environment variable settings file separately.
```bash
$ mkdir <node_home_directory> && cd <node_home_diectory>
$ vi env.sh && chmod +x ./env.sh
```
<env.sh file>
```bash
#!/bin/bash
CHAINID="gfin-guru"
MONIKER="my_node_name"   
GAIAD_HOME="<node_home_directory>/gaiad"
GAIACLI_HOME="<node_home_directory>/gaiacli"

# GURUFIN(Main Net) SEED Node Info
IP="43.200.69.12"
VALID="21d97d93f17a37878a2403ad63725a8b8e1bcf2c"
```
Next, create a node initialization configuration file.
```bash
$ ./env.sh
# Gaiacli Settings
$ gaiacli config chain-id ${CHAINID} --home=${GAIACLI_HOME}
$ gaiacli config indent true --home=${GAIACLI_HOME}
$ gaiacli config output json --home=${GAIACLI_HOME}
$ gaiacli config trust-node true --home=${GAIACLI_HOME}

# Chain initialization
$ gaiad init ${MONIKER} --chain-id=${CHAINID} --home=${GAIAD_HOME}

# Download and copy genesis.json file
$ wget https://doc.gurufin.io/guru_genesis.tar.gz && tar xvfz ./guru_genesis.tar.gz
$ cp ./genesis.json ${GAIAD_HOME}/config
```
Next, modify the settings file.
```bash
SEEDS="\"${VALID}@${IP}:26656\""
$ mv ${GAIAD_HOME}/config/config.toml ${GAIAD_HOME}/config/config.toml.bak
$ awk -v val=$SEEDS '/^seeds/{gsub($3, val)};{print}' ${GAIAD_HOME}/config/config.toml.bak > ${GAIAD_HOME}/config/config.toml

# Modify settings file parameters
$ sed -i "s/^persistent_peers = \"\"/persistent_peers = $SEEDS/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^addr_book_strict = true/addr_book_strict = false/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^max_packet_msg_payload_size = 1024/max_packet_msg_payload_size = 10240/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^grpc_max_open_connections = 900/grpc_max_open_connections = 10000/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^max_open_connections = 900/max_open_connections = 10000/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^size = 5000/size = 10000/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^max_txs_bytes = 1073741824/max_txs_bytes = 3221225472/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^timeout_commit = \"5s\"/timeout_commit = \"3s\"/" ${GAIAD_HOME}/config/config.toml
$ sed -i "s/^prometheus = false/prometheus = true/" ${GAIAD_HOME}/config/config.toml
```
Next, download the block data.
```bash
$ cd ${GAIAD_HOME}/gaiad
$ wget https://doc.gurufin.io/guru_data_20230517.tar.gz && tar xvfzp ./guru_data_20230517.tar.gz 
```
### Run a full node
**Note** it is convenient to manage the script contents below by running 
it using the [pm2](https://pm2.keymetrics.io/) daemon management program.
```bash
$ gaiad start \
  --home=${GAIAD_HOME} \
  --rpc.laddr="tcp://0.0.0.0:26657" \
  --p2p.laddr="tcp://0.0.0.0:26656" \
  --trace
```
### Run a LCD (Light Client Daemon), a local REST server
```bash
$ gaiacli rest-server \
--home=${GAIACLI_HOME} \
--chain-id=${CHAINID} \
--trust-node \
--laddr tcp://0.0.0.0:1317 \
--node tcp://127.0.0.1:26657 \
--max-open 9999 \
--trace
```
### GURUFIN Mainnet Explorer
**[https://scan.gurufin.io/](https://scan.gurufin.io/)**


