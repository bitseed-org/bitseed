Submitted by @milobitnode:matrix.org in the [#bitseed:matrix.org](https://riot.im/app/#/room/!KanLGcjxWlMDCxddGS:matrix.org/$15227317533871887KJsEw:matrix.org) channel on Matrix.

Create issues or pull requests in this repository to help improve this guide!

# LND on your Bitseed 3

This guide is a totally free and non professional test on my personal Bitseed 3, it works out well
so I give it here, but please kindly note I’m not a professional developer, I’m not working in the
Bitseed Team, and I can’t and I’ll not be able to give you further help. Try and use with caution,
and note that LND is still under development and that we’re still very early. Don’t use mid or
huge bitcoins quantities at this time, only small quantities, and consider it lost.

Note that this guide is solely possible because of the work of Stadicus, that released his guide for
LND on a Raspberry Pi, that I use for the LND installation on my node.

SSH into the node then enter:

`sudo nano /usr/local/bin/getpublicip.sh`

> #!/bin/bash  
> #RaspiBolt LND Mainnet: script to get public ip address  
> #/usr/local/bin/getpublicip.sh  
> echo 'getpublicip.sh started, writing public IP address every 10 minutes into /run/publicip'  
> while [ 0 ];  
> do  
> printf "PUBLICIP=$(curl -vv ipinfo.io/ip 2> /run/publicip.log)\n" > /run/publicip;  
> sleep 600  
> done;  

Make it executable:  

`sudo chmod +x /usr/local/bin/getpublicip.sh`  

Create corresponding systemd unit, save and exit:  

`sudo nano /etc/systemd/system/getpublicip.service`  

> #RaspiBolt LND Mainnet: systemd unit for getpublicip.sh script
> #/etc/systemd/system/getpublicip.service
> [Unit]
> Description=getpublicip.sh: get public ip address from ipinfo.io
> After=network.target
> [Service]
> User=root
> Group=root
> Type=simple
> ExecStart=/usr/local/bin/getpublicip.sh
> ExecStartPost=/bin/sleep 5
> Restart=always
> RestartSec=600
> TimeoutSec=10
> [Install]
> WantedBy=multi-user.target

Enable systemd startup:

`sudo systemctl enable getpublicip`  
`sudo systemctl start getpublicip`  
`sudo systemctl status getpublicip`  

Check if data file has been created:

`cat /run/publicip`

Install LND:

`cd /home/bitcoin/download`  
`wget https://github.com/lightningnetwork/lnd/releases/download/v0.4.2-beta/lnd-linux-386-
v0.4.2-beta.tar.gz`  
`tar -xzf lnd-linux-386-v0.4.2-beta.tar.gz`  
`ls -la`  
`sudo install -m 0755 -o root -g root -t /usr/local/bin lnd-linux-386-v0.4.2-beta/*`  
`lnd --version`  

> lnd version 0.4.2-beta

Note: for further updates, when released by LND Labs, all you need to do is to make this process
again, with the link to the latest released version, and it will update to the new version.

## LND configuration

Create the LND configuration file and paste the following content (adjust to
your alias). Save and exit.

`nano /home/bitcoin/.lnd/lnd.conf`  

> #RaspiBolt LND Mainnet: lnd configuration
> #/home/bitcoin/.lnd/lnd.conf
> [Application Options]
> debuglevel=info
> debughtlc=true
> maxpendingchannels=5
> alias=YOUR_NAME [LND]
> color=#68F442
> [Bitcoin]
> bitcoin.active=1
> #enable either testnet or mainnet
> #bitcoin.testnet=1
> bitcoin.mainnet=1
> bitcoin.node=bitcoind
> [autopilot]autopilot.active=1
> autopilot.maxchannels=5
> autopilot.allocation=0.6

Create LND systemd unit and with the following content:

`sudo nano /etc/systemd/system/lnd.service`

> #RaspiBolt LND Mainnet: systemd unit for lnd  
> #/etc/systemd/system/lnd.service  
> [Unit]  
> Description=LND Lightning Daemon  
> Wants=bitcoind.service  
> After=bitcoind.service  
> #for use with sendmail alert  
> #OnFailure=systemd-sendmail@%n  
> [Service]  
> #get var PUBIP from file  
> EnvironmentFile=/run/publicip  
> ExecStart=/usr/local/bin/lnd --externalip=${PUBLICIP}  
> PIDFile=/home/bitcoin/.lnd/lnd.pid  
> User=bitcoin  
> Group=bitcoin  
> LimitNOFILE=128000   
> Type=simple  
> KillMode=process  
> TimeoutSec=180  
> Restart=always  
> RestartSec=60  
> [Install]  
> WantedBy=multi-user.target  

Enable and start LND:  
`sudo systemctl enable lnd`  
`sudo systemctl start lnd`  
`systemctl status lnd`    

Monitor the LND logfile in realtime. Exit with Ctrl+C.

`sudo journalctl -f -u lnd`  

## LND wallet setup

Once LND is started, the process waits for us to create the integrated Bitcoin wallet (it does not use the bitcoind wallet). 

Create the mainnet wallet"

`lncli create`  

These 24 words, combined with your passphrase (optional password) is all that you need to restore your Bitcoin wallet and all Lighting channels. The current state of your channels, however, cannot be recreated from this seed, this requires a continuous backup and is still under development for LND. This information must be kept secret at all times. Write these 24 words down manually on a piece of paper and store it in a safe place. This piece of paper is all an attacker needs to completely empty your wallet! Do not store it on a computer. Do not take a picture with your mobile phone. This information should never be stored anywhere in digital form.

Monitor the LND startup progress until it caught up with the mainnet blockchain (about 515k blocks at the moment). This can take up to 2 hours, then you see a lot of very fast chatter (exit with Ctrl-C ).

`sudo journalctl -f -u lnd`  

Make sure that lncli works by getting some node info:

`lncli getinfo`  

Important: you need to manually unlock the lnd wallet after each restart of the lnd service! But when everything is configured, there’s no need to unlock : Bitseed is running 24h and lnd as well.

## Fund your node

To open channels and start using it, you need to fund it with some bitcoin. For starters, put only on your node what you are willing to lose. Monopoly money. Note that at the time, bitcoins that are stocked in your wallet balance are possibly restored with your 24 words passphrase, but all money in channels is lost if your node gets to be broken, and that you restore your node with another Bitseed or Raspberry Pi for example. You’ll need to close all channels and send your bitcoins from your wallet balance to a regular Bitcoin wallet BEFORE definitely stopping your Bitseed or node, if someday you’re planning to do so.

Generate a new Bitcoin address to receive funds on-chain.

`lncli newaddress np2wkh` 

> "address": "3.........................."

From your regular Bitcoin wallet, send a small amount of bitcoin to this
address.

Check your LND wallet balance

`lncli walletbalance`  

Monitor your transaction on a Blockchain explorer: https://smartbit.com.au

## LND in action

As soon as your funding transaction is mined and confirmed, LND will start to open and maintain channels. This feature is called "Autopilot" and is configured in the "lnd.conf" file. If you would like to maintain your channels manually, you can disable the autopilot.

**Some commands to try.**

List all arguments for the command line interface (cli):

`lncli`  

Get help for a specific argument:

`lncli help [ARGUMENT]`  

Find out some general stats about your node:

`lncli getinfo`  

Connect to a peer (you can find some nodes to connect to here: https://1ml.com/):

`lncli connect [NODE_URI]`

Check the peers you are currently connected to:

`lncli listpeers`  

Open a channel with a peer:

`lncli openchannel [NODE_PUBKEY] [AMOUNT_IN_SATOSHIS] 0`  

Keep in mind that [NODE_URI] includes @IP:PORT at the end, while [NODE_PUBKEY] doesn't.

Check the status of your pending channels:

`lncli pendingchannels`  

Check the status of your active channels:

`lncli listchannels`

Before paying an invoice, you should decode it to check if the amount and other infos are correct:

`lncli decodepayreq [INVOICE]`

Pay an invoice:

`lncli payinvoice [INVOICE]`

Check the payments that you sent:

`lncli listpayments`

Create an invoice:

`lncli addinvoice [AMOUNT_IN_SATOSHIS]`

List all invoices:

`lncli listinvoices`  

To close a channel, you need the following two arguments that can be determined with listchannels and are listed as
"channelpoint": 

> FUNDING_TXID : OUTPUT_INDEX

`lncli listchannels`  

`lncli closechannel [FUNDING_TXID] [OUTPUT_INDEX]`

To force close a channel (if your peer is offline or not cooperative), use:

`lncli closechannel --force [FUNDING_TXID] [OUTPUT_INDEX]`  

See LND API reference for additional information.
