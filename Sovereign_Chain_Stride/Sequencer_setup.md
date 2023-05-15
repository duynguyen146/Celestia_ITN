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
