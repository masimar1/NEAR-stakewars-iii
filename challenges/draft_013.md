# Stake Wars: Episode III. Challenge 013

- Published on: 2022-08-11
- Updated on: 2022-08-11
- Submitted by: Everstake
- Rewards: 25 UNP & 10 DNP

## Description

In this challenge, participants will learn how to update their node, migrate keys, and set up a **BACKUP** node. 

Please note that this challenge includes multiple parts: 
- **Best Practises**: is an informational section that outlines the optimal way of updating a validator node (no points) 
- **Backup node**: in this section participants will learn how to migrate validator keys. Only this part of the challenge will be evaluated.  (25 UNP & 10 DNP)  

### Acceptance Criteria

- Run a backup node
- Successfully migrate validator keys from the **MAIN** node to the **BACKUP** node

### Challenge submission

For submission please fill out the [form](https://docs.google.com/forms/d/e/1FAIpQLSfZV6_SUpdAMlOOjpwQSVa0xUcvCjO5iiNG3k9KrGDvCEEw3w/viewform?usp=sf_link)  

To submit your results make sure to include the following: 

Please include a screenshot of logs that shows that the `public_key` has been changed on the main & backup node and timestamps from the validator and backup nodes. Make sure to include the `peer_id` and that it matches `public_key` in `/<WORK_DIR>/mainnet/node_key.json` & `../<VALIDATOR_KEYS>/node_key.json`

## Best Practices

**Before updating your node make sure that you have checked**

1. An official announcement for the update.
2. Release Notes on Github.  
Be aware of *DB migration or other breaking changes*
3. Validator chats.  
Review *how many validators have been updated?  
Were there any noticeable problems?*
4. Update testnet/backup node.   
*Spend some time testing it on your backup node before moving to the main validator node.* 


⚠️ **We recommend that you keep the previous versions of compiled binaries in case of an urgent downgrade**


### Update binary

- Create a new directory e.g `sources` & *clone* or *update* `nearcore` project from GitHub

```bash
cd sources
git clone https://github.com/near/nearcore
cd nearcore
git fetch origin --tags
```

- Checkout to the branch you need. The latest unstable release is recommended if you are running on the **testnet** and the latest stable version is recommended if you are running on the **mainnet**. Please check the [releases page on GitHub](https://github.com/near/nearcore/releases).

```bash
git checkout <version>
```

- Compile `nearcore` binary

```bash
make release
```

```bash
./target/release/neard --version
```


⚠️ **Don’t forget to check version and hash of the binary, it should be the same as displayed on [releases page](https://github.com/near/nearcore/releases)**


- Replace binary in your work directory

```bash
cp ./target/release/neard <WORK_DIR>/bin/
```

- Restart the node and check the log file

**Helpful links:** [https://near-nodes.io/validator/validator-bootcamp](https://near-nodes.io/validator/validator-bootcamp)

### Task

*Try to use different folders for binaries & sources for version check*

## Backup node

In case of missed blocks or chunks the validator can be removed from the active validation set in the next auction. If you want to secure your validator NEAR node from a high downtime you can deploy a **backup** server with the preconfigured NEAR node. Having two provisioned nodes allows you to quickly switch from one server to another by migrating your validator keys, so you can continue producing blocks with minimal downtime. If the nearcore release has long database migration, or you must maintain/scale the server, we also recommend migrating your validator keys to the backup node. 

Always analyze your node activities and be ready to determine a problem and move the *main validator* keys to the **backup** node. 

### Validator keys migration

Firstly, you need to have a backup server, a NEAR node with a new `node_key.json` and one more generated `validator_key.json`. To switch quickly, prepare folders for the validator keys and reserve keys 


⚠️ **Be careful! Don't let two nodes run with the same keys!  
This will make your node slashed. This means that your node will be excluded from the active validation set and your funds may be burnt.**


If the server is **unavailable** (e.g. no network connection) deal with the hosting provider, get access and **only** then start the key migration procedure! Always make sure that the **MAIN** node doesn’t work 

If the server is **available**

1. Stop the **MAIN** node
2. Make sure the **MAIN** node isn’t working

```bash
ps aux | grep neard
netstat -tlpn | grep neard
```

3. Replace main validator keys on the **MAIN** node with the reserve ones (optional, if you are sure the node won’t start accidentally)

```bash
cd mainnet
rm validator_key.json node_key.json
cp ../<RESERVE_KEYS>/validator_key.json ../<RESERVE_KEYS>/node_key.json .
```

4. Stop the **BACKUP** node
5. Make sure the **BACKUP** node isn’t working

```bash
ps aux | grep neard
netstat -tlpn | grep neard
```

6. Replace reserve keys on the **BACKUP** node with main validator 

```bash
cd mainnet
rm validator_key.json node_key.json
cp ../<VALIDATOR_KEYS>/validator_key.json ../<VALIDATOR_KEYS>/node_key.json .
```

7. Check the `validator_key.json` and `node_key.json`

You can check file identity using `md5sum` command for `validator_state.json` and `node_key.json` on both nodes. If the checksums match, then the files are identical.

Or you use `diff` command

```bash
diff <WORK_DIR>/mainnet/validator_key.json ../<RESERVE_KEYS>/validator_key.json
diff <WORK_DIR>/mainnet/node_key.json ../<RESERVE_KEYS>/node_key.json

diff <WORK_DIR>/mainnet/validator_key.json ../<VALIDATOR_KEYS>/validator_key.json
diff <WORK_DIR>/mainnet/node_key.json ../<VALIDATOR_KEYS>/node_key.json
```

8. Start the **BACKUP** node

After starting in the node log, you can see the line

```bash
Feb 16 19:12:27.593  INFO stats: Server listening at peer_id=ed25519:Xx1XXX...XXXXxxx addr=0.0.0.0:24567
Feb 16 19:12:42.164  INFO stats: #59664178 Downloading blocks 100.00% (6)   4/3/40 peers ⬇ 726.1kiB/s ⬆ 2.3MiB/s 0.00 bps 0 gas/s CPU: 0%, Mem: 0 B
```

`peer_id` must be equal to `public_key` in `/<WORK_DIR>/mainnet/node_key.json` & `../<VALIDATOR_KEYS>/node_key.json`

You can also see you are a validator when in the logs of the **BACKUP** node you see "Validator |" 

```bash
2022-07-29T13:37:03.984160Z  INFO stats: #70892058 F6jTU9iyXRJ6jiQq8XUP6ENpzvHZ8FfYJpuS8w9zHb82 Validator | 100 validators 35 peers ⬇ 1.53 MB/s ⬆ 2.24 MB/s 0.80 bps 22.2 Tgas/s CPU: 39%, Mem: 6.91 GB
2022-07-29T13:37:13.984741Z  INFO stats: #70892066 2HwdtnRkosQ9mLkmKZpa3X28NNtzajfvct2YBBg63QW2 Validator | 100 validators 35 peers ⬇ 1.56 MB/s ⬆ 2.21 MB/s 0.80 bps 66.8 Tgas/s CPU: 38%, Mem: 6.91 GB
```

Simple logs

```bash
2022-07-29T13:44:20.905273Z  INFO stats: #70892419 8uHiGM2CuibYAyA1Sry15M9vcyyXrZD8CG3nf8nvetdU 100 validators 36 peers ⬇ 1.35 MB/s ⬆ 1.30 MB/s 0.80 bps 25.8 Tgas/s CPU: 42%, Mem: 6.01 GB
2022-07-29T13:44:30.905814Z  INFO stats: #70892427 4C7VPH79nx6auKD9JpPK81YtQmZAq35BC2JVJxNJgeWy 100 validators 36 peers ⬇ 1.36 MB/s ⬆ 1.31 MB/s 0.80 bps 22.9 Tgas/s CPU: 37%, Mem: 5.81 GB
```

## Update log

Updated 2022-08-11: Creation
