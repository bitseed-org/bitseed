# Using Electrum Personal Server (EPS) with a Bitseed node

## About

Electrum is a popular bitcoin wallet that provides users with a great deal of control over their coins and transactions. By default, Electrum syncs wallet balance data using a decentralized network of semi-trusted servers run by volunteers. This is convenient because it means that Electrum users do not have to download the whole blockchain to use the wallet. But it is bad for privacy because random servers run by strangers know the wallet balance information of Electrum users.

Electrum Personal Server (EPS) is a project that makes it easy for an Electrum wallet user to run their own Electrum server, bypassing the need to share wallet information with any third parties. This guide will show you how to install EPS on your Bitseed node and connect it to your wallet.

## Credit

All working instructions are owed credit to the following sources; any mistakes are the fault of the author of this guide alone.

- [Chris Belcher](https://github.com/chris-belcher) and the [Electrum Personal Server](https://github.com/chris-belcher/electrum-personal-server/) project
- [402 Payment Required](https://www.youtube.com/watch?v=1JMP4NZCC5g)
- [Raspi Bolt](https://github.com/Stadicus/guides/blob/master/raspibolt/raspibolt_64_electrum.md) project

## Disclaimer

These instructions work as of the date of the last commit made to this document. If you try to follow the instructions and they do not work, please [create an issue](https://github.com/bitseed-org/bitseed/issues), and if you discover a fix, please [submit a pull request](https://github.com/bitseed-org/bitseed/pulls) that fixes the issue.

## Getting started

SSH into your node, replacing `bitseed.ip.address` with your Bitseed's local IP address.

`ssh bitcoin@bitseed.ip.address`

### Install Python3

`sudo apt-get install python3-setuptools python3-pyqt5 python3-pip`

Type the password printed on your Bitseed documentation and hit `Enter`.

### Modify bitcoin.conf

`cd .bitcoin`

`nano bitcoin.conf`

Add or modify the following lines to match:

```
txindex=1
server=1
# disablewallet=1
```

Note the `rpcuser` and `rpcpassword` shown in the bitcoin.conf file. You will need these later.

Press `ctrl+x`, then `y`, then `Enter` to save the changes and exit.

`cd`

### Download EPS and verify signatures

In the commands below, replace the EPS version with the version you want to install. You can find the latest version on the EPS [Releases](https://github.com/chris-belcher/electrum-personal-server/releases) page.

`wget https://github.com/chris-belcher/electrum-personal-server/archive/eps-v0.1.6.tar.gz`

`wget https://github.com/chris-belcher/electrum-personal-server/releases/download/eps-v0.1.6/eps-v0.1.6.tar.gz.asc`

`wget https://github.com/chris-belcher/electrum-personal-server/raw/master/pgp/pubkeys/belcher.asc`

`gpg --import belcher.asc`

`gpg --verify eps-v0.1.6.tar.gz.asc eps-v0.1.6.tar.gz`

### Unpack and rename EPS directory

`tar -xzf eps-v0.1.6.tar.gz`

`mv electrum-personal-server-eps-v0.1.6 ~/eps-v0.1.6`

### Enter EPS directory and modify EPS conf file

`cd eps-v0.1.6`

`nano config.cfg`

Add wallet Master Public Keys under the `[master-public-keys]` section and/or add individual addresses you want to watch under the `[watch-only-addresses]` section of the `config.cfg` file. Use the same format shown in the examples included in the `config.cfg` file.

Uncomment `rpc_user` and `rpc_password` by removing the `#` symbol in front of these lines then enter the bitcoin.conf RPC username and password noted earlier:

```
rpc_user = username
rpc_password = password
```

Scroll down to where it shows the host IP address. Change it to `0.0.0.0`.

`host = 0.0.0.0`

Press `ctrl+x`, then `y`, then `Enter` to save changes and exit.

### Install EPS

`pip3 install --user .`

`mv config.cfg_sample config.cfg`

`cd`

### Run EPS

`cd .local/bin`

`electrum-personal-server /home/bitcoin/eps-v0.1.6/config.cfg`

`electrum-personal-server-rescan /home/bitcoin/eps-v0.1.6/config.cfg`

Enter the earliest date when you used the wallets you want to scan (make it a month or two earlier to be on the safe side). This could take a while (up to several hours or longer) depending on how many addresses need to be scanned and how far back in the blockchain's history the wallet needs to scan. You can check on progress by opening a new tab, SSH'ing into the node again, and entering:

`cd .bitcoin`

`nano debug.log`

`ctrl+w` then `ctrl+v`

The log will show what the most recent block scanned was and the progress e.g. 0.435 = 43.5% finished.

Once the wallet is done rescanning, run the script to start EPS again:

`electrum-personal-server /home/bitcoin/eps-v0.1.6/config.cfg`

Wait until the message `Listening for Electrum Wallet ...` appears, then go to the next step below. 

### Connect to your node from your Electrum wallet

Install Electrum wallet on your personal computer if you haven't already.

`https://electrum.org/#download`

Start Electrum with this command to ensure the wallet only connects to your Electrum server, replacing `bitseed.ip.address` with the local IP address of your Bitseed node (the same one you use to SSH into your Bitseed).

`electrum --oneserver --server bitseed.ip.address:50002:s`

Electrum should start and connect to your node. The connection light will turn from green to red, and the wallet will fill up with the transaction history and addresses. You can now use your Electrum wallet privately.
