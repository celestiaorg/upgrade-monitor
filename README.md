# Upgrade Monitor

Upgrade monitor is a binary that monitors that status of upgrades on a Celestia network. Upgrade monitor can optionally publish a signed transaction to trigger an upgrade if it detects that enough validators have signalled for that upgrade.

## Prerequisites

1. [Install Go](https://go.dev/doc/install) 1.21.5
1. Determine which Celestia network you would like to monitor upgrades on. Get a GRPC endpoint for a consensus node on that network.

> [!TIP]
> upgrade-monitor is only applicable for consensus nodes that are running app version >= 2. At the time of writing, Celestia mainnet and testnets are on app version 1 so upgrade-monitor can only be used on a local devnet.

## Install

```shell
# Build the binary from source
go build .

# Verify the binary is working
./upgrade-monitor --help
```

## Usage

1. In one terminal tab, start a [celestia-app](https://github.com/celestiaorg/celestia-app) local devnet.

    ```shell
    cd celestia-app

    # Upgrade monitor can only be run on app version >= 2 so checkout main and install celestia-appd.
    git checkout main
    make install

    # This will start a GRPC server at 0.0.0.0:9000
    ./scripts/single-node.sh
    ```

1. [optional] In a new terminal tab, prepare the try upgrade transaction

    ```shell
    # Export environment variables. Modify these based on your needs.
    export CHAIN_ID="private"
    export KEY_NAME="validator"
    export KEYRING_BACKEND="test"
    export FROM=$(celestia-appd keys show $KEY_NAME --address --keyring-backend $KEYRING_BACKEND)

    # Prepare the unsigned transaction.
    celestia-appd tx upgrade try-upgrade --from $FROM --keyring-backend $KEYRING_BACKEND --fees 420utia --generate-only > unsigned_tx.json

    # Sign the unsigned transaction.
    celestia-appd tx sign unsigned_tx.json --from $FROM --keyring-backend $KEYRING_BACKEND --chain-id $CHAIN_ID --output-document signed_tx.json
    ```

1. Run the upgrade-monitor.

    ```shell
    # Run the binary without auto publishing
    ./upgrade-monitor

    # [optional] Run the binary with auto publishing
    ./upgrade-monitor --auto-publish signed_tx.json
    ```
