# Gravity DEX

## Table of Contents

* [Basic Node Operation](Basic-Node-Operation)
  * [Resources](#Resources)
  * [Hardware Requirements](#Hardware-Requirements)
  * [Install Go](#Install-Go)
  * [Install gaiad from source](#Install-gaiad-from-source)
  * [Setup](#Setup)
* [Query Information](#Query-Information)
  * [Pool](#Pool)
  * [Pools](#Pools)
  * [Batch](#Batch)
  * [Deposit](#Deposit)
  * [Deposits](#Deposits)
  * [Withdraw](#Withdraw)
  * [Withdraws](#Withdraws)
  * [Swap](#Swap)
  * [Swaps](#Swaps)
  * [Error Codes](#Error-codes)
  * [Find blocks through endblock_events](#Find-blocks-through-endblock_events)
  * [REST API](#REST-API)
* [Transaction Information](#Transaction-Information)
  * [MsgCreatePool](#MsgCreatePool)
  * [MsgDepositWithinBatch](#MsgDepositWithinBatch)
  * [MsgWithdrawWithinBatch](#MsgWithdrawWithinBatch)
  * [MsgSwapWithinBatch](#MsgSwapWithinBatch)
  * [Error Codes](#Error-codes)
  * [Broadcast tx through REST API](#Broadcast-tx-through-REST-API)
* [Calculations](Calculations)
  * [Pool Price](#Pool-Price)
  * [Expected Swap Price](#Expected-Swap-Price)
  * [Expected Swap Amount](#Expected-Swap-Amount)
* [Liquidity Params](Liquidity-Params)
  * [Current parameter value for the mainnet](#Current-parameter-value-for-themainnet)
  * [MinInitDepositAmount](#MinInitDepositAmount)
  * [InitPoolCoinMintAmount](#InitPoolCoinMintAmount)
  * [MaxReserveCoinAmount](#MaxReserveCoinAmount)
  * [PoolCreationFee](#PoolCreationFee)
  * [SwapFeeRate](#SwapFeeRate)
  * [WithdrawFeeRate](#WithdrawFeeRate)
  * [MaxOrderAmountRatio](#MaxOrderAmountRatio)
  * [UnitBatchSize](#UnitBatchSize)
  * [CircuitBreakerEnabled](#CircuitBreakerEnabled)
* [References](References)
  * [Sample Project](#Sample-Project)


## Basic Node Operation

### Resources

- [Official Repository](https://github.com/cosmos/gaia)
- [The latest release version of gaia](https://github.com/cosmos/gaia/releases) (currently [v5.0.5](https://github.com/cosmos/gaia/releases/tag/v5.0.5))
- [Genesis file](https://github.com/cosmos/mainnet/raw/master/genesis.cosmoshub-4.json.gz) (genesis.cosmoshub-4.json)
- [Official documenation: Join the Cosmos Hub Mainnet](https://hub.cosmos.network/main/gaia-tutorials/join-mainnet.html)
- [Gaia Basic tutorials](https://hub.cosmos.network/main/resources/service-providers.html)
- [Cosmos Hub Mainnet ggithub repository](https://github.com/cosmos/mainnet)
- [Discord channel](https://discord.gg/vcExX9T)
- [Peers and seed nodes](https://hackmd.io/@KFEZk8oMTz6vBlwADz0M4A/BkKEUOsZu#)
- [Performance Testing for Liquidity module(Gravity DEX)](https://github.com/tendermint/liquidity/blob/develop/doc/Performance%20Testing%20for%20Liquidity%20Module.pdf)

### Hardware Requirements

Minimum requirements

- 2 Core CPU
- 8 GB RAM
- SSD
- Example: AWS m5.Large

Recommended spec

- 4 Core CPU
- 16 GB RAM
- Provisioned IOPS SSD

### Install Go

Minimum requirements

- 2 Core CPU
- 8 GB RAM
- SSD
- Example: AWS m5.Large

Recommended spec

- 4 Core CPU
- 16 GB RAM
- Provisioned IOPS SSD

Go 1.16+ or higher is required for the Cosmos SDK.

For example, to install Go on Ubuntu 18.04.2 LTS:

```bash
# First remove any existing old Go installation
sudo rm -rf /usr/local/go

# Install the latest version of Go using this helpful script 
curl https://raw.githubusercontent.com/canha/golang-tools-install-script/master/goinstall.sh | bash

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile

# Verify that Go is installed
# Should return go version go1.16.4 linux/amd64
go version
```

### Install `gaiad` from source 

```bash
# Clone mainnet version of the project
git clone -b v5.0.5 https://github.com/cosmos/gaia.git
cd gaia
make install

# Verify the osmosisd version
gaiad version
-> v5.0.5
```

### Set Up the Chain

1. Initialize the chain with the config files. 

    ```bash
    # (Recommend) Save chain id in gaiad config
    gaiad config chain-id cosmoshub-4

    # Initialize chain
    # Replace <moniker> for the public alias of your node 
    # (you can also edit moniker later)
    # Example: gaiad init testnode --chain-id cosmoshub-4
    gaiad init <moniker> --chain-id=cosmoshub-4

    # This will create a new $HOME/.gaiad folder in your HOME directory.
    ```

2. Fetch the mainnet's [genesis](https://github.com/cosmos/mainnet/blob/master/genesis.cosmoshub-4.json.gz) file

    ```bash
    wget https://github.com/cosmos/mainnet/raw/master/genesis.cosmoshub-4.json.gz
    gzip -d genesis.cosmoshub-4.json.gz
    mv genesis.cosmoshub-4.json $HOME/.gaia/config/genesis.json
    ```
    
### Basic Configuration Setup

1. Manually add the following peers list in your `$HOME/.gaia/config/config.toml` file. 

    ```bash
    # Comma separated list of seed nodes to connect to
    seeds = "45f1fe6187a92cdabc7e52e6f65f743ad72c1ba1@157.230.116.241:26656,bf8328b66dceb4987e5cd94430af66045e59899f@public-seed.cosmos.vitwit.com:26656,cfd785a4224c7940e9a10f6c1ab24c343e923bec@164.68.107.188:26656,d72b3011ed46d783e369fdf8ae2055b99a1e5074@173.249.50.25:26656,ba3bacc714817218562f743178228f23678b2873@public-seed-node.cosmoshub.certus.one:26656,3c7cad4154967a294b3ba1cc752e40e8779640ad@84.201.128.115:26656,366ac852255c3ac8de17e11ae9ec814b8c68bddb@51.15.94.196:26656"

    # Comma separated list of nodes to keep persistent connections to
    persistent_peers = "45f1fe6187a92cdabc7e52e6f65f743ad72c1ba1@157.230.116.241:26656,bf8328b66dceb4987e5cd94430af66045e59899f@public-seed.cosmos.vitwit.com:26656,cfd785a4224c7940e9a10f6c1ab24c343e923bec@164.68.107.188:26656,d72b3011ed46d783e369fdf8ae2055b99a1e5074@173.249.50.25:26656,ba3bacc714817218562f743178228f23678b2873@public-seed-node.cosmoshub.certus.one:26656,3c7cad4154967a294b3ba1cc752e40e8779640ad@84.201.128.115:26656,366ac852255c3ac8de17e11ae9ec814b8c68bddb@51.15.94.196:26656"

    # The ibc and swap state are very large, causing the node itself 
    # to stop on excessive queries. Therefore, we customize the rpc configuration 
    # as above.

    # How long to wait for a tx to be committed during /broadcast_tx_commit.
    # WARNING: Using a value larger than 10s will result in increasing the
    # global HTTP write timeout, which applies to all connections and endpoints.
    # See https://github.com/tendermint/tendermint/issues/3435
    timeout_broadcast_tx_commit = "600s"
    ```

2. Another change to make in the `$HOME/.gaia/config/config.toml` file is to increase the timeout for a transaction to be committed and maximum size of request body and header. Since IBC and swap state transactions are very large, your node might stop from time to time when your node receives too many requests. Configure these values to increase the timeout: 

    ```bash
    # Maximum size of request body, in bytes
    max_body_bytes = 10000000

    # Maximum size of request header, in bytes
    max_header_bytes = 10485760
    ```

    Note: If your node is unable to connect to any of the seeds listed here, you can find seeds and peers in the [stargate peers and seed nodes](https://hackmd.io/@KFEZk8oMTz6vBlwADz0M4A/BkKEUOsZu#) document that is maintained by community members and in the [Atlas node explorer](https://atlas.cosmos.network/nodes), a list that is automatically generated by crawling the network. You can also join the [Discord channel](https://discord.gg/vcExX9T) to reach out to Cosmos community members to get help.

3. Run these commands to configure the seed nodes:

    ```bash
    sed -i '' 's#"tcp://127.0.0.1:26657"#"tcp://0.0.0.0:26657"#g' $HOME/.gaia/config/config.toml 
    sed -i '' 's/enable = false/enable = true/g' $HOME/.gaia/config/config.toml 
    sed -i '' 's/swagger = false/swagger = true/g' $HOME/.gaia/config/config.toml 
    ```

### State sync

State sync allows a new node to join a network by fetching a snapshot of the application state at a recent height instead of fetching and replaying all historical blocks. This state sync can reduce the time needed to sync with the network from days to minutes. See the [Cosmos SDK State Sync Guide](https://blog.cosmos.network/cosmos-sdk-state-sync-guide-99e4cf43be2f) to know more about Cosmos SDK state sync.

**Option 1. Set up using state synced data dump**

- The data that is state sync based on A height is about 1 gigabyte, so it can be set lighter and faster, but there is no index for past blocks.
- Download from one of these download links:
    - [cosmoshub-4_state_synced_7271293.tar](https://ipfs.io/ipfs/QmZqGwAQh9QHurbg3YvJJSkQd7T6tg35cFVD1FRF4bAm8J?filename=cosmoshub-4_state_synced_7271293.tar) (IPFS)
    - [cosmoshub-4_state_synced_7271293.tar](https://drive.google.com/file/d/1ku0HZ8xXrG4UDPe0ywdG4NQHve1iZ4tP/view?usp=sharing) [](https://ipfs.io/ipfs/QmZqGwAQh9QHurbg3YvJJSkQd7T6tg35cFVD1FRF4bAm8J?filename=cosmoshub-4_state_synced_7271293.tar)(Google Drive)
- Unzip

    `tar xvzf cosmoshub-4_state_synced_7271293.tar`

- Rename the gaia data files

    `mv gaia_synced ~/.gaia`

- Start node and sync

    `gaiad start`

**Option 2. State sync by using snapshot supply nodes**

1. Snapshot supply nodes (2 node setting, Block must be at the latest height)

    Manually add the following setting in the `$HOME/.gaiad/config/app.toml` file.

    ```bash
    [state-sync]

    # snapshot-interval specifies the block interval at which local state sync snapshots are
    # taken (0 to disable). Must be a multiple of pruning-keep-every.
    snapshot-interval = 100

    # snapshot-keep-recent specifies the number of recent snapshots to keep and serve (0 to keep all).
    snapshot-keep-recent = 2
    ```

2. Node to receive snapshot (1 node that needs to sync)

    Manually add the following setting in the `$HOME/.gaiad/config/config.toml` file.

    Be sure to set the peers list as well.

    `#"snapshot-interval = 100, snapshot-keep-recent = 2" → lastblock - 100 = trust_height`

    ```bash
    curl -s http://<Public-RPC-NODE>/block?height=7270300 |   jq -r '.result.block.header.height + "\n" + .result.block_id.hash'
    ```

    ```bash
    [statesync]
    # State sync rapidly bootstraps a new node by discovering, fetching, and restoring a state machine
    # snapshot from peers instead of fetching and replaying historical blocks. Requires some peers in
    # the network to take and serve state machine snapshots. State sync is not attempted if the node
    # has any local state (LastBlockHeight > 0). The node will have a truncated block history,
    # starting from the height of the snapshot.
    enable = true

    # RPC servers (comma-separated) for light client verification of the synced state machine and
    # retrieval of state data for node bootstrapping. Also needs a trusted height and corresponding
    # header hash obtained from a trusted source, and a period during which validators can be trusted.
    #
    # For Cosmos SDK-based chains, trust_period should usually be about 2/3 of the unbonding time (~2
    # weeks) during which they can be financially punished (slashed) for misbehavior.
    rpc_servers = "http://<Public-RPC-NODE1>,http://<Public-RPC-NODE2>"
    trust_height = 7270300
    trust_hash = "7115EDFC7ADD3A71F6685584DF9669600E8B8D0C8514A0089C66956F355FA027"
    trust_period = "336h"

    # Time to spend discovering snapshots before initiating a restore.
    discovery_time = "15s"

    # Temporary directory for state sync snapshot chunks, defaults to the OS tempdir (typically /tmp).
    # Will create a new, randomly named directory within, and remove it when done.
    temp_dir = "<snapshot save dir>"
    ```

    - `gaiad start`

        If your log looks like this, it's going well.

        ```bash
        7:11PM INF Fetching snapshot chunk chunk=7 format=1 height=7271000 module=statesync total=33
        7:11PM INF Applied snapshot chunk to ABCI app chunk=3 format=1 height=7271000 module=statesync total=33

        ...

        7:15PM INF executed block height=7271024 module=state num_invalid_txs=0 num_valid_txs=0
        7:15PM INF commit synced commit=436F6D6D697449447B5B313130203439203133392031353420313336203235342032303620313439203134203230352031323420313033203134322031392039392032303020313536203130372032392031343520313137203232362036342033372031333220343320323433203834203131392032313320323331203133355D3A3645463237307D
        7:15PM INF committed state app_hash=6E318B9A88FECE950ECD7C678E1363C89C6B1D9175E24025842BF35477D5E787 height=7271024 module=state num_txs=0
        7:15PM INF indexed block height=7271024 module=txindex
        7:15PM INF minted coins from module account amount=4453682uatom from=mint module=x/bank
        7:15PM INF executed block height=7271025 module=state num_invalid_txs=0 num_valid_txs=0
        7:15PM INF commit synced commit=436F6D6D697449447B5B3233312039372032333720313738203230392031333020323435203431203234332039203139362032343620313036203231392031393920393120343220313732203234372031313120313632203238203738203631203232392033312036322035352032333620363420323237203233335D3A3645463237317D
        7:15PM INF committed state app_hash=E761EDB2D182F529F309C4F66ADBC75B2AACF76FA21C4E3DE51F3E37EC40E3E9 height=7271025 module=state num_txs=0
        7:15PM INF indexed block height=7271025 module=txindex
        ```

    - When the sync is complete, set it up as follows:

        ```bash
        [statesync]
        # State sync rapidly bootstraps a new node by discovering, fetching, and restoring a state machine
        # snapshot from peers instead of fetching and replaying historical blocks. Requires some peers in
        # the network to take and serve state machine snapshots. State sync is not attempted if the node
        # has any local state (LastBlockHeight > 0). The node will have a truncated block history,
        # starting from the height of the snapshot.
        enable = false
        ```

Query, tx are available when synchronization is up to the latest height.

Details are available in the [Join the Cosmos Hub Mainnet](https://hub.cosmos.network/main/gaia-tutorials/join-mainnet.html) blog post.

## Query Information

### Pool

> Query details of a liquidity pool

Example `pool` query command using `pool-id` argument:

```bash
gaiad query liquidity pool 1
```

Result:

```yaml
pool:
    id: "1"
    pool_coin_denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
    reserve_account_address: cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we
    reserve_coin_denoms:
    - uatom
    - uusd
    type_id: 1
```

Example `pool` query command using `--pool-coin-denom` flag:

```bash
gaiad query liquidity pool --pool-coin-denom=pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
```

Result:

```yaml
pool:
    id: "1"
    pool_coin_denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
    reserve_account_address: cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we
    reserve_coin_denoms:
    - uatom
    - uusd
    type_id: 1
```

Example `pool` query command using `--reserve-acc` flag:

```bash
gaiad query liquidity pool --reserve-acc=cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we
```

Result:

```yaml
pool:
    id: "1"
    pool_coin_denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
    reserve_account_address: cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we
    reserve_coin_denoms:
    - uatom
    - uusd
    type_id: 1

```

> Query reserve coins of the pool:

```bash
gaiad query bank balances cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we
```

Result:

```yaml
balances:
- amount: "999003494"
    denom: uatom
- amount: "50050075000"
    denom: uusd
pagination:
    next_key: null
    total: "0"
```

> Query total supply of the pool coin:

```bash
gaiad query bank total --denom=pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
```

Result:

```yaml
amount: "1000000"
denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
```

### Pools

> Query for all liquidity pools

Example `pools` query command:

```bash
gaiad query liquidity pools
```

Result:

```yaml
pagination:
    next_key: null
    total: "2"
pools:
- id: "1"
    pool_coin_denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
    reserve_account_address: cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we
    reserve_coin_denoms:
    - uatom
    - uusd
    type_id: 1
- id: "2"
    pool_coin_denom: poolA4648A10F8D43B8EE4D915A35CB292618215D9F60CE3E2E29216489CF1FAE049
    reserve_account_address: cosmos153jg5y8c6sacaexezk34ev5jvxpptk0kscrx0x
    reserve_coin_denoms:
    - stake
    - uusd
    type_id: 1
```

### Batch

> Query details of a liquidity pool batch

Example `batch` query command:

```bash
gaiad query liquidity batch 1
```

Result:

```yaml
batch:
    begin_height: "563"
    deposit_msg_index: "2"
    executed: false
    index: "3"
    pool_id: "1"
    swap_msg_index: "2"
    withdraw_msg_index: "2"

```

### Deposit

> Query for the deposit message on the batch of the liquidity pool

Example `deposit` query command:

```bash
gaiad query liquidity deposit 1 1
```

Result:

```yaml
deposit:
    executed: true
    msg:
    deposit_coins:
    - amount: "1000000000"
        denom: uatom
    - amount: "50000000000"
        denom: uusd
    depositor_address: cosmos1le0a8y0ha99txx0ngsh0qhyyx7cwnjmmju52ed
    pool_id: "1"
    msg_height: "35"
    msg_index: "2"
    succeeded: true
    to_be_deleted: true
```

### Deposits

> Query for all deposit messages on the batch of the liquidity pool

Example `deposits` query command:

```bash
gaiad query liquidity deposits 1
```

Result:

```yaml
deposits:
- executed: true
    msg:
    deposit_coins:
    - amount: "100000000"
        denom: uatom
    - amount: "5000000000"
        denom: uusd
    depositor_address: cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3
    pool_id: "1"
    msg_height: "458"
    msg_index: "1"
    succeeded: true
    to_be_deleted: true
pagination:
    next_key: null
    total: "1"
```

### Withdraw

> Query for the withdraw message on the batch of the liquidity pool

Example `withdraw` query command:

```bash
gaiad query liquidity withdraws 1 2
```

Result:

```yaml
pagination:
    next_key: null
    total: "1"
withdraws:
- executed: true
    msg:
    pool_coin:
        amount: "10000"
        denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
    pool_id: "1"
    withdrawer_address: cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3
    msg_height: "562"
    msg_index: "1"
    succeeded: true
    to_be_deleted: true
```

### Withdraws

> Query for all withdraw messages on the batch of the liquidity pool

Example `withdraws` query command:

```bash
gaiad query liquidity withdraws 1
```

Result:

```yaml
pagination:
    next_key: null
    total: "1"
withdraws:
- executed: true
    msg:
    pool_coin:
        amount: "10000"
        denom: pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295
    pool_id: "1"
    withdrawer_address: cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3
    msg_height: "562"
    msg_index: "1"
    succeeded: true
    to_be_deleted: true
```

### Swap

> Query for the swap message on the batch of the liquidity pool

Example `swap` query command:

```bash
gaiad query liquidity swaps 1 2
```

Result:

```json
pagination:
    next_key: null
    total: "1"
swaps:
- exchanged_offer_coin:
    amount: "50000000"
    denom: uusd
    executed: true
    msg:
    demand_coin_denom: uatom
    offer_coin:
        amount: "50000000"
        denom: uusd
    offer_coin_fee:
        amount: "75000"
        denom: uusd
    order_price: "0.019000000000000000"
    pool_id: "1"
    swap_requester_address: cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3
    swap_type_id: 1
    msg_height: "178"
    msg_index: "1"
    order_expiry_height: "178"
    remaining_offer_coin:
    amount: "0"
    denom: uusd
    reserved_offer_coin_fee:
    amount: "0"
    denom: uusd
    succeeded: true
    to_be_deleted: true
```

### Swaps

> Query for all swap messages on the batch of the liquidity pool

Example `swaps` query command:

```bash
gaiad query liquidity swaps 1
```

Result:

```yaml
pagination:
    next_key: null
    total: "1"
swaps:
- exchanged_offer_coin:
    amount: "50000000"
    denom: uusd
    executed: true
    msg:
    demand_coin_denom: uatom
    offer_coin:
        amount: "50000000"
        denom: uusd
    offer_coin_fee:
        amount: "75000"
        denom: uusd
    order_price: "0.019000000000000000"
    pool_id: "1"
    swap_requester_address: cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3
    swap_type_id: 1
    msg_height: "178"
    msg_index: "1"
    order_expiry_height: "178"
    remaining_offer_coin:
    amount: "0"
    denom: uusd
    reserved_offer_coin_fee:
    amount: "0"
    denom: uusd
    succeeded: true
    to_be_deleted: true
```

### Error Codes

For error codes with the description, see [errors.go](https://github.com/tendermint/liquidity/blob/develop/x/liquidity/types/errors.go).

### Find blocks through endblock_events

- `curl "78.46.243.107:26657/block_search?query=\"swap_transacted.success='success'\""`
- http://78.46.243.107:26657/block_search?query=%22swap_transacted.success=%27success%27%20AND%20swap_transacted.pool_id=%271%27%22
- http://78.46.243.107:26657/block_search?query=%22swap_transacted.success=%27success%27%20AND%20swap_transacted.pool_id=%271%27%22
- http://78.46.243.107:26657/block_results?height=7342649

### REST API

A node exposes the REST server default port of 1317. Configure the port in [api] section of the app.toml file located in your $HOME/.liquidityd/config/ directory. When swagger param is set to true, you can open up your browser and check out the Swagger documentation in http://localhost:1317/swagger-liquidity/. You can also reference the public api documentation in [this link](https://app.swaggerhub.com/apis-docs/bharvest/cosmos-sdk_liquidity_module_rest_and_g_rpc_gateway_docs/).

REST API Specs (swagger)

- [Cosmos SDK](https://v1.cosmos.network/rpc/v0.42.6)
- [Gravity DEX](https://v1.cosmos.network/rpc/gravity-dex)
- [IBC](https://v1.cosmos.network/rpc/ibc)

## Transaction Information

With the exception of creating the liquidity pool, all commands are implemented to execute on the batch.

### MsgCreatePool

> Create liquidity pool

Example `create-pool` tx command:

```bash
gaiad tx liquidity create-pool 1 1000000000uatom,50000000000uusd --from user1 --keyring-backend test --chain-id testing -y
```

JSON Structure:

```json
{
    "body": {
    "messages": [
        {
        "@type": "/tendermint.liquidity.MsgCreatePool",
        "pool_creator_address": "cosmos1s6cjfm4djg95jkzsfe490yfc9k6wazx6culyft",
        "pool_type_id": 1,
        "deposit_coins": [
            {
            "denom": "uatom",
            "amount": "1000000000"
            },
            {
            "denom": "uusd",
            "amount": "50000000000"
            }
        ]
        }
    ],
    "memo": "",
    "timeout_height": "0",
    "extension_options": [],
    "non_critical_extension_options": []
    },
    "auth_info": {
    "signer_infos": [],
    "fee": {
        "amount": [],
        "gas_limit": "200000",
        "payer": "",
        "granter": ""
    }
    },
    "signatures": []
}

```

Result:

```json
{
    "height": "5",
    "txhash": "C326C06CFB50589F72CBACD6F0028EE00B94F259C869D55653CEE11208531496",
    "codespace": "",
    "code": 0,
    "data": "0A0D0A0B6372656174655F706F6F6C",
    "raw_log": "...",
    "logs": [
    {
        "msg_index": 0,
        "log": "",
        "events": [
        {
            "type": "create_pool",
            "attributes": [
            {
                "key": "pool_id",
                "value": "1"
            },
            {
                "key": "pool_type_id",
                "value": "1"
            },
            {
                "key": "pool_name",
                "value": "uatom/uusd/1"
            },
            {
                "key": "reserve_account",
                "value": "cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we"
            },
            {
                "key": "deposit_coins",
                "value": "1000000000uatom,50000000000uusd"
            },
            {
                "key": "pool_coin_denom",
                "value": "pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295"
            }
            ]
        },
        {
            "type": "message",
            "attributes": [
            {
                "key": "action",
                "value": "create_pool"
            },
            {
                "key": "sender",
                "value": "cosmos1s6cjfm4djg95jkzsfe490yfc9k6wazx6culyft"
            },
            {
                "key": "sender",
                "value": "cosmos1tx68a8k9yz54z06qfve9l2zxvgsz4ka3hr8962"
            },
            {
                "key": "sender",
                "value": "cosmos1s6cjfm4djg95jkzsfe490yfc9k6wazx6culyft"
            },
            {
                "key": "module",
                "value": "liquidity"
            }
            ]
        },
        {
            "type": "transfer",
            "attributes": [
            {
                "key": "recipient",
                "value": "cosmos1jmhkafh94jpgakr735r70t32sxq9wzkayzs9we"
            },
            {
                "key": "amount",
                "value": "1000000000uatom,50000000000uusd"
            },
            {
                "key": "recipient",
                "value": "cosmos1s6cjfm4djg95jkzsfe490yfc9k6wazx6culyft"
            },
            {
                "key": "amount",
                "value": "1000000pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295"
            },
            {
                "key": "recipient",
                "value": "cosmos1jv65s3grqf6v6jl3dp4t6c9t9rk99cd88lyufl"
            },
            {
                "key": "sender",
                "value": "cosmos1s6cjfm4djg95jkzsfe490yfc9k6wazx6culyft"
            },
            {
                "key": "amount",
                "value": "100000000stake"
            }
            ]
        }
        ]
    }
    ],
    "info": "",
    "gas_wanted": "200000",
    "gas_used": "163716",
    "tx": null,
    "timestamp": ""
}
```

### MsgDepositWithinBatch

> Deposit to the liquidity pool batch

Example `deposit` tx command:

```bash
gaiad tx liquidity deposit 1 100000000uatom,5000000000uusd --from validator --keyring-backend test --chain-id testing -y
```

JSON Structure:

```json
{
    "body": {
    "messages": [
        {
        "@type": "/tendermint.liquidity.MsgDepositWithinBatch",
        "depositor_address": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3",
        "pool_id": "1",
        "deposit_coins": [
            {
            "denom": "uatom",
            "amount": "100000000"
            },
            {
            "denom": "uusd",
            "amount": "5000000000"
            }
        ]
        }
    ],
    "memo": "",
    "timeout_height": "0",
    "extension_options": [],
    "non_critical_extension_options": []
    },
    "auth_info": {
    "signer_infos": [],
    "fee": {
        "amount": [],
        "gas_limit": "200000",
        "payer": "",
        "granter": ""
    }
    },
    "signatures": []
}
```

Result:

```json
{
    "height": "458",
    "txhash": "8D8FA31125AB2A984D28F362ADC05946208C0E7927B13F984D9AD6E8E5327782",
    "codespace": "",
    "code": 0,
    "data": "0A160A146465706F7369745F77697468696E5F6261746368",
    "raw_log": "...",
    "logs": [
    {
        "msg_index": 0,
        "log": "",
        "events": [
        {
            "type": "deposit_within_batch",
            "attributes": [
            {
                "key": "pool_id",
                "value": "1"
            },
            {
                "key": "batch_index",
                "value": "1"
            },
            {
                "key": "msg_index",
                "value": "1"
            },
            {
                "key": "deposit_coins",
                "value": "100000000uatom,5000000000uusd"
            }
            ]
        },
        {
            "type": "message",
            "attributes": [
            {
                "key": "action",
                "value": "deposit_within_batch"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "module",
                "value": "liquidity"
            }
            ]
        },
        {
            "type": "transfer",
            "attributes": [
            {
                "key": "recipient",
                "value": "cosmos1tx68a8k9yz54z06qfve9l2zxvgsz4ka3hr8962"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "amount",
                "value": "100000000uatom,5000000000uusd"
            }
            ]
        }
        ]
    }
    ],
    "info": "",
    "gas_wanted": "200000",
    "gas_used": "79385",
    "tx": null,
    "timestamp": ""
}
```

### MsgWithdrawWithinBatch

> Withdraw pool coin from the liquidity pool

Example `withdraw` tx command:

```bash
gaiad tx liquidity withdraw 1 10000pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295 --from validator --chain-id testing --keyring-backend test -y
```

JSON Structure:

```json
{
    "body": {
    "messages": [
        {
        "@type": "/tendermint.liquidity.MsgWithdrawWithinBatch",
        "withdrawer_address": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3",
        "pool_id": "1",
        "pool_coin": {
            "denom": "pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295",
            "amount": "10000"
        }
        }
    ],
    "memo": "",
    "timeout_height": "0",
    "extension_options": [],
    "non_critical_extension_options": []
    },
    "auth_info": {
    "signer_infos": [],
    "fee": {
        "amount": [],
        "gas_limit": "200000",
        "payer": "",
        "granter": ""
    }
    },
    "signatures": []
}

```

Result:

```json
{
    "height": "562",
    "txhash": "BE8827F69E8BC5909A0FFC713B6D267606A91A1CFA07552E69020638E9E1D563",
    "codespace": "",
    "code": 0,
    "data": "0A170A1577697468647261775F77697468696E5F6261746368",
    "raw_log": "...",
    "logs": [
    {
        "msg_index": 0,
        "log": "",
        "events": [
        {
            "type": "message",
            "attributes": [
            {
                "key": "action",
                "value": "withdraw_within_batch"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "module",
                "value": "liquidity"
            }
            ]
        },
        {
            "type": "transfer",
            "attributes": [
            {
                "key": "recipient",
                "value": "cosmos1tx68a8k9yz54z06qfve9l2zxvgsz4ka3hr8962"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "amount",
                "value": "10000pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295"
            }
            ]
        },
        {
            "type": "withdraw_within_batch",
            "attributes": [
            {
                "key": "pool_id",
                "value": "1"
            },
            {
                "key": "batch_index",
                "value": "2"
            },
            {
                "key": "msg_index",
                "value": "1"
            },
            {
                "key": "pool_coin_denom",
                "value": "pool96EF6EA6E5AC828ED87E8D07E7AE2A8180570ADD212117B2DA6F0B75D17A6295"
            },
            {
                "key": "pool_coin_amount",
                "value": "10000"
            }
            ]
        }
        ]
    }
    ],
    "info": "",
    "gas_wanted": "200000",
    "gas_used": "67701",
    "tx": null,
    "timestamp": ""
}
```

### MsgSwapWithinBatch

> Swap offer coin with demand coin from the liquidity pool with the given order price

Example `swap` tx command:

```bash
gaiad tx liquidity swap 1 1 50000000uusd uatom 0.019 0.003 --from validator --chain-id testing --keyring-backend test -y
```

JSON Structure:

```json
{
    "body": {
    "messages": [
        {
        "@type": "/tendermint.liquidity.MsgSwapWithinBatch",
        "swap_requester_address": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3",
        "pool_id": "1",
        "swap_type_id": 1,
        "offer_coin": {
            "denom": "uusd",
            "amount": "50000000"
        },
        "demand_coin_denom": "uatom",
        "offer_coin_fee": {
            "denom": "uusd",
            "amount": "75000"
        },
        "order_price": "0.019000000000000000"
        }
    ],
    "memo": "",
    "timeout_height": "0",
    "extension_options": [],
    "non_critical_extension_options": []
    },
    "auth_info": {
    "signer_infos": [],
    "fee": {
        "amount": [],
        "gas_limit": "200000",
        "payer": "",
        "granter": ""
    }
    },
    "signatures": []
}
```

Result:

```json
{
    "height": "178",
    "txhash": "AA9A3A50D9AC639730F61824AA2BD3BA9EBCCEA7E52147353C0E680041F21243",
    "codespace": "",
    "code": 0,
    "data": "0A130A11737761705F77697468696E5F6261746368",
    "raw_log": "...",
    "logs": [
    {
        "msg_index": 0,
        "log": "",
        "events": [
        {
            "type": "message",
            "attributes": [
            {
                "key": "action",
                "value": "swap_within_batch"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "module",
                "value": "liquidity"
            }
            ]
        },
        {
            "type": "swap_within_batch",
            "attributes": [
            {
                "key": "pool_id",
                "value": "1"
            },
            {
                "key": "batch_index",
                "value": "1"
            },
            {
                "key": "msg_index",
                "value": "1"
            },
            {
                "key": "swap_type_id",
                "value": "1"
            },
            {
                "key": "offer_coin_denom",
                "value": "uusd"
            },
            {
                "key": "offer_coin_amount",
                "value": "50000000"
            },
            {
                "key": "offer_coin_fee_amount",
                "value": "75000"
            },
            {
                "key": "demand_coin_denom",
                "value": "uatom"
            },
            {
                "key": "order_price",
                "value": "0.019000000000000000"
            }
            ]
        },
        {
            "type": "transfer",
            "attributes": [
            {
                "key": "recipient",
                "value": "cosmos1tx68a8k9yz54z06qfve9l2zxvgsz4ka3hr8962"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "amount",
                "value": "50000000uusd"
            },
            {
                "key": "recipient",
                "value": "cosmos1tx68a8k9yz54z06qfve9l2zxvgsz4ka3hr8962"
            },
            {
                "key": "sender",
                "value": "cosmos1h6ht09xx0ue0fqmezk7msgqcc9k20a5x5ynvc3"
            },
            {
                "key": "amount",
                "value": "75000uusd"
            }
            ]
        }
        ]
    }
    ],
    "info": "",
    "gas_wanted": "200000",
    "gas_used": "95327",
    "tx": null,
    "timestamp": ""
}
```

### Error codes

For error codes with the description, see [errors.go](https://github.com/tendermint/liquidity/blob/develop/x/liquidity/types/errors.go).

### Broadcast tx through REST API

The POST endpoints of the new gGPC-gateway REST are not available. The [Migrating to New REST Endpoints](https://docs.cosmos.network/master/migrations/rest.html#migrating-to-new-rest-endpoints) Cosmos SDK guide suggests to use Protobuf directly. You can use the command line interface or use the temporarily available REST API at `localhost:1317/cosmos/tx/v1beta1/txs`.

For example, to broadcast a transaction by using the [New gRPC-gateway REST Endpoint](https://github.com/cosmos/cosmos-sdk/blob/master/docs/migrations/rest.md#migrating-to-new-rest-endpoints):

```bash
curl --header "Content-Type: application/json" --request POST --data '{"tx_bytes":"CoMBCoABCh0vdGVuZGVybWludC5saXF1aWRpdHkuTXNnU3dhcBJfCi1jb3Ntb3MxN3dncHpyNGd2YzN1aHBmcnUyNmVhYTJsc203NzJlMnEydjBtZXgQAhgBIAEqDQoFc3Rha2USBDEwMDAyBGF0b206EzExNTAwMDAwMDAwMDAwMDAwMDASWApQCkYKHy9jb3Ntb3MuY3J5cHRvLnNlY3AyNTZrMS5QdWJLZXkSIwohAqzfoAEi0cFg0zqwBuGNvHml4XJNS3EQuVti8/yGH88NEgQKAgh/GAgSBBDAmgwaQGTRN67x2WYF/L5DsRD3ZY1Kt9cVpg3rW+YbXtihxcB6bJWhMxuFr0u9SnGkCuAgOuLH9YU8ROFUo1gGS1RpTz0=","mode":1}' localhost:1317/cosmos/tx/v1beta1/txs
```

## Calculations

### Pool Price

Each pair of tokens in a pool has a pool price defined by the reserves of that pair of tokens. 

![pool price gravity dex](https://user-images.githubusercontent.com/20435620/129909416-38f133e9-bdbe-4a0d-971e-9c73f88c7da2.png)

Example

Let's get pool 1 information

[http://116.202.170.226:1317/cosmos/liquidity/v1beta1/pools/1](http://116.202.170.226:1317/cosmos/liquidity/v1beta1/pools/1) 

```json
{
  "pool": {
    "id": "1",
    "type_id": 1,
    "reserve_coin_denoms": [
      "ibc/14F9BC3E44B8A9C1BE1FB08980FAB87034C9905EF17CF2F5008FC085218811CC",
      "uatom"
    ],
    "reserve_account_address": "cosmos1m7uyxn26sz6w4755k6rch4dc2fj6cmzajkszvn",
    "pool_coin_denom": "poolDFB8434D5A80B4EAFA94B6878BD5B85265AC6C5D37204AB899B1C3C52543DA7E"
  }
}
```

Use the reserve_account_address to get reserve coin amounts

[http://116.202.170.226:1317/cosmos/bank/v1beta1/balances/cosmos1m7uyxn26sz6w4755k6rch4dc2fj6cmzajkszvn](http://116.202.170.226:1317/cosmos/bank/v1beta1/balances/cosmos1m7uyxn26sz6w4755k6rch4dc2fj6cmzajkszvn) 

```json
{
  "balances": [
    {
      "denom": "ibc/14F9BC3E44B8A9C1BE1FB08980FAB87034C9905EF17CF2F5008FC085218811CC",
      "amount": "60727676783"
    },
    {
      "denom": "uatom",
      "amount": "9018280896"
    }
  ],
  "pagination": {
    "next_key": null,
    "total": "2"
  }
}
```

### Expected Swap Price

When users swap tokens in the pool, they want to know how much the expected swap price is. Here, the expected swap price is the price considering slippage. (swap fees are not considered)

![Expected Swap Price Gravity DEX](https://user-images.githubusercontent.com/20435620/129909554-f514ba87-4957-437a-9310-66437c757a3f.png)

### Expected Swap Amount

For practical purposes, users also would like to know what amount of tokens they will receive at a specific price. 

- If users want to swap X token to Y token, the swap price has to be higher than current pool price. The amount of X token they will provide to pool and the amount of Y token they will receive are below.

![Expected Swap Amount Gravity DEX 1](https://user-images.githubusercontent.com/20435620/129909665-09271783-2077-49e2-a369-8fe677b407b5.png)


- Reversely, if users want to swap Y token to X token, the swap price has to be lower than current pool price. The amount of Y token they will provide to pool and the amount of X token they will receive are below.

![Expected Swap Amount Gravity DEX 2](https://user-images.githubusercontent.com/20435620/129909752-fa24fae5-8ade-49fd-acff-988d33af500f.png)

## 5. Liquidity Params

Query liquidity parameters using cli

```bash
gaiad query liquidity params --output json
```

Query liquidity parameters using REST API

- {API-SERVER}/cosmos/liquidity/v1beta1/params
- ex) [http://116.202.170.226:1317/cosmos/liquidity/v1beta1/params](http://116.202.170.226:1317/cosmos/liquidity/v1beta1/params)

### Current parameter value for the mainnet

(cosmoshub-4, height 7,270,000) 

```json
{
  "params": {
    "pool_types": [
      {
        "id": 1,
        "name": "StandardLiquidityPool",
        "min_reserve_coin_num": 2,
        "max_reserve_coin_num": 2,
        "description": "Standard liquidity pool with pool price function X/Y, ESPM constraint, and two kinds of reserve coins"
      }
    ],
    "min_init_deposit_amount": "1000000",
    "init_pool_coin_mint_amount": "1000000",
    "max_reserve_coin_amount": "0",
    "pool_creation_fee": [
      {
        "denom": "uatom",
        "amount": "40000000"
      }
    ],
    "swap_fee_rate": "0.003000000000000000",
    "withdraw_fee_rate": "0.000000000000000000",
    "max_order_amount_ratio": "0.100000000000000000",
    "unit_batch_height": 1,
    "circuit_breaker_enabled": false
  }
}
```

### MinInitDepositAmount

Minimum number of coins to be deposited to the liquidity pool upon pool creation.

### InitPoolCoinMintAmount

Initial mint amount of pool coin on pool creation.

### MaxReserveCoinAmount

Limit the size of each liquidity pool. The deposit transaction fails if the total reserve coin amount after the deposit is larger than the reserve coin amount.

The default value of zero means no limit.

Note: Especially in the early phases of liquidity module adoption, set `MaxReserveCoinAmount` to a non-zero value to minimize risk on error or exploitation.

### PoolCreationFee

Fee paid for to create a LiquidityPool creation. This fee prevents spamming and is collected in in the community pool of the distribution module.

### SwapFeeRate

Swap fee rate for every executed swap. When a swap is requested, the swap fee is reserved:

- Half reserved as `OfferCoinFee`
- Half reserved as `ExchangedCoinFee`

The swap fee is collected when a batch is executed.

### WithdrawFeeRate

Reserve coin withdrawal with less proportion by `WithdrawFeeRate`. This fee prevents attack vectors from repeated deposit/withdraw transactions.

### MaxOrderAmountRatio

Maximum ratio of reserve coins that can be ordered at a swap order.

### UnitBatchHeight

The smallest unit batch size for every liquidity pool.

### CircuitBreakerEnabled

The intention of circuit breaker is to have a contingency plan for a running network which maintains network liveness. This parameter enables or disables all transaction message types in liquidity module.

## 6. Reference

### Sample Project

- https://github.com/allinbits/gravity-dex-stats

