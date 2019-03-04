# Installing and using LND over Tor on Bitseed 3

## About

The Lightning Network is a new "Layer 2" smart contract system built on top of bitcoin that enables micropayments to be made between connected nodes on the network. These micropayments happen "off-chain", eliminating the need to pay a relatively expensive mining fee for each transaction or wait for block confirmations between each transaction and reducing the amount of data that needs to be stored on the blockchain. The Lightning Network protocol includes protective measures to prevent nodes from double-spending or stealing funds held in their channels, making the security guarantees of Lightning Network transactions similar to those of on-chain transactions.

If you are interested in reading more about how the Lightning Network works, I encourage you to read the following documentation resources:

- [Lightning Network Summary](https://lightning.network/lightning-network-summary.pdf)
- [Lightning Network Whitepaper](https://lightning.network/lightning-network-paper.pdf)
- [Lightning Network explainer series by Aaron van Wirdum](https://bitcoinmagazine.com/articles/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel-1464710791/)

## Credit

All working instructions are owed credit to the following sources; any mistakes are the fault of the author of this guide alone.

- [LND installation instructions](https://github.com/lightningnetwork/lnd/blob/master/docs/INSTALL.md#installation)

- [Detailed LND Guide](https://gist.github.com/bretton/0b22a0503a9eba09df86a23f3d625c13)

- [Tor installation instructions](https://www.torproject.org/docs/debian.html.en#ubuntu)

- [Zcash over Tor guide (used here for some of the Tor configuration instructions)](https://github.com/ZcashAnonymous/zcash-tor-manual)

- [LND "Configuring Tor" documentation](https://github.com/wpaulino/lnd/blob/master/docs/configuring_tor.md)

- [LND Basic Commands (originally written for Litecoin, but easily applicable for bitcoin)](https://github.com/ecurrencyhodler/Litecoin-Resources/blob/master/LTC%20Guides/Basic%20lnd%20Commands.md#basic-lnd-commands)

## Disclaimer
LND is not officially supported by Bitseed. Use this guide at your own risk. We will not provide official support if anything goes wrong.

This guide involves using the Linux command line to install and run LND. Users are expected to have basic familiarity with the Linux command line. Beginners are welcome but may get stuck in some places if they do not have the knowledge to self-correct and undo errors.

If you run into any issues you can join the [Bitseed community chatroom](https://riot.im/app/#/room/#bitseed:matrix.org) on Matrix or join the LND community on [Slack](https://join.slack.com/t/lightningcommunity/shared_invite/enQtMzQ0OTQyNjE5NjU1LWRiMGNmOTZiNzU0MTVmYzc1ZGFkZTUyNzUwOGJjMjYwNWRkNWQzZWE3MTkwZjdjZGE5ZGNiNGVkMzI2MDU4ZTE) to ask for help from the open source community.

The instructions in this guide work as of the date of the last commit made to this document. The author was able to install and run LND, deposit funds, open a channel, make a payment, close the channel, and get withdraw the funds back to an external bitcoin wallet. If you try to follow the instructions and they do not work, please [create an issue](https://github.com/bitseed-org/bitseed/issues), and if you discover a fix, please [submit a pull request](https://github.com/bitseed-org/bitseed/pulls) that fixes the issue.

## Getting started

SSH into your node, replacing `bitseed.ip.address` with your Bitseed's local IP address.

`ssh bitcoin@bitseed.ip.address`

Type the password printed on your Bitseed documentation and hit `Enter`.

## Install and run Tor + nyx

Make sure your existing software and systems are up to date.

`sudo apt-get update`

Type the password printed on your Bitseed documentation and hit `Enter`.

`sudo apt-get upgrade`

`sudo apt-get dist-upgrade`

Add the Tor package repo to your apt sources.

`sudo nano /etc/apt/sources.list`

Scroll to the bottom of the file and paste these lines:

> deb https://deb.torproject.org/torproject.org xenial main  
> deb-src https://deb.torproject.org/torproject.org xenial main


Press `ctrl+x`, `y`, and `enter` to save and exit.

Add the Tor signing keys to your GPG keyring:

`curl https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import`

`gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -`

`sudo apt install tor deb.torproject.org-keyring`

Update apt again.

`sudo apt-get update`

Install Tor.

`sudo apt-get install tor`

Make sure Tor is not running.

`sudo service tor stop`

Install nyx, a command-line tool for monitoring Tor.

`sudo apt-get install tor-geoipdb apparmor-utils torsocks`

Edit the Tor configuration file.

`sudo nano /etc/tor/torrc`

Press and hold `CTRL+K` to delete the existing content until the file is blank then copy+paste the following into the file:

> ClientOnly 1  
> SOCKSPort 9050  
> SOCKSPolicy accept 127.0.0.1/8  
> Log notice file /var/log/tor/notices.log  
> ControlPort 9051  
> HiddenServiceStatistics 0  
> ORPort 9001  
> LongLivedPorts 21,22,706,1863,5050,5190,5222,5223,6523,6667,6697,8300,8233  
> ExitPolicy reject *:*  
> DisableDebuggerAttachment 0  

Press `CTRL+X` then `Y` then `Enter` to save and exit.

Start Tor.

`sudo service tor start`

Start nyx.

`sudo -H -u debian-tor nyx`

You can navigate nyx with your arrow keys. Pressing `M` will show the menu. `Q` to quit. `R` to reconnect. Note that Tor can also be stopped or restarted via the nyx menu.

## Install Go

Open a new Tab in Terminal then SSH into your server in the new Tab.

`wget https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz`

`sha256sum go1.11.5.linux-amd64.tar.gz | awk -F " " '{ print $1 }'`

The final output of the command above should be `ff54aafedff961eb94792487e827515da683d61a5f9482f668008832631e5d25`. If it isn't, then the target REPO HAS BEEN MODIFIED, and you shouldn't install this version of Go. If it matches, then proceed to install Go:

`tar -C /usr/local -xzf go1.11.5.linux-amd64.tar.gz`

`export PATH=$PATH:/usr/local/go/bin`

`sudo nano .bashrc`

Scroll to the bottom of the file and paste these lines:

> export GOPATH=$HOME/go  
> export PATH=$PATH:$GOPATH/bin

Press `ctrl+x`, `y`, and `enter` to save and exit.

`source .bashrc`

## Install LND

`go get -d github.com/lightningnetwork/lnd`

`cd $GOPATH/src/github.com/lightningnetwork/lnd`

`make && make install`

## Configure bitcoind to connect with LND and connect to the bitcoin network over Tor

`cd .bitcoin`

`sudo nano bitcoin.conf`

Note the bitcoin rpc user and password saved in this file. You will need these later.

Scroll to the bottom of the file and paste these lines if your bitcoin configuration file doesn't already have them:

> txindex=1  
> server=1  
> rpcport=8332  
> upnp=1  
> daemon=1  
> listen=1  
> listenonion=1  
> onlynet=onion  
> proxy=127.0.0.1:9050  
> bind=127.0.0.1  
> rpcallowip=192.168.1.92  
> rpcallowip=127.0.0.1  
> zmqpubrawblock=tcp://127.0.0.1:28332  
> zmqpubrawtx=tcp://127.0.0.1:28333  

Press `ctrl+x`, `y`, and `enter` to save and exit.

`cd`

## Configure LND to connect with bitcoind

`sudo nano .lnd/lnd.conf`

Press and hold `CTRL+K` to delete the existing content (if any) until the file is blank then copy+paste the following into the file:

> [Application Options]  
> debuglevel=trace  
> maxpendingchannels=100  
>   
> [Bitcoin]  
> bitcoin.active=1  
> bitcoin.node=bitcoind  
> bitcoin.mainnet=1  
>  
> [Bitcoind]  
> bitcoind.rpcuser=REPLACE  
> bitcoind.rpcpass=REPLACE  
> bitcoind.zmqpubrawblock=tcp://127.0.0.1:28332  
> bitcoind.zmqpubrawtx=tcp://127.0.0.1:28333  

Replace "REPLACE" with the bitcoind rpc user and password found in the bitcoin.conf file from the previous section.

Press `ctrl+x`, `y`, and `enter` to save and exit.

`cd`

## Start LND

Start LND with connections running over Tor:

`lnd --tor.active --tor.streamisolation --tor.v3 --listen=localhost`

Open a new Tab in Terminal then SSH into your server in the new Tab.

To create a new wallet run:

`lncli create`

Enter a strong password. Enter the password again to confirm it. Store this password somewhere safe, like a password manager.

LND will ask "Do you have an existing cipher seed mnemonic you want to use? (Enter y/n):"

`n`

LND will say "Input your passphrase you wish to encrypt it (or press enter to proceed without a cipher seed passphrase):"

Enter a strong password. Enter the password again to confirm it. Store this password somewhere safe, like a password manager.

LND will show your seed phrase. Store this somewhere safe, you will need it to recover any funds in your wallet. (Note that the seed is insufficient to recover _lightning channel state_ if your LND node data is lost or corrupted. More on this later.)

Unlock your new wallet:

`lncli unlock`

Enter the wallet password you created earlier.

LND will now finish syncing in the other tab.

## Start using LND

After LND is finished syncing, open a new Tab in Terminal then SSH into your server in the new Tab.

Check to see that LND is running correctly:

`lncli getnetworkinfo

You will see a response that looks like this if LND is running correctly:

```
{
    "graph_diameter": 0,
    "avg_out_degree": 5.173188686555599,
    "max_out_degree": 509,
    "num_nodes": 2581,
    "num_channels": 13352,
    "total_network_capacity": "47654328939",
    "avg_channel_size": 3569077.9612792092,
    "min_channel_size": "1100",
    "max_channel_size": "16777216"
}
```

Setup a new wallet to deposit some bitcoin:

`lncli newaddress p2wkh`

Send a relatively small amount of bitcoin to this address for testing purposes, an amount large enough to pay for on-chain fees and make several test lightning transactions, but not so large that you would be upset if you lost it all due to a bug or some other error.

After your transaction is confirmed you can verify that the deposit was successful with this command:

`lncli walletbalance`

It will return a positive balance if the deposit was successful.

You can now open channels and begin sending payments!

Use `lncli -h` to see what commands are available to use with LND. You can also use `lncli command -h` to get more information about a command.

Here are some commands that will be helpful to know:

**Connect to a Node**

Connecting to a node is how Lightning Network nodes communicate with each other about important information such as node routes.

In a web browser, go to this Lightning Network explorer: https://1ml.com/

Click on a node that you want to connect to and find their URI that’s listed. Make sure the URI has a public key and IP address included.

Now go to your terminal and type the follow command:

`lncli connect <URI>`

Example: 

`lncli connect pubkey@ip.add.re.ss:9735`

**Open a Channel**

Next, fund and open the channel once you’re connected:

`lncli openchannel pubkey amount`

Opening a payment channel is how LN nodes pay one another. The pubkey is the long string before the @ symbol in the URI. The channel amount is denominated in satoshis.

Example:

`lncli openchannel pubkey 200000`

If this fails, either the node you connected to is dead or you sent too little. 

Example failure error message:

`[lncli] rpc error: code = Code(199) desc = chan size of 0.003 BTC is below min chan size of 0.1 BTC`

A sucessful transaction will return a txid.

Example successful transaction:

```
{
	"funding_txid": "ae67845876ec41d24638375966076d6ay875e9224243464a5463838cggf75asueb3"
}
```

You should see that it is a “pending channel” if you enter this command:

`lncli pendingchannels`

After 3 to 6 confirmations your channel will leave the pending state and show as open. You can check the balance of open channels with this command:

`lncli channelbalance`

**Send a Payment**

Currently, the only way to pay is if you have the invoice of the node you are paying.  Ask them to send you an invoice.  Then enter this command:

`lncli sendpayment --pay_req=invoice`

Example:

`lncli sendpayment --pay_req=lnbc30n1pdvnvhypp5zegwluf8ptul93lw5c9az04g7d4wtw9l07tsq55m93u9uyhtlxesdqqcqzjq3rycdgt0tqx9f9wgnre8mjt2vkhj8thzvvjs0tq57cvdlkwpclxnez0f3ev2sxnpcs8tjt6sjpea03w0z4qhv7sq9r4ywk8wuczd89qqe9rsmj`

Note that the invoice string is the same in both commands.

**Close a channel**

First, find and select which channel you want to close:

`lncli listchannels`

Then look for their "channelpoint". Here's an example of what that looks like:

`428d5acbef17418f9849fe736f53f2bd830563f386d3d601ab0b37a38d98b1f8:5` 

Then close the channel following this syntax:

`lncli closechannel <chainpoint x>`

Example:

`lncli --chain=litecoin closechannel 428d5acbef17418f9849fe736f53f2bd830563f386d3d601ab0b37a38d98b1f8 5`

## Update LND

`cd $GOPATH/src/github.com/lightningnetwork/lnd`

`git pull`

`make clean && make && make install`

## Shut down

`lnd stop`

`bitcoin-cli stop`

## Open questions

After writing this guide, I still have some open questions about using Lightning with my Bitseed:

- Is there a desktop UI that I can "pair" with the bitcoind + LND node running on my Bitseed so I don't have to SSH into Bitseed and do all this from the command line?

- What kind of monitoring tools are available to make sure that my nodes stay online and don't get force-closed or have my bond seized?

- How do I safely shut down my LND node in a way where I don't risk losing money?

- How do I see and change what mining fee rate is used to open and close LND channels?

- When I opened a channel, I selected 300000 as the amount. However the `channelbalance` shown after the transaction was confirmed showed `297120`. Where did the missing satoshis go?

- How long exactly do users need to wait for channels to open and close before the funds are "free" to spend?

- How do I deal with uncooperative peers or prevent getting robbed by a peer? Does the node handle all of this for me?

- How do I get an incoming channel to my LND node when it is running over Tor?

- How do I backup my channel states from the Bitseed node so that I can recover if the data get corrupted or something?

- I see a lot of action happening in the console when I watch LND while it's running, even though I'm not making any transactions. What is all this data my node is showing?

- Does LND have official, comprehensive usage docs that would answer these questions (and others that I'm sure would come up)? This seems like it would be a useful resource to have vs cobbling together instructions from a bunch of different sources and figuring things out by trial and error, as I did while writing this guide.
