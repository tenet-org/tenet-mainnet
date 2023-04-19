## Initialize Chain

### 1. Clone home directory into ~/.tenetd

```bash
git clone https://github.com/tenet-org/tenet-mainnet.git ~/.tenetd
```

### 2. Start and sync node
```bash
tenetd start
```

### 3. Create Your Validator

Your node consensus public key (tenetvalconspub...) can be used to create a new validator by staking atenet tokens. You can find your validator pubkey by running:

```bash
tented tendermint show-validator
```

You can generate an account for validator by running:

```bash
tenetd keys add my_val_key
```

*Do not forget to top up created address with sufficient funds*

To create your validator on mainnet, just use the following command:

```bash
tenetd tx staking create-validator \
  --amount=1000000000000000000atenet \
  --pubkey=$(tenetd tendermint show-validator) \
  --moniker="choose a moniker" \
  --commission-rate="0.05" \
  --commission-max-rate="0.10" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --gas-prices="20000000000atenet" \
  --gas-adjustment=1.5
  --from=my_val_key
```

### Track Validator Signing Information

In order to keep track of a validator's signatures in the past you can do so by using the signing-info command:

```
tenetd query slashing signing-info <validator-pubkey>
```

### Unjail Validator
When a validator is "jailed" for downtime, you must submit an Unjail transaction from the operator account in order to be able to get block proposer rewards again (depends on the zone fee distribution).

```
tenetd tx slashing unjail \
  --from=<key_name>
```

Confirm Your Validator is Running
Your validator is active if the following command returns anything:

```
tenetd query tendermint-validator-set | grep "$(tenetd tendermint show-address)"
```


You should now see your validator in one of Tenet explorers. You are looking for the bech32 encoded address in the `~/.tenetd/config/priv_validator.json` file.

### Halting Your Validator
When attempting to perform routine maintenance or planning for an upcoming coordinated upgrade, it can be useful to have your validator systematically and gracefully halt. You can achieve this by either setting the halt-height to the height at which you want your node to shutdown or by passing the `--halt-height` flag to tenetd. The node will shutdown with a zero exit code at that given height after committing the block.

### Minimum Requirements
To run mainnet or testnet validator nodes, you will need a machine with the following minimum hardware requirements:

- 4 or more physical CPU cores
- At least 500GB of SSD disk storage
- At least 16GB of memory (RAM)
- At least 100mbps network bandwidth

As the usage of the blockchain grows, the server requirements may increase as well, so you should have a plan for updating your server as well.

## Common Problems
#### Problem #1: My validator has voting_power: 0
Your validator has become jailed. Validators get jailed, i.e. get removed from the active validator set, if they do not vote on 500 of the last 10000 blocks, or if they double sign.

If you got jailed for downtime, you can get your voting power back to your validator. First, if tenetd is not running, start it up again:

`tenetd start`

Wait for your full node to catch up to the latest block. Then, you can unjail your validator

Lastly, check your validator again to see if your voting power is back.

`tenetd status`

You may notice that your voting power is less than it used to be. That's because you got slashed for downtime!

#### Problem #2: My node crashes because of too many open files
The default number of files Linux can open (per-process) is 1024. tenetd is known to open more than 1024 files. This causes the process to crash. A quick fix is to run ulimit -n 4096 (increase the number of open files allowed) and then restart the process with tenetd start. If you are using systemd or another process manager to launch tenetd this may require some configuration at that level. A sample systemd file to fix this issue is below:

```
# /etc/systemd/system/tenetd.service
[Unit]
Description=Tenet Node
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/go/bin/tenetd start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
