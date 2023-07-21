# Tenet multichain upgrade

This upgrade will move funds (wTENET) from the Multichain bridge contract (which is not working anymore: https://twitter.com/multichainorg/status/1679768407628185600) to the Tenet Foundation.

The current ERC20 version will then be replaced 1:1 with a new upgraded version that will be backed by the TENET removed from the Multichain bridge.

Assuming the proposal passes the chain will stop at given upgrade height.

The logs should look something like:

```bash
E[2019-11-05|12:44:18.913] UPGRADE "multichain" NEEDED at height: 2330000:       module=main
```

**Only after** you see this log you can shut down your node, replace binary with the new one and start it again. 
If you do it before upgrade height, you will permanently damage your node’s database.

You can find actual node’s binary release (v11.2.0) here: https://github.com/tenet-org/tenet-mainnet/releases/tag/v11.2.0

Notes:

1. If you are a validator and you miss upgrade block, when you need to upgrade and sync your node to latest height and 
then send `unjail` transaction to participate in consensus again.
2. Remember, that if you are a validator and you will occasionally run 2+ nodes 
with your private key (config/priv_validator_key.json), then your validator will be 
`tombstoned` (banned forever) and punished by 5% of its stake.
3. If you are willing to automate upgrade process you might want to setup `cosmovisor` 
daemon: [https://docs.cosmos.network/v0.46/run-node/cosmovisor.html](https://docs.cosmos.network/v0.46/run-node/cosmovisor.html)

# Installing `cosmovisor` for seamless upgrade

1. `go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest`

2. Init cosmovisor
```bash
export DAEMON_HOME=<<< path to .tenetd folder >>>
export DAEMON_NAME=tenetd
export DAEMON_ALLOW_DOWNLOAD_BINARIES=true

cosmovisor init $(which tenetd)
```
3. Stop your node
4. `cosmovisor run start --home=$DAEMON_HOME`

See full documentation for more information: https://docs.cosmos.network/v0.46/run-node/cosmovisor.html
