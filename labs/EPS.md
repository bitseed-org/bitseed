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
EPS is not officially supported by Bitseed. Use this guide at your own risk. We will not provide support if anything goes wrong.

This guide involves using the Linux command line to install and run EPS. Users are expected to have basic familiarity with the Linux command line. Beginners are welcome but may get stuck in some places if they do not have the knowledge to self-correct and undo errors.

If you run into any issues you can join the [Bitseed community chatroom](https://riot.im/app/#/room/#bitseed:matrix.org) on Matrix or join the #electrum channel on [Freenode IRC](https://webchat.freenode.net/) to ask for help from the open source community.

The instructions in this guide work as of the date of the last commit made to this document. If you try to follow the instructions and they do not work, please [create an issue](https://github.com/bitseed-org/bitseed/issues), and if you discover a fix, please [submit a pull request](https://github.com/bitseed-org/bitseed/pulls) that fixes the issue.

## Getting started

SSH into your node, replacing `bitseed.ip.address` with your Bitseed's local IP address.

`ssh bitcoin@bitseed.ip.address`

Type the password printed on your Bitseed documentation and hit `Enter`.

### Install Python3

`sudo apt-get install python3-setuptools python3-pyqt5 python3-pip -y`

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

Note: If you want to use EPS with your Ledger wallet, skip ahead to the "Appendix I. Use EPS with Ledger wallet" section below, then come back here once you have your Ledger wallet master public key. If you want to use EPS with private keys stored in your Electrum wallet, skip ahead to the "Appendix II. Use EPS with Electrum wallet" section below, then come back here once you have your Electrum wallet master public key.

`cd eps-v0.1.6`

`nano config.cfg_sample`

Add wallet Master Public Keys under the `[master-public-keys]` section and/or add individual addresses you want to watch under the `[watch-only-addresses]` section of the `config.cfg_sample` file. Use the same format shown in the examples included in the `config.cfg_sample` file.

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

Enter the earliest date when you used the wallets you want to scan (make it a month or two earlier to be on the safe side). This could take a while (up to several hours or longer) depending on how many addresses need to be scanned and how far back in the blockchain's history the wallet needs to scan. 

Note: you will need to stop EPS with `ctrl+c` and enter the above two commands each time you add a new Master Public Key to your EPS config.cfg file, so that the server knows to look for the new addresses.

You can check on rescan progress by opening a new tab, SSH'ing into the node again, and entering:

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

Electrum should start and connect to your node. The connection light will turn from red to green, and the wallet will fill up with the transaction history and addresses after it is done syncing (syncing may take a minute or two). Once the wallet is done syncing, you can now use your Electrum wallet privately with your own Electrum Personal Server running on your Bitseed node.


### Appendix I. Use EPS with Ledger wallet

**If you already have used accounts in your Ledger wallet**

If there are already accounts in your Ledger wallet that you have used with the Ledger app, then these wallets can be treated as "compromised" - or at least "associated" - from a privacy perspective. The Ledger company already knows these wallets are tied to the same user. You have two choices regarding what to do with these accounts:

1. Run the coins in them through a mix network like that offered by JoinMarket or Wasabi Wallet, then transfer the resulting coins back to a fresh Ledger account (described below), or
2. Keep these accounts segregated from the private accounts you're about to create and just remember that they have been associated with each other by the Ledger company.

Regardless of which choice you make, you will need fresh new accounts to work off of that haven't been associated by the Ledger company. To create these, you're going to need a new Master Public Key that hasn't been sent to Ledger's servers yet. This means starting over with a fresh Master Public Key on your Ledger. The easiest way to do this is by setting a new passphrase + PIN as detailed [on Ledger's website](https://support.ledger.com/hc/en-us/articles/115005214529-Advanced-passphrase-security) under "Option 1 - Attach to PIN code". You will want to be sure to save the passphrase in a safe place, perhaps the same way you've saved the seed phrase when you first set up your Ledger. You will need both the seed phrase and the passphrase to regain access to your new accounts. And of course you will need the PIN that you set with your new passphrase to access the new accounts when regularly using your Ledger.

After creating a new passphrase and PIN, you are now ready to connect your fresh Ledger accounts to your Electrum wallet. Remember to never use your new passphrase and PIN to log in to the Ledger app, or else all the effort expended to gain privacy by using your EPS will be for naught. Proceed to follow the rest of the instructions in the "If you have not already used accounts in your Ledger wallet" section below.

**If you have not already used accounts in your Ledger wallet**

Install Electrum wallet on your personal computer if you haven't already.

`https://electrum.org/#download`

Start Electrum with this command to ensure the wallet only connects to your Electrum server, replacing `bitseed.ip.address` with the local IP address of your Bitseed node (the same one you use to SSH into your Bitseed).

`electrum --oneserver --server bitseed.ip.address:50002:s`

Follow the [instructions](https://support.ledger.com/hc/en-us/articles/115005161925-Set-up-and-use-Electrum) on the Ledger website for connecting to Electrum. Use your PIN to log in to your Ledger and connect it to Electrum. You will get to a page that asks you to select the type of wallet and the account number. Here you can choose the type of wallet you want (p2pkh "legacy", p2wpkh-p2sh "legacy segwit", or p2wpkh "bech32 aka native segwit") and enter the derivation path. You can leave the derivation path as-is. If you later choose to create separate accounts you can iterate the derivation path e.g. by changing the path from `49'/0'/0'/` to `49'/0'/1'/`, and then later to `49'/0'/2'/` for a third account, and so on.

After you create your wallet in Electrum, you'll notice that nothing happens. That's because Electrum Personal Server doesn't know anything about your wallet yet. In the Electrum wallet, click Wallet > Information in the Electrum menu bar and copy the Master Public Key. Then go back to the "Enter EPS directory and modify EPS conf file" section above and proceed from there.

### Appendix II. Use EPS with Electrum wallet

**If you already have used accounts in your Electrum wallet**

If there are already accounts in your Electrum wallet that you have used with default Electrum settings, then these wallets can be treated as "compromised" - or at least "associated" - from a privacy perspective, even if you've only ever connected the wallet over Tor. The Electrum servers you have connected to already know these wallets are tied to the same user. You have two choices regarding what to do with these accounts:

1. Run the coins in them through a mix network like that offered by JoinMarket or Wasabi Wallet, then transfer the resulting coins back to a fresh Electrum wallet (described below), or
2. Keep these accounts segregated from the private accounts you're about to create and just remember that they have been associated with each other by the Electrum servers.

Regardless of which choice you make, you will need fresh new accounts to work off of that haven't been associated by the Electrum servers. To create these, you're going to need a new master public key that hasn't been sent to any third-party Electrum servers yet. This means starting over with a fresh Electrum wallet.

Start Electrum with this command to ensure the wallet only tries to connect to your EPS, replacing `bitseed.ip.address` with the local IP address of your Bitseed node (the same one you use to SSH into your Bitseed).

`electrum --oneserver --server bitseed.ip.address:50002:s`

The connection light will be red because your EPS is not running yet. This is ok. Click File > New/Restore in the Electrum menu bar then create a new wallet. Once the new wallet is open, click Wallet > Information in the Electrum menu bar and copy the Master Public Key. Then go back to the "Enter EPS directory and modify EPS conf file" section above and proceed from there.

**If you have not already used accounts in your Electrum wallet**

Start Electrum with this command to ensure the wallet only tries to connect to your EPS, replacing `bitseed.ip.address` with the local IP address of your Bitseed node (the same one you use to SSH into your Bitseed).

`electrum --oneserver --server bitseed.ip.address:50002:s`

You will be prompted to create a new wallet. Once the new wallet is open, click Wallet > Information in the Electrum menu bar and copy the Master Public Key. Then go back to the "Enter EPS directory and modify EPS conf file" section above and proceed from there.

### Appendix III. Starting Electrum wallet when using a Linux AppImage

1. Download the AppImage file from https://electrum.org.
2. Open the directory that the AppImage file was downloaded to in your Linux terminal e.g. `cd Downloads`.
3. Enter `chmod a+x electrum-file-name.AppImage`, replacing electrum-file-name with the actual name of the Electrum AppImage file you downloaded (minus the .AppImage extension, which is already included in the example).
4. Enter `./electrum-file-name.AppImage --oneserver --server bitseed.ip.address:50002:s` to start Electrum wallet connecting only to your own EPS instance.
