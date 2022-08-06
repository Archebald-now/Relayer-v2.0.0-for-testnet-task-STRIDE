# Set up a v2.0.0
(This guide was created in collaboration with goooodnes#8929 and Zuka#5870)
<br>

In current example we will learn how to set up IBC relayer between two cosmos chains

## Update system
```
     sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
     sudo apt install wget git make htop unzip -y
```
## Install GO 1.18.3 | ONE COMMAND
```
     cd $HOME && \
     ver="1.18.3" && \
     wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
     sudo rm -rf /usr/local/go && \
     sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
     rm "go$ver.linux-amd64.tar.gz" && \
     echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
     source $HOME/.bash_profile && \
     go version
```
## Make relayer home dir
```
     cd $HOME
     mkdir -p $HOME/.relayer/config
```

## Download go-v2 relayer
```
     git clone https://github.com/cosmos/relayer.git
     cd relayer && git checkout v2.0.0
     make install
 
```
## Make you own config for Relayer v2
```
     cd $HOME
     mkdir -p $HOME/.relayer/config
     
     MEMO=Your memo #archebald#2945
     
     KEYSTRIDE=Name of your key #stride
     KEYGAIA=Name of your key
     
     IPSTRIDE=IP your STRIDE node #164.68.125.90
     PORTSTRIDE=Port RPC your STRIDE node #10002

     IPGAIA=IP you GAIA node #164.68.125.90
     PORTGAIA=RPC port of your GAIA node #10004
```

# NEXT - To add the chain config files manually
```
$ rly chains add --url https://gist.githubusercontent.com/Archebald-now/3aef116b9dd67009600d8da1746dfe1f/raw/06f7e8959d5d9735576867ae723ca5c35f485aed/GAIA_config.json gaia
$ rly chains add --url https://gist.githubusercontent.com/Archebald-now/3aef116b9dd67009600d8da1746dfe1f/raw/06f7e8959d5d9735576867ae723ca5c35f485aed/STRIDE-TESTNET-2_config.json stride
```
after which you need to make changes in the configuration file
```nano /root/.relaer/config/config.json```
and replace rpc-addr with your knowledge

# NEXT - COPY ALL ONE COMAND TO TERMINAL
```

sudo tee $HOME/.relayer/config/config.yaml > /dev/null <<EOF
global:
    api-listen-addr: :5183
    timeout: 10s
    memo: $MEMO
    light-cache-size: 20
chains:
    GAIA:
        type: cosmos
        value:
            key: $KEYGAIA
            chain-id: GAIA
            rpc-addr: http://$IPGAIA:$PORTGAIA
            account-prefix: cosmos
            keyring-backend: test
            gas-adjustment: 1.3
            gas-prices: 0.01uatom
            debug: false
            timeout: 10s
            output-format: json
            sign-mode: sync
            strategy:
            type: native
            version: ics20-1
            order: UNORDERED
    stride:
        type: cosmos
        value:
            key: $KEYSTRIDE
            chain-id: STRIDE-TESTNET-2
            rpc-addr: http://$IPSTRIDE:$PORTSTRIDE
            account-prefix: stride
            keyring-backend: test
            gas-adjustment: 1.3
            gas-prices: 0.01ustrd
            debug: false
            timeout: 20s
            output-format: json
            sign-mode: sync
            strategy:
            type: native
            version: ics20-1
            order: UNORDERED
paths:
    gaia-stride:
        src:
            chain-id: STRIDE-TESTNET-2
            client-id: 07-tendermint-0
            connection-id: connection-0
            port-id: icacontroller-GAIA.DELEGATION,transfer,icacontriiler-GAIA.FEE,icacontroller-GAIA.WITHDRAWAL,icacontroller-GAIA.REDEMPTION
  
        dst:
            chain-id: GAIA
            client-id: 07-tendermint-0
            connection-id: connection-0
            port-id: icahost,icacontroller-GAIA.DELEGATION,transfer,icacontriiler-GAIA.FEE,icacontroller-GAIA.WITHDRAWAL,icacontroller-GAIA.REDEMPTION
        src-channel-filter:
            rule: allowlist
            channel-list:
                - channel-0 
                - channel-1 
                - channel-2 
                - channel-3 
                - channel-4 
        dst-channel-filter:
            rule: allowlist
            channel-list:
                - channel-0 
                - channel-1 
                - channel-2 
                - channel-3 
                - channel-4
EOF
```

## Import  keys for the relayer to use when signing and relaying transactions
   ```
     rly keys restore stride $KEYSTRIDE "mnemonic words here"
     rly keys restore gaia $KEYGAIA "mnemonic words here"
   ```
## Create go-v2 relayer service file
 (copy and paste into the terminal with one command)
```
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=Relayer_v2
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start gaia-stride --log-format logfmt --processor events
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```

## Start service
```
     sudo systemctl daemon-reload
     sudo systemctl enable rlyd
     sudo systemctl restart rlyd && journalctl -u rlyd -f
```