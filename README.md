**Manual Installation**

Official Documentation
```
Recommended Hardware: 6 Cores, 16GB RAM, 400GB of storage (NVME), 100 Mb/s
```

**install dependencies, if needed**
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**install go, if needed**
```
cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

**set vars**
```
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export STORY_CHAIN_ID="odyssey-0"" >> $HOME/.bash_profile
echo "export STORY_PORT="52"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
**download binaries**
```
cd $HOME
wget -O geth https://github.com/piplabs/story-geth/releases/download/v0.11.0/geth-linux-amd64
chmod +x $HOME/geth
mv $HOME/geth ~/go/bin/
[ ! -d "$HOME/.story/story" ] && mkdir -p "$HOME/.story/story"
[ ! -d "$HOME/.story/geth" ] && mkdir -p "$HOME/.story/geth"
```

**install Story**
```
cd $HOME
rm -rf story
git clone https://github.com/piplabs/story
cd story
git checkout v0.13.2
go build -o story ./client 
mv $HOME/story/story $HOME/go/bin/
```

**init story app**
```
story init --moniker test --network odyssey
```

**set seeds and peers**
```
SEEDS="434af9dae402ab9f1c8a8fc15eae2d68b5be3387@story-testnet-seed.itrocket.net:29656"
PEERS="c2a6cc9b3fa468624b2683b54790eb339db45cbf@story-testnet-peer.itrocket.net:26656,0bf2a03400f7443e3c27d566fdedb6f95b7a35af@149.50.124.195:26156,12d299d9556a6414e8228c6320b345c995f33f30@65.109.25.222:13656,cd9250f1f07753169ef9da942b6cdc449618c9c0@185.187.170.53:656,39fda3b8a88377f7e428b3e5fd0cf655859c5a35@149.50.107.108:26156,6fd813318b6c267af8160eabd9143844070e04c2@149.50.109.121:26156,89f06acb78296b3abcde19c31a5be31e688a60d0@100.42.183.228:656,a3917959725d34a50626557e50ad82083c6f4f06@149.50.114.229:26156,7d67b4236c03f426e281a32cb71ffce8139c3679@37.60.236.168:656,04006dd629dc18d16ab915537e97eebcfbece34f@149.50.119.44:26156,11dc2be345d15741dc747d6a6e5bcdc90d812756@100.42.183.223:656,26cac71cbc88d50d08d363aa889c6a52e07973d0@100.42.183.131:656,5baa971ad0fa830ae8afca410868afc0a39e5d2f@157.173.121.179:26656,ea1abe78850bfa58a3992f88fceb7b6131d335ad@31.220.74.227:26656,e1866a76129a740e0ccf8b1f9ca89917539c0785@193.34.214.9:26156,bf55695f3616be7e3133d457778d5d07fd96b28b@45.85.146.254:26656,0a0c5806679042ce8c8f0c3030826dbd75300627@65.109.113.219:26656,0be25c290c9827799decbbdab699274c179bd25a@149.50.118.200:26156,2ac7e5089cb547df5444105d5045892fc34ace06@135.181.139.234:26656,d8ec37ec649ef01fdc7218d489c9e0481ded2334@65.109.62.39:52656,f4824bf54020da9d9cc366f53e7552f2084fbd2b@149.50.124.159:26156,92ed2bd8e6c9d98f73abc16cfb0330c0e45a0d82@62.84.178.14:26656,8c1ec13ec9cbcbfad914f34351fdc6844fbe30c7@193.34.214.28:26156,c1d1eef2e94c2c75fb0172ed5dcde99c125686da@149.50.114.162:26156,434af9dae402ab9f1c8a8fc15eae2d68b5be3387@65.109.19.111:29656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml
```
**download genesis and addrbook**
```
wget -O $HOME/.story/story/config/genesis.json https://server-7.itrocket.net/testnet/story/genesis.json
wget -O $HOME/.story/story/config/addrbook.json  https://server-7.itrocket.net/testnet/story/addrbook.json
```
**set custom ports in story.toml file**
```
sed -i.bak -e "s%:1317%:${STORY_PORT}317%g;
s%:8551%:${STORY_PORT}551%g" $HOME/.story/story/config/story.toml
```

