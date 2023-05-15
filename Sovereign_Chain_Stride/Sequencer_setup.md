## 1. Introduction
- Deploy a sovereign rollup chain on top of Celestia Blockspacerace network is one of bonus tasks of Celestia ITN program.
- In this part, i will show how to convert and build a rollup chain from a existing Cosmos SDK L1 - `Stride` 

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

- Add genesis account
```
strided keys add wallet1 --keyring-backend test
strided keys add wallet2 --keyring-backend test
```

- Adjust parameter in `genesis.json` following chain information
```
jq ".app_state[\"staking\"][\"params\"][\"bond_denom\"]=\"$DENOM\"" $HOME/.stride/config/genesis.json > $HOME/.stride/config/tmp_genesis.json && mv $HOME/.stride/config/tmp_genesis.json $HOME/.stride/config/genesis.json

jq ".app_state[\"crisis\"][\"constant_fee\"][\"denom\"]=\"$DENOM\"" $HOME/.stride/config/genesis.json > $HOME/.stride/config/tmp_genesis.json && mv $HOME/.stride/config/tmp_genesis.json $HOME/.stride/config/genesis.json

jq ".app_state[\"gov\"][\"deposit_params\"][\"min_deposit\"][0][\"denom\"]=\"$DENOM\"" $HOME/.stride/config/genesis.json > $HOME/.stride/config/tmp_genesis.json && mv $HOME/.stride/config/tmp_genesis.json $HOME/.stride/config/genesis.json

jq ".app_state[\"inflation\"][\"params\"][\"mint_denom\"]=\"$DENOM\"" $HOME/.stride/config/genesis.json > $HOME/.stride/config/tmp_genesis.json && mv $HOME/.stride/config/tmp_genesis.json $HOME/.stride/config/genesis.json

jq ".app_state[\"mint\"][\"params\"][\"mint_denom\"]=\"$DENOM\"" $HOME/.stride/config/genesis.json > $HOME/.stride/config/tmp_genesis.json && mv $HOME/.stride/config/tmp_genesis.json $HOME/.stride/config/genesis.json

jq '.consensus_params["block"]["max_gas"]="10000000"' $HOME/.stride/config/genesis.json > $HOME/.stride/config/tmp_genesis.json && mv $HOME/.stride/config/tmp_genesis.json $HOME/.stride/config/genesis.json
```

- Allocate genesis accounts (cosmos formatted addresses)
```
strided add-genesis-account wallet1 100000000000000000000000000${DENOM} --keyring-backend test
strided add-genesis-account wallet2 100000000000000000000000000${DENOM} --keyring-backend test
```

- Sign, collect genesis transaction then validate genesis json file
```
strided gentx wallet1 1000000000000000000000${DENOM} --keyring-backend test --chain-id $CHAIN_ID
strided collect-gentxs 
strided validate-genesis 
```

- Setup information related to DA node
```
BASE_URL="http://localhost:26659" # If DA node is on different server with sequencer node, change `localhost` to `public ip` of DA node
NAMESPACE="YOUR_NAME_SPACE_ID" 
ROLLKIT_BLOCKTIME="3s" 
ROLLKIT_DA_BLOCKTIME="12s"
DA_BLOCK_HEIGHT=$(curl -s https://rpc-blockspacerace.pops.one/block | jq -r '.result.block.header.height')
```

- Create systemD service
```
sudo tee /etc/systemd/system/sequencer.service > /dev/null <<EOF
[Unit]
Description=Stride Sequencer
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start --rollkit.aggregator true \
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
            --p2p.seed_mode             
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable sequencer
```

- Start sequencer node
```
sudo systemctl restart sequencer
```

- Check log of sequencer node
```
sudo journalctl -u sequencer -f -o cat
```

### 2.5 Reset Rollup chain `Stride`
- If you wanna reset the rollup Sequencer, please follow 
```
DA_BLOCK_HEIGHT=$(curl -s https://rpc-blockspacerace.pops.one/block | jq -r '.result.block.header.height')
sed -i.bak -e  "s/rollkit.da_start_height [0-9]* /rollkit.da_start_height $DA_BLOCK_HEIGHT /g; s/\"fee\":[0-9]*,/\"fee\":1000,/g" /etc/systemd/system/sequencer.service
sudo systemctl daemon-reload
sudo systemctl restart sequencer
sleep 10;


DA_BLOCK_HEIGHT=$(curl -s https://rpc-blockspacerace.pops.one/block | jq -r '.result.block.header.height')
sed -i.bak -e  "s/rollkit.da_start_height [0-9]* /rollkit.da_start_height $DA_BLOCK_HEIGHT /g; s/\"fee\":[0-9]*,/\"fee\":100,/g" /etc/systemd/system/sequencer.service
sudo systemctl daemon-reload
sudo systemctl restart sequencer
```

## 3. Connect your rollup chain to Kelpr
- Expose API and RPC
```
sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.stride/config/config.toml
```

- Restart sequencer
```
sudo systemctl restart sequencer
```

- Open a text file, then input below code
```
window.keplr.experimentalSuggestChain({
chainId: "stride_local",
chainName: "stride rollup",
rpc: "http://95.216.21.32:26657",
rest: "http://95.216.21.32:1317",
bip44: {
    coinType: 118,
},
bech32Config: {
    bech32PrefixAccAddr: "stride",
    bech32PrefixAccPub: "stride" + "pub",
    bech32PrefixValAddr: "stride" + "valoper",
    bech32PrefixValPub: "stride" + "valoperpub",
    bech32PrefixConsAddr: "stride" + "valcons",
    bech32PrefixConsPub: "stride" + "valconspub",
},
currencies: [ 
    { 
        coinDenom: "STRD", 
        coinMinimalDenom: "ustrd", 
        coinDecimals: 6, 
        coinGeckoId: "ustrd", 
    }, 
],
feeCurrencies: [
    { 
        coinDenom: "STRD", 
        coinMinimalDenom: "ustrd", 
        coinDecimals: 6, 
        coinGeckoId: "ustrd", 
    },
],
stakeCurrency: { 
        coinDenom: "STRD", 
        coinMinimalDenom: "ustrd", 
        coinDecimals: 6, 
        coinGeckoId: "ustrd", 
},
coinType: 118,
gasPriceStep: {
    low: 0.0,
    average: 0.025,
    high: 0.03,
},
});
```
