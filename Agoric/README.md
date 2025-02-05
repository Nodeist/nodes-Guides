# Agoric mainnet guide
![AG (2)](https://user-images.githubusercontent.com/44331529/181192613-feff0b48-086b-41f3-9540-152ff4a08694.png)
![AG (1)](https://user-images.githubusercontent.com/44331529/181192625-d034ab43-ba09-4636-8656-c3c6afd9975c.png)


[EXPLORER 1](https://agoric.explorers.guru/validators) \
[EXPLORER 2](https://explorer.postcapitalist.io/agoric/staking) \
[EXPLORER 3](https://agoric.bigdipper.live/validators?sort=votingPower&dir=-1)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| korellia  |   4| 8GB  | 160GB    |

### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.3 (one command)

    ver="1.18.3" && \
    cd $HOME && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
    source $HOME/.bash_profile && \
    go version
    
# Build 27.07.22
    export GIT_BRANCH=agoric-6
    git clone https://github.com/Agoric/ag0
    cd ag0
    git checkout agoric-upgrade-6
    make build && make install
    . $HOME/.bash_profile
    cp $HOME/ag0/build/ag0 /usr/local/bin
`ag0 version`
- HEAD-31c78ba3aa872b54c4de448763c5b8044b8f950c

      ag0 init <moniker> --chain-id agoric-3
    

## Create/recover wallet

    ag0 keys add <walletname>
    ag0 keys add <walletname> --recover

## Download Genesis

    curl https://main.agoric.net/genesis.json > $HOME/.agoric/config/genesis.json 

## Peers/Seeds

    seeds=""
    peers="2c03e71116d1a2f9ba39a63a97058fcdeabfe2be@159.148.31.233:26656,ef12448f0f8671a195ab38c590cac713ad703a8b@146.70.66.202:26656,320dd22ee85e2b68f891b670331eb9fec9dc419e@80.64.208.63:26656,f095bb53006ebddcbbf29c8df70dddcba6419e36@142.93.145.13:26656,0c370d803934e3273c61b2577a0c6e91b9f677e0@139.59.7.33:26656,c03f4e7fe0f4c081b14f6731e74aa89ff2d4c197@84.244.95.237:26656,8c30ee29afc4b77cf98222edcc3fe823cf1e8306@195.201.106.244:26656,b2285313e3411e3d5bcbee72e526108e6bd07da4@185.147.80.110:26656,68c9c4e8388ed6936ff147ffe6b9913e79328957@35.215.62.66:26656,99968808ecae7bc41b14df3bcb51b724ee5f782f@134.209.154.162:26656,2d352e7a97cef2a6b253906d3741efaee16b6af0@64.227.14.179:26656,5a6c74c824805c3e75cea44df019b69db8fb935a@142.132.149.55:26656,0464c8dded70d01f5ab50a8d6047a6b27ddf2ccd@84.244.95.232:26656,9cd93ebaa554e68990ecec234de74e848c7755e7@137.184.45.31:10003,f4b809dcf7004b8a30eaa4e9bb0a65164368b75a@49.12.165.122:26656,4d0953252dd26b5ff96292bd2a836bd8a77f4eed@159.69.63.222:26656,f554d57fd9326a90580483e23cab8d728bfb232a@78.46.84.150:26656,c84170667fcf54024b24f05b2f9dd6608570ac8c@157.90.35.145:28656,cb6ae22e1e89d029c55f2cb400b0caa19cbe5523@15.223.138.194:26603,1da72d9acd9c26a332c99e5e5f91b586f1ebc7c4@3.14.237.44:26656"
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.agoric/config/config.toml

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.agoric/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.agoric/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.agoric/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.agoric/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.agoric/config/config.toml
 
 
[SNAPSHOT](https://polkachu.com/tendermint_snapshots/agoric)
=

## Download addrbook

    wget -O $HOME/.agoric/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Agoric/addrbook.json"


# Create a service file

    tee /etc/systemd/system/agoricd.service > /dev/null <<EOF
    [Unit]
    Description=Agoric Cosmos daemon
    After=network-online.target
    [Service]
    # OPTIONAL: turn on JS debugging information.
    #SLOGFILE=.agoric/data/chain.slog
    User=$USER
    # OPTIONAL: turn on Cosmos nondeterminism debugging information
    #ExecStart=$(which ag0) start --log_level=info --trace-store=.agoric/data/kvstore.trace
    ExecStart=$(which ag0) start --log_level=info
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

## Start

    sudo systemctl daemon-reload && \ 
    sudo systemctl enable agoricd && \
    sudo systemctl restart agoricd && \
    sudo journalctl -u agoricd -f -o cat


### Create validator
    ag0 tx staking create-validator \
    --amount=1000000ubld \
    --broadcast-mode=block \
    --pubkey=`ag0 tendermint show-validator` \
    --moniker=<Moniker> \
    --commission-rate="0.1" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --from=<walletName> \
    --chain-id=agoric-3 \
    --gas-adjustment=1.4 -y


## Delete node
    sudo systemctl stop agoricd && \
    sudo systemctl disable agoricd && \
    rm /etc/systemd/system/agoricd.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf ag0 && \
    rm -rf .agoric && \
    rm -rf $(which ag0)

