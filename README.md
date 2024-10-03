# crunch_bot

Crunch is a command line interface (CLI) for easily automating staking reward payouts on Substrate-based chains.

Crunch allows you to:
- Request staking rewards for a single or a list of validators at the end of each era or every X hours.
- Receive notifications about the amount and rate of total staking rewards received by each validator and their nominators.
- Obtain statistics for each validator, such as the inclusion ratio, claimed rewards ratio, epoch points trend, and activity for the current epoch.
- Check for any unclaimed eras for a given validator.

You can find all the features of Crunch on the official GitHub page.

---

## Installing Crunch Bot

Create a directory and download the binary file:
```bash
mkdir $HOME/.kusama/crunch-bot && cd $HOME/.kusama/crunch-bot
wget https://github.com/turboflakes/crunch/releases/download/v0.18.1/crunch
chmod +x $HOME/.kusama/crunch-bot/crunch
cp $HOME/.kusama/crunch-bot/crunch /usr/local/bin/
crunch --version
#crunch 0.18.1
```

When running on Ubuntu 22.04, you might encounter an OpenSSL library error. You can manually install it:
```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb
dpkg -i libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb
```

---

## Configuring `.env`

Create the main configuration file `.env` and set it up. Below is a simplified example configuration file using a single Kusama validator stash wallet. You can find the full functionality [here](#).

By default, Crunch will try to connect to `ws://IP:9944`. If you are using RPC for connection, add the following flag:  
`CRUNCH_SUBSTRATE_WS_URL=ws://IP:9944`

Replace:
- `CRUNCH_STASHES` with your validator's stash.
- `CRUNCH_MATRIX_USER` with your main Matrix account.
- `CRUNCH_MATRIX_BOT_USER` with your additional Matrix account, which you will need to create in advance, and from which you will receive messages.
- `CRUNCH_MATRIX_BOT_PASSWORD` with the password for your additional Matrix account.

```bash
nano $HOME/.kusama/crunch-bot/.env
```

**.env Example Configuration**
```bash
# ----------------------------------------------------------------
# crunch CLI configuration variables 
# ----------------------------------------------------------------
#
CRUNCH_STASHES=JHRygZAwLR5oScvgF6QcLLR3sFx9GFZWYzirx2cvgF6QcLL
CRUNCH_LIGHT_CLIENT_ENABLED=true
CRUNCH_MAXIMUM_PAYOUTS=4
CRUNCH_MAXIMUM_HISTORY_ERAS=4
CRUNCH_MAXIMUM_CALLS=3
#
# ----------------------------------------------------------------
# Matrix configuration variables
# ----------------------------------------------------------------
#
CRUNCH_MATRIX_DISABLED=false
CRUNCH_MATRIX_PUBLIC_ROOM_DISABLED=true
CRUNCH_MATRIX_USER=@matrix:matrix.org
CRUNCH_MATRIX_BOT_USER=@matrix_bot:matrix.org
CRUNCH_MATRIX_BOT_PASSWORD="password_bot"
#
# ----------------------------------------------------------------
# Nomination Pools configuration variables
# ----------------------------------------------------------------
CRUNCH_POOL_IDS=2
# 1 DOT = 10000000000 PLANCKS
# 1 KSM = 1000000000000 PLANCKS
CRUNCH_POOL_COMPOUND_THRESHOLD=1000000000000
CRUNCH_POOL_ONLY_OPERATOR_COMPOUND_ENABLED=true
```

You need to create a separate wallet for transaction payments. Fund this wallet and write its seed phrase to `.private.seed`:
```bash
echo "your private seed">> .private.seed
```

---

## Running Crunch Bot

Now we can view which rewards from the last 84 eras have been claimed and which have not.

**For Kusama:**
```bash
crunch kusama view
```
Note: The `crunch view` mode only logs information to the terminal.

---

## Creating a Service File

- `era` - Runs Crunch right after the EraPaid event in the chain.
- `daily` - Repeats the Crunch task every 24 hours.
- `turbo` - Repeats the Crunch task every 6 hours.
- `once` - Attempts to make a payout once and then exits.

```bash
tee /etc/systemd/system/crunch.service > /dev/null <<EOF
[Unit]
Description=Kusama Crunch Bot

[Service]
User=$USER
ExecStart=/usr/local/bin/crunch kusama --config-path $HOME/.kusama/crunch-bot/.env rewards daily --seed-path $HOME/.kusama/crunch-bot/.private.seed
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable crunch
systemctl restart crunch && journalctl -u crunch -f -o cat
```

---

## Useful Commands

**View logs:**
```bash
journalctl -u crunch -f -o cat
```

**Remove Crunch Bot:**
```bash
systemctl stop crunch
systemctl disable crunch
rm /etc/systemd/system/crunch.service
systemctl daemon-reload
cd $HOME
rm -rf .kusama/crunch-bot
```
