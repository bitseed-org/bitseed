[OpenBazaar](https://openbazaar.org) is a decentralized ecommerce application that uses a [distributed hash table](https://en.wikipedia.org/wiki/Distributed_hash_table) to find peer nodes, [Blockstack](https://blockstack.org) to secure usernames, and [Bitcoin](https://bitcoin.org) for payments. Following the instructions in this post will install the OpenBazaar server software on a Bitseed personal server and allow OpenBazaar clients to connect to the server to create and update user profiles and stores. User profiles and stores created this way will be self-hosted on the Bitseed personal server.

**Note:** the connection between the OpenBazaar client and the remote server is currently **unencrypted** and **unauthenticated**. This means that **usernames and passwords sent to the server will be sent unencrypted**. It is therefore recommended that users only open their OpenBazaar desktop client and access their server when they are **on the same local network** that the server is hosted on. This network should be **secured against untrusted connections** to prevent unauthorized access to the server.  

Open an SSH terminal on your desktop and SSH into Bitseed (instructions [here](https://bitseed.org/bitseed-bitcoin-edition-quickstart-guide/)) then enter the following commands:  

`sudo apt-get update`   

`sudo apt-get install build-essential libssl-dev libffi-dev python-dev openssl python-pip libzmq3-dev`  

In your desktop web browser, go to:  

https://github.com/jedisct1/libsodium/releases  

Under the latest release at the top of the page, find the file *Source code (zip)*.  

Right click on the link to this file and click *copy link address*.  

In your SSH terminal window, type:  

`wget`  

Then paste the link you copied in the previous step. The result should look like this, but with the link address you just copied:  

`wget https://github.com/jedisct1/libsodium/archive/1.0.10.zip`  

Press *Enter*.

Copy the file name after the last forward-slash in the link copied above. In this example, the file name is *1.0.10.zip*.  

In your SSH terminal window, type:  

`unzip`  

Then paste the file name copied in the previous step. The result should look like this, but with the file name you just copied:  

`unzip 1.0.10.zip`  

Press *Enter*.

After the file finishes unzipping, enter the following commands:  

`cd libsodium-1.0.10`  

`./autogen.sh`  

`./configure`  

`make`  

Wait for libsodium to finish building. This step will take about twenty minutes. Then enter the following commands:  

`sudo make install`  

`cd ..`  

`sudo pip install cryptography`  

`sudo apt-get install autoconf pkg-config libtool`  

`git clone https://github.com/zeromq/libzmq`  

`cd libzmq`  

`./autogen.sh && ./configure && make -j 4`  

`sudo make install`  

`sudo ldconfig`  

`cd ..`  

`sudo apt-get install npm`  

`curl -sL https://deb.nodesource.com/setup_0.12 | sudo bash -`  

`sudo apt-get install -y nodejs`  

`npm install electron-prebuilt`  

In your desktop web browser, go to:  

https://github.com/OpenBazaar/OpenBazaar-Server/releases  

Under the latest release at the top of the page, find the file *Source code (zip)*.  

Right click on the link to this file and click *copy link address*.  

In your SSH terminal window, type:  

`wget`  

Then paste the link you copied in the previous step. The result should look like this, but with the link address you just copied:  

`wget https://github.com/OpenBazaar/OpenBazaar-Server/archive/v0.1.8.zip`  

Press *Enter*.

Copy the file name after the last forward-slash in the link copied above. In this example, the file name is *v0.1.8.zip*.  

`In your SSH terminal window, type:`  

`unzip`  

Then paste the file name copied in the previous step. The result should look this, but with the file name you just copied:  

`unzip v0.1.8.zip`  

Press *Enter*.

Copy just the numbers in the version number of the zip file you just unzipped e.g. *0.1.8*  

Then type:

`cd OpenBazaar-Server-`  

And paste the version number immediately after the dash. The command will look like this, but with the server version you just copied:  

`cd OpenBazaar-Server-0.1.8`  

Press *Enter*, then enter the following commands:  

`sudo pip install -r requirements.txt`  

`python openbazaard.py start`  

Wait for the GUID to be generated. After about ten minutes, press *ctrl+C* to stop the server. Then copy+paste:  

`nano /home/linaro/OpenBazaar-Server-`  

And paste the version number followed by */ob.cfg*. The whole line will look like this, but with the server version you just copied:  

`nano /home/linaro/OpenBazaar-Server-0.2.0/ob.cfg`  

Press *Enter*. The configuration file will appear.  

Hit the down arrow until you find these lines:  

    #USERNAME=username
    #PASSWORD=password  

Remove the # at the beginning of each of these lines and change the lower case username and password to your own login info for your store. Use a password manager like KeePass ([Windows](http://keepass.info/download.html) or [Linux/OS X](https://www.keepassx.org/downloads)) to save this information.  

Press *ctrl+x* to exit, then press *y*, then press *Enter* to save. Then enter the following command:  

`python openbazaard.py start -d --allowip=0.0.0.0`  

OpenBazaar is now running on Bitseed!  

Go to the desktop you want to use OpenBazaar with and install the OpenBazaar client from https://openbazaar.org.  

Run the OpenBazaar client and go through the setup process. Then in the Menu go to *Settings --> Advanced --> Server Settings* and change the server setting from localhost to the local IP address of the Bitseed node hosting the OpenBazaar server.  

Enter the username and password that you set in the OpenBazaar server configuration file, then save the changes.  

You are now viewing and controlling your own self-hosted OpenBazaar server. Any stores or user profiles that you create with your desktop client will be self-hosted on your Bitseed server and served to other OpenBazaar peers over Bitseed's internet connection. If you want to protect your OpenBazaar server's IP address from peers on the network, follow the instructions in the [Virtual Private Network (VPN) How-To](https://github.com/OpenBazaar/OpenBazaar-Server/wiki/Virtual-Private-Network-(VPN)-How-To) published by the OpenBazaar team. For OpenBazaar support, join the OpenBazaar [Slack community](https://openbazaar-slackin-drwasho.herokuapp.com/) or contact the team by [email](http://project@openbazaar.org).  
