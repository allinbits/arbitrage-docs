# Osmosis

## Table Of Contents

* [Basic Node Operation](Basic-Node-Operation)
  * [Resources](#Resources)
  * [Hardware Requirements](#Hardware-Requirements)
  * [Install Go](#Install-Go)
  * [Install osmosisd from source](#Install-osmosisd-from-source)
  * [Setup](#Setup)
  * [Basic Configuration](#Basic-Configuration)
  * [State Sync](#State-Sync)
  * [Start](#Start)
* [Calculations](Calculations)
  * [Pool Price](#Pool-Price)
  * [Expected Swap Price](#Expected-Swap-Price)
  * [Expected Swap Amount](#Expected-Swap-Amount)
* [Find Arbitrage Opportunity](Find-Arbitrage-Opportunity)
  * [Global Price](#Global-Price)
  * [Pool Price](#Pool-Price)
  * [Expected Swap Price](#Expected-Swap-Price)
* [Osmosis Params](Osmosis-Params)
  * [SwapFee](#SwapFee)
  * [ExitFee](#ExitFee)
  * [MinPoolAssets](#MinPoolAssets)
  * [MaxPoolAssets](#MaxPoolAssets)
  * [InitPoolSharesSupply](#InitPoolSharesSupply)
* [References](References)
  * [Command-line Interface](#Command-line-Interface)

## Basic Node Operation

### Resources

- [Official Repository](https://github.com/osmosis-labs/osmosis)
- [Genesis file](https://github.com/osmosis-labs/networks/blob/main/osmosis-1/genesis.json)
- [Setting Up a Genesis Osmosis Validator Guide](https://github.com/osmosis-labs/networks/blob/main/genesis-validators.md)
- [Peers](https://github.com/osmosis-labs/networks/blob/main/peers.md) (Mainnet)

### Hardware Requirements

At the time of writing this documentation, this is the recommended server specs by osmosis team. As the usage of the blockchain grows, the server requirements may increase as well. 

- 4 or more physical CPU cores
- At least 500GB of SSD disk storage
- At least 16GB of memory
- At least 100mbps network bandwidth

### Install Go

Osmosis is built using Go and requires Go version 1.15+. 

In this example, we will be installing Go on Ubuntu 18.04.2 LTS:

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

### Install `osmosisd` from source

```bash
# Clone the project
# At the time of writing this guide, mainnet uses 3.1.0 version
git clone -b v3.1.0 https://github.com/osmosis-labs/osmosis.git
cd osmosis
go mod tidy

# Build osmosis node software "osmosisd"
# This command will install the executable osmosis node daemon to your GOPATH
make install

# Verify the osmosisd version
osmosisd version --long
-> 
name: osmosis
server_name: osmosisd
version: 3.1.0
commit: 13916d1e10bca718b6ea7f4b84715710bc319e6d
build_tags: netgo,ledger
go: go version go1.16.3 linux/amd64
```

### Setup

1. Initialize the chain 

    ```bash
    # (Recommend) Save chain id in osmosisd config
    #
    # This command will save the chain id into your client.toml which is located
    # in your home directory (default: $HOME/.osmosisd/config/client.toml)
    osmosisd config chain-id osmosis-1

    # Initialize chain
    #
    # Replace <moniker> for the public alias of your node (you can edit moniker later)
    # Example: osmosisd init testnode --chain-id osmosis-1
    # This will create a new $HOME/.osmosisd folder in your HOME directory
    osmosisd init <moniker> --chain-id=osmosis-1
    ```

2. Fetch the mainnet's genesis file

    [https://github.com/osmosis-labs/networks/blob/main/osmosis-1/genesis.json](https://github.com/osmosis-labs/networks/blob/main/osmosis-1/genesis.json) 

    ```bash
    wget -O $HOME/.osmosisd/config/genesis.json https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json
    ```

### Basic Configuration

1. Manually add the below peers list in your `$HOME/.osmosisd/config/config.toml` file. You can find the peers list in [this link](https://github.com/osmosis-labs/networks/blob/main/peers.md). Since IBC and swap state transactions are very large, it is suggested to increase the timeout for a transaction to be committed and maximum size of request body and header. Without configuring those values, you may experience the node to stop from time to time when your node receives too many requests. 

    ```bash
    seeds = "8f67a2fcdd7ade970b1983bf1697111d35dfdd6f@52.79.199.137:26656,00c328a33578466c711874ec5ee7ada75951f99a@35.82.201.64:26656,cfb6f2d686014135d4a6034aa6645abd0020cac6@52.79.88.57:26656,8d9967d5f865c68f6fe2630c0f725b0363554e77@134.255.252.173:26656,785bc83577e3980545bac051de8f57a9fd82695f@194.233.164.146:26656,778fdedf6effe996f039f22901a3360bc838b52e@161.97.187.189:36657,64d36f3a186a113c02db0cf7c588c7c85d946b5b@209.97.132.170:26656,4d9ac3510d9f5cfc975a28eb2a7b8da866f7bc47@37.187.38.191:26656,2115945f074ddb038de5d835e287fa03e32f0628@95.217.43.85:26656,bf2c480eff178d2647ba1adfeee8ced568fe752c@91.65.128.44:26656,2f9c16151400d8516b0f58c030b3595be20b804c@37.120.245.167:26656,bada684070727cb3dda430bcc79b329e93399665@173.212.240.91:26656,83adaa38d1c15450056050fd4c9763fcc7e02e2c@ec2-44-234-84-104.us-west-2.compute.amazonaws.com:26656,23142ab5d94ad7fa3433a889dcd3c6bb6d5f247d@95.217.193.163:26656,f82d1a360dc92d4e74fdc2c8e32f4239e59aebdf@95.217.121.243:26656,e437756a853061cc6f1639c2ac997d9f7e84be67@144.76.183.180:26656,3fea02d121cb24503d5fbc53216a527257a9ab55@143.198.145.208:26656,7024d1ca024d5e33e7dc1dcb5ed08349768220b9@134.122.42.20:26656,d326ad6dffa7763853982f334022944259b4e7f4@143.110.212.33:26656,e7916387e05acd53d1b8c0f842c13def365c7bb6@176.9.64.212:26666,55eea69c21b46000c1594d8b4a448563b075d9e3@34.107.19.235:26656,9faf468b90a3b2b85ffd88645a15b3715f68bb0b@195.201.122.100:26656,ffc82412c0261a94df122b9cc0ce1de81da5246b@15.222.240.16:26656,5b90a530464885fd28c31f698c81694d0b4a1982@35.183.238.70:26656,7b6689cb18d625bbc069aa99d9d5521293db442c@51.158.97.192:26656,fda06dcebe2acd17857a6c9e9a7b365da3771ceb@52.206.252.176:26656,b4592eea84652e565b464cada03279dc4aac618c@177.66.42.189:26656"

    # The ibc and swap state are very large, causing the node itself 
    # to stop on excessive queries. Therefore, we customize the rpc configuration 
    # as above.

    # How long to wait for a tx to be committed during /broadcast_tx_commit.
    # WARNING: Using a value larger than 10s will result in increasing the
    # global HTTP write timeout, which applies to all connections and endpoints.
    # See https://github.com/tendermint/tendermint/issues/3435
    timeout_broadcast_tx_commit = "600s"

    # Maximum size of request body, in bytes
    max_body_bytes = 10000000

    # Maximum size of request header, in bytes
    max_header_bytes = 10485760
    ```

2. Replace necessary configuration 

    ```bash
    sed -i '' 's#"tcp://127.0.0.1:26657"#"tcp://0.0.0.0:26657"#g' $HOME/.osmosisd/config/config.toml 
    sed -i '' 's/enable = false/enable = true/g' $HOME/.osmosisd/config/config.toml 
    sed -i '' 's/swagger = false/swagger = true/g' $HOME/.osmosisd/config/config.toml 
    ```

### State Sync

State sync allows a new node to join a network by fetching a snapshot of the application state at a recent height instead of fetching and replaying all historical blocks. This can reduce the time needed to sync with the network from days to minutes. Reference [this article](https://blog.cosmos.network/cosmos-sdk-state-sync-guide-99e4cf43be2f) to know more about Cosmos SDK state sync.

**Option 1. Setup using state synced data dump**

- You can follow [this guide](https://osmosis.quicksync.io/) operated by ChainLayer

**Option 2. State sync via snapshot supply nodes**

1. snapshot supply nodes (2 node setting- Block must be at the latest height)

    Manually add the below setting in `$HOME/.osmosisd/config/app.toml` file.

    ```bash
    ###############################################################################
    ###                        State Sync Configuration                         ###
    ###############################################################################

    # State sync snapshots allow other nodes to rapidly join the network without replaying historical
    # blocks, instead downloading and applying a snapshot of the application state at a given height.
    [state-sync]

    # snapshot-interval specifies the block interval at which local state sync snapshots are
    # taken (0 to disable). Must be a multiple of pruning-keep-every.
    snapshot-interval = 100

    # snapshot-keep-recent specifies the number of recent snapshots to keep and serve (0 to keep all).
    snapshot-keep-recent = 2
    ```

2. Node to receive snapshot (1 node that needs to sync)

    Manually change the state sync values in your `$HOME/.osmosisd/config/app.toml` file.

    (Set the peers list as well)

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
    trust_hash = ""
    trust_period = "336h"

    # Time to spend discovering snapshots before initiating a restore.
    discovery_time = "15s"

    # Temporary directory for state sync snapshot chunks, defaults to the OS tempdir (typically /tmp).
    # Will create a new, randomly named directory within, and remove it when done.
    temp_dir = "<snapshot save dir>"
    ```

    - `osmosisd start`

        If you see log like this, it's going well.

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
### Start

Wait for at least 10 minutes to connect with peers. If you continue to see dialing peers in logs, then there may not be a single node that accepts connection. You need to find working peers by joining Osmosis [Discord](https://discord.com/invite/osmosis) and ask for it. 

Simple way to start

```bash
osmosisd start
```

Use `systemd service` to start 

```json
sudo tee /etc/systemd/system/osmosisd.service > /dev/null <<EOF
[Unit]
Description=Osmosis Daemon
After=network-online.target
Requires=osmosisd.service

[Service]
User=root
ExecStart=/root/goApps/bin/osmosisd start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target

# Enable osmosisd and start 
sudo systemctl daemon-reload
sudo systemctl enable osmosisd
sudo systemctl start osmosisd

# Monitor osmosisd service logs
journalctl -u osmosisd -f
```

## Calculations

### Pool Price

Each pair of tokens in a pool has a pool price defined by the reserves and weights of that pair of tokens. 

![Pool Price Osmosis](https://user-images.githubusercontent.com/20435620/129910631-7493a236-9df7-4984-8b2f-f41f4ea70c8c.png)

### Expected Swap Price

When users swap tokens in the pool, they want to know how much the expected swap price is. Here, the expected swap price is the price considering slippage. (swap fees are not considered)

![Expected Swap Price Osmosis](https://user-images.githubusercontent.com/20435620/129910553-80b8b1ac-bb62-43f7-8d35-bc9f5da0d862.png)

### Expected Swap Amount

Not available yet

## Find Arbitrage Opportunity

In this section, we're going to analyze ATOM / OSMO pool based on the calculation written above. 

Note that REST API endpoint used in this section doesn't guarantee availability and it may not be stable to use for production level. We recommend you to set up your own if you consider doing arbitrage opportunity in mainnet.

### Global Price

At the time of writing (2021.08.12 03:17:00 UTC), the following prices are current ATOM and OSMO prices from CoinGecko

ATOM: $15.06: [https://www.coingecko.com/en/coins/cosmos](https://www.coingecko.com/en/coins/cosmos)

OSMO: $1.99: [https://www.coingecko.com/en/coins/osmosis](https://www.coingecko.com/en/coins/osmosis)

![Global Price Osmosis](https://user-images.githubusercontent.com/20435620/129910462-506030f6-7ee8-4d2f-a334-7f97e253449d.png)

This means 1 OSMO is equal to 0.132138114209827 ATOM

**→ Global Price: 0.132138114209827**

### Pool Price

In order to get pool price, you need an information about two reserve tokens amount.

You can use this REST API to get the information

First, let's get the pool information. 

- Query ATOM/OSMO pool uses pool id of 1

    [https://lcd-osmosis.keplr.app/osmosis/gamm/v1beta1/pools/1](https://lcd-osmosis.keplr.app/osmosis/gamm/v1beta1/pools/1) 

    You can see there is an information about two different tokens and their weights inside `poolAssets` array. 

    Note that denom starting with `ibc/{hash}` is a denom that is transferred using IBC protocol.

    You need to use another REST API to get the name of denom.

    ```json
    {
      "pool": {
        "@type": "/osmosis.gamm.v1beta1.Pool",
        "address": "osmo1mw0ac6rwlp5r8wapwk3zs6g29h8fcscxqakdzw9emkne6c8wjp9q0t3v8t",
        "id": "1",
        "poolParams": {
          "swapFee": "0.003000000000000000",
          "exitFee": "0.000000000000000000",
          "smoothWeightChangeParams": null
        },
        "future_pool_governor": "24h",
        "totalShares": {
          "denom": "gamm/pool/1",
          "amount": "82104367121967389456514715"
        },
        "poolAssets": Array[2][
          {
            "token": {
              "denom": "ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2",
              "amount": "1083043337742"
            },
            "weight": "536870912000000"
          },
          {
            "token": {
              "denom": "uosmo",
              "amount": "8147463787407"
            },
            "weight": "536870912000000"
          }
        ],
        "totalWeight": "1073741824000000"
      }
    }
    ```

- Query a denomination trace information

    [https://lcd-osmosis.keplr.app/ibc/applications/transfer/v1beta1/denom_traces/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2](https://lcd-osmosis.keplr.app/ibc/applications/transfer/v1beta1/denom_traces/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2) 

    You can see that the denom is `uatom` from the API.

    ```json
    {
      "denom_trace": {
        "path": "transfer/channel-0",
        "base_denom": "uatom"
      }
    }
    ```

Let's use the fields from the API result to calculate the pool price.

![Pool Price Osmosis](https://user-images.githubusercontent.com/20435620/129910314-4248b3ec-aefc-4f67-8d9c-32f7d5c771ad.png)

This means 1 OSMO is equal to 0.132930119851031 ATOM

→ Pool Price: 0.132930119850976

### Expected Swap Price

Let's say we would like to swap 100 ATOM (100000000uatom) from the pool.

What is the expected amount of OSMO to receive and swap price?

![Expected Swap Price Osmosis 1](https://user-images.githubusercontent.com/20435620/129910000-201c5c9d-ad2e-4e6f-8c2f-31d503246d9d.png)

→ Expected Swap Amount: 752205502.588607647064247 (uosmo)

→ Expected Swap Price: 0.132942393609013

Gap between global price and pool price: 0.005958060084779

![Expected Swap Price Osmosis 2](https://user-images.githubusercontent.com/20435620/129910150-3cf82539-64db-42ca-8d42-55205957de76.png)

In ATOM/OSMO pool case, it is not worth of doing arbitrage since the gap is too small. However, if there is a enough gap that is worthwhile to take the opportunity, then you can send swap transaction to the network in your need. Reference section **6. References** to get to know about how to use available command-line interfaces.

## Osmosis Params

The following parameters are some params that you should take into consideration. 

### SwapFee

 it is managed by the token holders of the pool respectively. You can query pools to know about the information. For example, [https://lcd-osmosis.keplr.app/osmosis/gamm/v1beta1/pools/1](https://lcd-osmosis.keplr.app/osmosis/gamm/v1beta1/pools/1) 

### ExitFee

 it is managed by the token holders of the pool respectively. You can query pools to know about the information. For example, [https://lcd-osmosis.keplr.app/osmosis/gamm/v1beta1/pools/1](https://lcd-osmosis.keplr.app/osmosis/gamm/v1beta1/pools/1) 

### MinPoolAssets

it is a minimum number of tokens when creating a pool

### MaxPoolAssets

It is a maximum number of tokens when creating a pool

### InitPoolSharesSupply

It is a fixed amount of share tokens (`10^18 * 100`) is minted at an initial creation of the pool. 

## References

The `gamm` module is designed to handle assets of a chain using the AMM and its concept of pool shares. It is where core logics of osmosis project locates. 

### Command-line Interface

**Querying commands for the `gamm` module**

```bash
Available Commands:
  estimate-swap-exact-amount-in  Query estimate-swap-exact-amount-in
  estimate-swap-exact-amount-out Query estimate-swap-exact-amount-out
  num-pools                      Query number of pools
  pool                           Query pool
  pool-assets                    Query pool-assets
  pool-params                    Query pool-params
  pools                          Query pools
  spot-price                     Query spot-price
  total-liquidity                Query total-liquidity
  total-share                    Query total-share
```

- num-pools

    ```bash
    # Command-line Interface
    osmosisd q gamm num-pools --output json | jq

    # REST API
    http://localhost:1317/osmosis/gamm/v1beta1/num_pools
    ```

    ```json
    {
      "numPools": "1"
    }
    ```

- pools

    ```bash
    # Command-line Interface
    osmosisd q gamm pools --output json | jq

    # REST API
    http://localhost:1317/osmosis/gamm/v1beta1/pools
    ```

    ```json
    {
      "pools": [
        {
          "@type": "/osmosis.gamm.v1beta1.Pool",
          "address": "osmo1500hy75krs9e8t50aav6fahk8sxhajn9ctp40qwvvn8tcprkk6wszun4a5",
          "id": "1",
          "poolParams": {
            "swapFee": "0.003000000000000000",
            "exitFee": "0.000000000000000000",
            "smoothWeightChangeParams": null
          },
          "future_pool_governor": "24h",
          "totalShares": {
            "denom": "gamm/pool/1",
            "amount": "110000000000000000000"
          },
          "poolAssets": [
            {
              "token": {
                "denom": "uatom",
                "amount": "5503000000"
              },
              "weight": "5368709120"
            },
            {
              "token": {
                "denom": "uosmo",
                "amount": "10994021257"
              },
              "weight": "5368709120"
            }
          ],
          "totalWeight": "10737418240"
        }
      ],
      "pagination": {
        "next_key": null,
        "total": "2"
      }
    }
    ```

- pool <poolID>

    ```bash
    # Command-line Interface
    osmosisd q gamm pool 1 --output json | jq

    # REST API
    http://localhost:1317/osmosis/gamm/v1beta1/pools/1
    ```

    ```json
    {
      "pools": [
        {
          "@type": "/osmosis.gamm.v1beta1.Pool",
          "address": "osmo1500hy75krs9e8t50aav6fahk8sxhajn9ctp40qwvvn8tcprkk6wszun4a5",
          "id": "1",
          "poolParams": {
            "swapFee": "0.003000000000000000",
            "exitFee": "0.000000000000000000",
            "smoothWeightChangeParams": null
          },
          "future_pool_governor": "24h",
          "totalShares": {
            "denom": "gamm/pool/1",
            "amount": "110000000000000000000"
          },
          "poolAssets": [
            {
              "token": {
                "denom": "uatom",
                "amount": "5503000000"
              },
              "weight": "5368709120"
            },
            {
              "token": {
                "denom": "uosmo",
                "amount": "10994021257"
              },
              "weight": "5368709120"
            }
          ],
          "totalWeight": "10737418240"
        }
      ],
      "pagination": {
        "next_key": null,
        "total": "2"
      }
    }
    ```

- pool-params <poolID>

    ```bash
    # Command-line Interface
    osmosisd q gamm pool-params 1 --output json | jq

    # REST API
    /osmosis/gamm/v1beta1/pools/1/params
    ```

    ```json
    {
      "params": {
        "swapFee": "0.003000000000000000",
        "exitFee": "0.000000000000000000",
        "smoothWeightChangeParams": null
      }
    }
    ```

- pool-assets

    ```bash
    osmosisd q gamm pool-assets 1 --output json | jq

    http://localhost:1317/osmosis/gamm/v1beta1/pools/1/tokens
    ```

    ```json
    {
      "poolAssets": [
        {
          "token": {
            "denom": "uatom",
            "amount": "5503000000"
          },
          "weight": "5368709120"
        },
        {
          "token": {
            "denom": "uosmo",
            "amount": "10994021257"
          },
          "weight": "5368709120"
        }
      ]
    }
    ```

- spot-price

    ```bash
    # Command-line Interface
    # Calculation: (B_in / W_in) / (B_out / W_out)
    # swap fee flag does not exist in cli whereas codebase deals with it
    osmosisd q gamm spot-price 1 uatom uosmo --output json | jq
    osmosisd q gamm spot-price 1 uosmo uatom --output json | jq

    # REST API Not Available
    ```

    ```json
    {
      "spotPrice": "0.500000000000000000"
    }
    ```

- total-liquidity

    ```bash
    # Command-line Interface
    osmosisd q gamm total-liquidity --output json | jq

    http://localhost:1317/osmosis/gamm/v1beta1/total_liquidity
    ```

    ```json
    {
      "liquidity": [
        {
          "denom": "uatom",
          "amount": "10000000000"
        },
        {
          "denom": "uosmo",
          "amount": "20000000000"
        }
      ]
    }
    ```

- total-share

    ```bash
    # Command-line Interface
    # initial pool shares is 10^18 * 100 (100000000000000000000)
    osmosisd q gamm total-share 1 --output json | jq

    # NOT WORKING
    http://localhost:1317/osmosis/gamm/v1beta1/pools/1/total_share
    ```

    ```json
    {
      "totalShares": {
        "denom": "gamm/pool/1",
        "amount": "110000000000000000000"
      }
    }
    ```

- estimate-swap-exact-amount-in

    ```bash
    # Command-line Interface
    osmosisd q gamm estimate-swap-exact-amount-in 1 osmo185fflsvwrz0cx46w6qada7mdy92m6kx4qm4l9k 1000000uatom \
    --swap-route-denoms uosmo \
    --swap-route-pool-ids 1 \
    --output json | jq

    # REST API (Not working, need to dig more)
    http://localhost:1317/osmosis/gamm/v1beta1/1/estimate/swap_exact_amount_in?poolId=1&sender=osmo185fflsvwrz0cx46w6qada7mdy92m6kx4qm4l9k&tokenIn=1000000uatom
    ```

    ```json
    {
      "tokenOutAmount": "1993602"
    }
    ```

- estimate-swap-exact-amount-out

    ```bash
    # Command-line Interface
    osmosisd q gamm estimate-swap-exact-amount-out 1 osmo185fflsvwrz0cx46w6qada7mdy92m6kx4qm4l9k 1000000uatom \
    --swap-route-denoms uosmo \
    --swap-route-pool-ids 1 \
    --output json | jq

    osmosisd q gamm estimate-swap-exact-amount-out 1 osmo185fflsvwrz0cx46w6qada7mdy92m6kx4qm4l9k 1000000uosmo \
    --swap-route-denoms uatom \
    --swap-route-pool-ids 1 \
    --output json | jq

    ```

    ```json
    {
      "tokenInAmount": "2005618"
    }
    ```

**Transaction commands for the `gamm` module**

```json
Available Commands:
  create-pool                 create a new pool and provide the liquidity to it
  exit-pool                   exit a new pool and withdraw the liquidity from it
  exit-swap-extern-amount-out exit swap extern amount out
  exit-swap-share-amount-in   exit swap share amount in
  join-pool                   join a new pool and provide the liquidity to it
  join-swap-extern-amount-in  join swap extern amount in
  join-swap-share-amount-out  join swap share amount out
  swap-exact-amount-in        swap exact amount in
  swap-exact-amount-out       swap exact amount out
```

- create-pool

    > create a new pool and provide the liquidity to it

    An example of `plan.json` file.

    ```json
    {
      "weights": "5uatom,5uosmo",
      "initial-deposit": "5000000000uatom,10000000000uosmo",
      "swap-fee": "0.003",
      "exit-fee": "0.00",
      "future-governor": "24h"
    }
    ```

    ```bash
    osmosisd tx gamm create-pool \
    --chain-id localnet \
    --pool-file="pool.json" \
    --from user2 \
    --keyring-backend test \
    --broadcast-mode block \
    --gas 500000 \
    --yes
    ```

- join-pool

    > join a new pool and provide the liquidity to it

    ```bash
    # --share-amount-out: shareRatio is the desired number of shares
    # divided by the total number of shares currently in the pool.
    #
    # http://localhost:1317/osmosis/gamm/v1beta1/pools
    #
    osmosisd tx gamm join-pool \
    --chain-id localnet \
    --from user2 \
    --keyring-backend test \
    --broadcast-mode block \
    --pool-id 1 \
    --share-amount-out 10000000000000000000 \
    --max-amounts-in 500000000uatom \
    --max-amounts-in 1000000000uosmo \
    --yes
    ```

- exit-pool

    > exit a new pool and withdraw the liquidity from it

    ```bash
    osmosisd tx gamm exit-pool \
    --chain-id localnet \
    --from user2 \
    --keyring-backend test \
    --broadcast-mode block \
    --pool-id 1 \
    --share-amount-in 1000000000000000000 \
    --min-amounts-out 50000000uatom \
    --min-amounts-out 100000000uosmo \
    --yes
    ```

- swap-exact-amount-in

    ```bash
    # Command-line Interface
    osmosisd tx gamm swap-exact-amount-in 1000000uatom 1993602 \
    --chain-id localnet \
    --from user2 \
    --keyring-backend test \
    --broadcast-mode block \
    --swap-route-pool-ids 1 \
    --swap-route-denoms uosmo \
    --yes
    ```

- swap-exact-amount-out

    ```bash
    osmosisd tx gamm swap-exact-amount-out 1000000uatom 2005618 \
    --chain-id localnet \
    --from user2 \
    --keyring-backend test \
    --broadcast-mode block \
    --swap-route-pool-ids 1 \
    --swap-route-denoms uosmo \
    --yes
    ```

