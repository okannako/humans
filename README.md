![Fu0MyzUXwAECMfY](https://user-images.githubusercontent.com/73176377/235451314-0b18ea13-93de-4c1a-8bfc-2a277fbf14c1.jpg)

- The Friction incentivised testnet >>> https://friction.humans.ai, Humans.ai's non-public test, has begun. In this testnet, I am sharing the steps related to node installation and how I do it step by step below.But before these steps, those selected by the team performed gentx operations and genesis was published.

## System Requirements
- CPU: 4 Cores
- Memory: 8 GB RAM
- Disk: 250 GB NVME
- Bandwidth: 1 Gbps for Download/100 Mbps for Upload

## System Update
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

## Installing Go
```
ver="1.20"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

- After updating and installing Go, we install the latest binary with the following codes.

## Binary Installation
```
cd $HOME
rm -rf ~/humans
git clone https://github.com/humansdotai/humans
cd humans
git checkout tags/v0.1.0
make install
```

``humansd version --long`` the version is checked with the code.

```
commit: baab99e300d97ef17ee7ff33cb59e183d1ece2b2
cosmos_sdk_version: v0.46.11
go: go version go1.20 linux/amd64
name: humans
server_name: humansd
version: 0.1.0
```

- After the version check, we continue operations.

## Configuration Setting
```
humansd config chain-id humans_3000-1
```

## Initialize
```
MONIKER="Your Moniker"
humansd init $MONIKER --chain-id humans_3000-1
```

## Genesis
```
wget -O $HOME/.humansd/config/genesis.json "https://raw.githubusercontent.com/humansdotai/testnets/master/friction/genesis.json"
```

## Add Seeds
```
PEERS="4762fa22edb91acd78010026f8da5fb71e174acb@142.165.207.19:26656,1c8629b474c9e0f0b24e81684992448963cf8d11@65.109.84.103:26656,b52e0c6553fb42b5899b93dd584959db2bb23422@95.217.200.36:26656,b9767aa2312748caaf67425890768d85186b69b1@5.9.87.205:26656,054c81f412213c90e77b295d6061a4a4b34d8f8f@141.95.99.214:26656,b1639fb460c9f55bb3acc3006df64ac5013f1412@91.194.30.203:26656,8c3c25fa9cd6289d0342cf0e42916f381a1fd671@78.46.64.59:26656,9da3f749f0b9b5180399d2099e9cb4f7b5d2ccaf@85.239.240.45:26656,36366c17222568d14e29e10c6678ba51103dc182@81.30.157.35:26656,df5cb643d8aeade8ef288a3dd90e4fd8220954ba@162.55.243.211:26656,78a8721148c5f30a87ab474d549618c33b69862e@178.23.126.70:26656,36f956fa2fe317a5d3163d0b6c7b104e33aa62e9@103.180.28.79:26656,0ab5a0c4fd3d85462da5fa149f3eb6d5702a4f32@118.193.37.229:26656,ca1e46048e4a9b65d60bc9e749aa431401f34ed7@144.76.45.59:26656,acce1a823a281b371bb9167180f9c80422071ece@143.110.190.223:26656,946b549550e9c564193bf4c963d84b17e5415a50@136.243.136.241:26656,ce7eddddd4bfd1c47f995f745d409554b2361075@202.61.236.219:26656,19230fad7145e6fe80566a72f66b9ca3ec3f04d5@212.47.234.144:26656,34fbb5fb1d5ec684238c9a6911e6587b00b6bf63@46.4.23.42:26656,be5158df5152ec7e6a4eca04c89e40494d19927c@51.79.101.159:26656,d0793e44dd807c1533da68d78db7bd0d6975fd57@65.109.82.112:26656,ab8e65e4d71c016501515a48bdcf70d6ec02ec1b@162.19.31.150:26656,6f6a90701f3cc459a9007456fa84abf647a68ef5@159.69.69.183:26656"
sed -i -e s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.humansd/config/config.toml
```

## Pruning (Optional)
```
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.humansd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.humansd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"40\"/" $HOME/.humansd/config/app.toml
```

## Gas Setting
```
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0aheart"/g' $HOME/.humansd/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.humansd/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.humansd/config/config.toml
```

## Create Service
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humansd Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## Start Node
```
sudo systemctl daemon-reload && sudo systemctl enable humansd && sudo systemctl restart humansd
```
## Log Control
```
journalctl -u humansd -f
```

## Create a New Wallet
```
humansd keys add walletname
```

## Restore Wallet
```
humansd keys add walletname --recover
```

- After this step, whether by comparing logs from explorer (https://explorer.humans.zone/humans-testnet) or when you enter the code at the bottom, when true>false, it means that you have been synchronized to the system, and then we created a validator with the following code.

```
humansd status 2>&1 | jq .SyncInfo
```

## Create Validator
```
humansd tx staking create-validator \
--amount 1000000aheart \
--from walletname \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(humansd tendermint show-validator) \
--moniker "MonikerName" \
--identity "" \
--details "I love blockchain ❤️" \
--chain-id humans_3000-1 \
--gas auto --gas-adjustment 1.5 \
-y
```

## Delete Node Files
```
sudo systemctl stop humansd
sudo systemctl disable humansd
sudo rm -rf /etc/systemd/system/humansd.service
sudo rm -rf $HOME/humans
sudo rm -rf $HOME/.humansd
```







