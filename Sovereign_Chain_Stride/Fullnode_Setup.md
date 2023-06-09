## 1. Introduction
- Deploy a sovereign rollup chain on top of Celestia Blockspacerace network is one of bonus tasks of Celestia ITN program.
- In this part, i will show how to convert and deploy a fullnode of a rollup chain from a existing Cosmos SDK L1 - `Stride` 

## 2. Deployment
### 2.1 Install Prerequisites
```
sudo apt-get update && sudo apt upgrade -y
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

### 2.2 Install Go version 1.20.x
```
ver="1.19.8"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

### 2.3 Clone repo and compile binary 
```
cd $HOME
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout v9.0.0
go mod edit -replace github.com/cosmos/cosmos-sdk=github.com/rollkit/cosmos-sdk@v0.46.7-rollkit-v0.7.3-no-fraud-proofs
go mod edit -replace github.com/tendermint/tendermint=github.com/celestiaorg/tendermint@v0.34.22-0.20221202214355-3605c597500d
go mod tidy
go mod download
make install
```

### 2.4 Setup Rollup chain `Stride`
- Initialize the rollup chain
```
CHAIN_ID="stride_local"
MONIKER="sequencer"
DENOM="ustrd"

strided config keyring-backend test
strided config chain-id $CHAIN_ID
strided init $MONIKER --chain-id $CHAIN_ID
```

- Setup information related to DA node
```
BASE_URL="http://localhost:26659" # If DA node is on different server with rollup full node, change `localhost` to `public ip` of DA node
NAMESPACE="YOUR_NAME_SPACE_ID" # Using the same `namespaceid` configured during setup your sequencer node on rollupchain
ROLLKIT_BLOCKTIME="3s" 
ROLLKIT_DA_BLOCKTIME="12s"
DA_BLOCK_HEIGHT=$(curl -s https://rpc-blockspacerace.pops.one/block | jq -r '.result.block.header.height')
SEQUENCER_IP="YOUR_SEQ_IP" # Fill public ip of your sequencer node
SEQUENCER_ID=$(curl -s http://$SEQUENCER_IP:26657/status | jq -r .result.node_info.id)
```

- Download `genesis.json` from sequencer server to fullnode server
```
scp username@$SEQUENCER_IP:$HOME/.stride/config/genesis.json $HOME/.stride/config/genesis.json 
```

- Create systemD service
```
sudo tee /etc/systemd/system/fullnode.service > /dev/null <<EOF
[Unit]
Description=Stride fullnode
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start \
            --rollkit.block_time ${ROLLKIT_BLOCKTIME} \
            --rollkit.da_block_time ${ROLLKIT_DA_BLOCKTIME} \
            --rollkit.da_layer celestia \
            --rollkit.da_config='{"base_url":"$BASE_URL","timeout":60000000000,"fee":100,"gas_limit":100000}' \
            --rollkit.namespace_id ${NAMESPACE}  \
            --rollkit.da_start_height ${DA_BLOCK_HEIGHT} \
            --p2p.laddr "0.0.0.0:26656" \
            --rpc.laddr "tcp://0.0.0.0:26657" \
            --grpc.address "0.0.0.0:9090" \
            --grpc-web.address "0.0.0.0:9091" \
            --p2p.seeds "${SEQUENCER_ID}@${SEQUENCER_IP}:26656"
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable fullnode
```

- Start rollup full node
```
sudo systemctl restart fullnode
```

- Check log of full node
```
sudo journalctl -u fullnode -f -o cat
```
