# private ethereum network

1. edit files/genesis.json, modify nonce and config.chainId
2. use terraform to deploy infrastructure 
    * install terraform
    * run terraform:
        ```
        terraform init
        terraform plan
        terraform apply -var 'keyname=YOUR AWS KEYNAME'
        ```
3. bootnode
    * login to bootnode
        ```bash
        ssh admin@$(terraform output bootnode_public_ip)
        ```
    * generate key
        ```bash
        bootnode -genkey boot.key
        ```
    * run bootnode
        ```bash
        echo "BOOTNODE_ADDRESS=enode://$(bootnode -nodekey boot.key -writeaddress)@$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4):30301"
        bootnode -nodekey boot.key
        ```
4. geth miners
    * login to each miner, the IPs are: ``terraform output node_public_ip``
    * initialize using genesis.json
        ```bash
        geth init /tmp/genesis.json
        ```
    * create ethereum account to store mining profits
        ```bash
        geth account new
        ```
    * run miner
        ```bash
        geth -networkid $(jq .config.chainId < /tmp/genesis.json) \
             -maxpeers 128 \
             -bootnodes $BOOTNODE_ADDRESS \
             -mine -minerthreads=1 \
             -etherbase=0x$(jq -r .address < ~/.ethereum/keystore/UTC*) \
             -rpc \
             -nat extip:$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
        ```
5. local miner
    * initialize using genesis.json
        ```bash
        geth init files/genesis.json
        ```
    * create ethereum account to store mining profits
        ```bash
        geth account new
        ```
    * run miner
        ```bash
        geth -networkid $(jq .config.chainId < files/genesis.json) \
             -bootnodes $BOOTNODE_ADDRESS \
             -mine -minerthreads=1 \
             -etherbase=0x$(jq -r .address < ~/.ethereum/keystore/UTC*) \
             -rpc
        ```
    * attach to console: ``geth attach``
    * install and run mist: https://github.com/ethereum/mist