**set custom ports in config.toml file**
```
sed -i.bak -e "s%:26658%:${STORY_PORT}658%g;
s%:26657%:${STORY_PORT}657%g;
s%:26656%:${STORY_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STORY_PORT}656\"%;
s%:26660%:${STORY_PORT}660%g" $HOME/.story/story/config/config.toml
```

**enable prometheus and disable indexing**
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.story/story/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.story/story/config/config.toml
```

**create geth servie file**
```
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/geth --odyssey --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 0.0.0.0 --http.port ${STORY_PORT}545 --authrpc.port ${STORY_PORT}551 --ws --ws.api eth,web3,net,txpool --ws.addr 0.0.0.0 --ws.port ${STORY_PORT}546
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

**create story service file**
```
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Service
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/.story/story
ExecStart=$(which story) run

Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
**download snapshots**

**backup priv_validator_state.json**
```
cp $HOME/.story/story/data/priv_validator_state.json $HOME/.story/story/priv_validator_state.json.backup
```
**remove old data and unpack Story snapshot**
```
rm -rf $HOME/.story/story/data
curl https://server-7.itrocket.net/testnet/story/story_2025-03-11_2878408_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/story
```

# restore priv_validator_state.json
mv $HOME/.story/story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json

# delete geth data and unpack Geth snapshot
rm -rf $HOME/.story/geth/odyssey/geth/chaindata
mkdir -p $HOME/.story/geth/odyssey/geth
curl https://server-7.itrocket.net/testnet/story/geth_story_2025-03-11_2878408_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/geth/odyssey/geth

# enable and start geth, story
sudo systemctl daemon-reload
sudo systemctl enable story story-geth
sudo systemctl restart story-geth && sleep 5 && sudo systemctl restart story

# check logs
journalctl -u story -u story-geth -f
Automatic Installation
source <(curl -s https://itrocket.net/api/testnet/story/story-autoinstall/)
Cosmovisor Setup
Install go, if needed:

cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
Install and init Cosmovisor:

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
echo "export DAEMON_NAME="story"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.story/story"" >> $HOME/.bash_profile
source $HOME/.bash_profile
cosmovisor init $(which story)
Create a directory and download the current version of story:

mkdir -p $HOME/.story/story/cosmovisor/upgrades/v0.13.0/bin
wget -O $HOME/.story/story/cosmovisor/upgrades/v0.13.0/bin/story https://github.com/piplabs/story/releases/download/v0.13.0/story-linux-amd64
chmod +x $HOME/.story/story/cosmovisor/upgrades/v0.13.0/bin/story
Update service file:

sudo tee /etc/systemd/system/story.service > /dev/null << EOF
[Unit]
Description=story node service
After=network-online.target

[Service]
User=$USER
Environment="DAEMON_NAME=story"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/data"
ExecStart=$(which cosmovisor) run run
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
Enable and start Story using Cosmovisor:

sudo systemctl daemon-reload
sudo systemctl enable story
sudo systemctl restart story && sudo journalctl -u story -f
Congrats, you are now using Cosmovisor! 🎊
Create validator
View your validator key

story validator export
Export EVM private key

story validator export --export-evm-key
View EVM private key and make a key backup

cat $HOME/.story/story/config/private_key.txt
Use this private key to import your account into a wallet, e.g. Metamask or Phantom. Add the odyssey testnet to your wallet via faucet. Then, copy your 'EVM address' from the wallet and request $IP tokens. Now you can see the balance and make transactions in the wallet app.

Before creating a validator, wait for your node to get fully synced. Once "catching_up" is "false", move on to the next step
curl localhost:$(sed -n '/\[rpc\]/,/laddr/ { /laddr/ {s/.*://; s/".*//; p} }' $HOME/.story/story/config/config.toml)/status | jq
Create validator
story validator create --stake 1500000000000000000000 --moniker $MONIKER --chain-id 1516 --private-key $(cat $HOME/.story/story/config/private_key.txt | grep "PRIVATE_KEY" | awk -F'=' '{print $2}')
Remember to backup your validator priv_key from here:

cat $HOME/.story/story/config/priv_validator_key.json
Firewall rules
Configure firewall rules:

sudo ufw allow 30303/tcp comment geth_p2p_port
sudo ufw allow 26656/tcp comment story_p2p_port
Delete node
sudo systemctl stop story story-geth
sudo systemctl disable story story-geth
rm -rf $HOME/.story
sudo rm /etc/systemd/system/story.service /etc/systemd/system/story-geth.service
sudo systemctl daemon-reload
