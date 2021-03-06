# Dilmapunk Installation Procedure

This guide will assist you with installing Dilmapunk. This document assumes you are running Ubuntu 12.04 LTS, adjustments may need to be made for other OSes.

If you don't understand how to use this document, **Dilmapunk is not for you**. Dilmapunk requires a commanding understanding of UNIX system administration to be run safely.

## Install Prerequisites

Update your repository data and packages if this is a fresh install of Ubuntu:

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git autoconf libtool ntp build-essential
```

It is recommended you enable [unattended security updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates) to help protect your system from security issues:

```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

## Install NodeJS

The latest information on installing NodeJS for your platform is [available here](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager), this is the current procedure for Ubuntu:

```
sudo apt-get install python-software-properties python g++ make
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

## Install and Configure Redis

Redis is used to store your wallet data.

```
sudo apt-get install redis-server
```

Now you will need to edit `/etc/redis/redis.conf` to be more data persistent:

Change `appendonly no` to `appendonly yes`.
Change `appendfsync everysec` to `appendfsync always`.

Restart redis: `sudo service redis-server restart`.

## Install and Configure dilmacoind

Currently Dilmapunk depends on a custom build of dilmacoind using [this patch](https://github.com/dayreiner/dilmacoin-watchonly).

Install prerequisites:
```
sudo apt-get install libdb4.8++ libdb4.8++-dev pkg-config libprotobuf-dev \
libminiupnpc8 minissdpd libboost-all-dev ccache libssl-dev
```

Checkout and make [dilmacoin-watchonly](https://github.com/dayreiner/dilmacoin-watchonly):
```
git clone https://github.com/dayreiner/dilmacoin-watchonly.git
cd dilmacoin-watchonly/src
make -f makefile.unix USE_UPNP=0 USE_QRCODE=1 USE_IPV6=1
strip dilmacoind
```

Create a user for dilmacoind and move the binary to where it can access it:
```
adduser dilmacoin
mkdir -p ~dilmacoin/bin
mv dilmacoind /usr/home/dilmacoin/bin
```

Now you need to configure dilmacoind:

```
su - dilmacoin
mkdir -p ~/.dilmacoin
vi ~/.dilmacoin/dilmacoin.conf
```

And add the following information (set the `rpcuser` and `rpcpassword` to something else):

```
rpcuser=NEWUSERNAME
rpcpassword=NEWPASSWORD
rpcallowip=127.0.0.1
rpcport=21056
listen=1
daemon=1
server=1
rpcallowip=127.0.0.1 
rpcconnect=127.0.0.1
rpcallowip=127.0.0.1
addnode=54.194.99.126
addnode=54.84.229.24

```
**If your dilmacoind crashes due to memory consumption**, try limiting your connections by adding `maxconnections=10`. Try further adjusting to 3 if you are still having issues.

Consider adding a startup script for dilmacoind to either init.d or via upstart.

Start dilmacoind as the dilmacoin user:

```
su - dilmacoin
dilmacoind &
```

**dilmacoind may take up to an hour (or more, depending on growth) to download the blockchain.** Dilmapunk will not be able to function properly until this has occurred. Please be patient.

If you want something to monitor dilmacoind to ensure it stays running and start it on system restart, take a look at [Monit](http://mmonit.com/monit/).

## Install and Configure Dilmapunk

Go to your user's home directory (`cd ~`), clone the repository and install nodejs dependencies:

```
git clone https://github.com/Dilmacoin/dilmapunk.git
cd dilmapunk
npm install
```

Now you will need to create and configure your config.json file, one for the main folder and one in `public`. From the `dilmapunk` directory:

```
cp config.template.json config.json
```

Edit the file to connect to `dilmacoind` on port 11056 using the user/password you set when configuring dilmacoind:

```
{
  "bitcoind": "http://NEWUSERNAME:NEWPASSWORD@127.0.0.1:11056",
  "pricesUrl": "http://localhost:8080/rates.json",
  "httpPort": 8080
}
```

For SSL:

```
{
  "bitcoind": "http://NEWUSERNAME:NEWPASSWORD@127.0.0.1:11056",
  "pricesUrl": "http://localhost:8080/rates.json",
  "httpPort": 8080,
  "httpsPort": 8086,
  "sslKey": "./Dilmapunk.key",
  "sslCert": "./Dilmapunk.crt"
}
```
Alternately, you can use Nginx as your SSL endpoint and proxy requests over to Dilmapunk instead of opening your node install directly to the world.

Now copy the client application's config:

```
cp public/config.template.json public/config.json
```

And change `network` to `prod` instead of `testnet` to use Dilmapunk in production mode.

## Start Dilmapunk

You can start Dilmapunk from the command line:

```
node start.js
```

Try to connect by going to http://YOURADDRESS.COM:8080  (If you're using the SSL config then try  http://YOURADDRESS.COM:8085. OR https://YOURADDRESS.COM:8086) If it loads, then you should be ready to use Dilmapunk!


## Backing up Database

Redis maintains a file called `/var/lib/redis/dump.rdb`, which is a backup of your Redis database. It is safe to copy this file while Redis is running. **It is strongly recommended that you backup this file frequently.** You can also setup a Redis slave to listen to master in real time. Ideally you should do both!

## Extra Steps for Contributors

If you want to contribute code to this project, you will need to use Grunt. Grunt is a task-runner that presently handles minifying and uglifying Dilmapunk's CSS and JS resources.  Grunt is installed by the `npm install` you ran from the Dilmapunk directory.

Running `./node_modules/grunt-cli/bin/grunt` in your Dilmapunk directory will minify and uglify everything, and running `./node_modules/grunt-cli/bin/grunt watch` will automatically uglify your JS files when they change.

You can also install grunt system-wide with `sudo npm install -g grunt-cli`.
