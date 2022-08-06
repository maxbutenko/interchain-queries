## ICQ packets (option #1 of task #9 Stride)


The following steps assume that RelayerGo2 is running on your host (we copy the keys from there) and you are aware of your settings.

Download interchain-queries:

    cd
    wget https://github.com/Stride-Labs/interchain-queries/archive/refs/tags/v0.0.4.zip
    cd interchain-queries-0.0.4
    go build
    cp /root/interchain-queries-0.0.4/interchain-queries /usr/local/bin
    cd

Copy the keys from the relayer:

    mkdir -p /root/.icq
    cp -r /root/.relayer/keys /root/.icq/keys

Downloading the config:

    wget -O /root/.icq/config.yaml https://raw.githubusercontent.com/maxbutenko/interchain-queries/main/config.yaml

Highlight what needs to be changed:

    
    global: memo: **ReBoot#9655_ICQ_TEST** 
    default_chain: gaia 
    chains:
     gaia:
      key: **wallet**
       chain-id: GAIA
        rpc-addr: **http://192.168.0.61:23657** 
        grpc-addr: **http://192.168.0.61:23090** 
        account-prefix: cosmos 
        keyring-backend: test 
        gas-adjustment: 1.2 
        gas-prices: 0.01uatom 
        key-directory: /root/.icq/keys 
        debug: true 
        timeout: 20s block-timeout: "" 
        output-format: json 
        sign-mode: **direct** 
      stride: 
        key: **wallet** 
        chain-id: STRIDE-TESTNET-2
        rpc-addr: **http://127.0.0.1:26657**
        grpc-addr: **http://127.0.0.1:9090**
        account-prefix: stride 
        keyring-backend: test 
        gas-adjustment: 1.2 
        gas-prices: 0.01ustrd 
        key-directory: /root/.icq/keys 
        debug: true 
        timeout: 20s 
        block-timeout: "" 
        output-format: json 
        sign-mode: **direct** 
        cl: {}

**Memo** is not displayed in transactions, **wallet** is in /root/.icq/keys, **sign-mode** can be changed to **sync** (I haven’t noticed much difference yet).

Create a service:

    tee /etc/systemd/system/interchaind.service > /dev/null <<EOF
    [Unit]
    Description=InterChainQ
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=/usr/local/bin/interchain-queries run --home $HOME/.icq
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

Starting service:

    systemctl daemon-reload
    systemctl enable interchaind
    systemctl restart interchaind

Checking logs:

    journalctl -u interchaind -f -o cat

The binary is unusually laconic, so the logs will be "a teaspoon per hour", this is normal, after a while we see:

    Started InterChainQ. store/bank/key 
    Fetching client update for height height 183839 
    store/bank/key 
    Fetching client update for height height 183839 
    Requerying lightblock 
    Requerying lightblock
    Requerying lightblock 
    Requerying lightblock 
    Requerying lightblock 
    Fetching client update for height height 183839 
    Requerying lightblock 
    Fetching client update for height height 183839
    Send batch of 4 messages 
    1 ClientUpdate message 
    1 SubmitResponse message 
    1 ClientUpdate message 
    1 SubmitResponse message 
    Sent batch of 2 (deduplicated) messages

Thank you for your attention, good luck!
