# Silent Dragon Lite IN THE RAW (SDL-RAW)

This documentation is how you setup Silent Dragon Lite to use a Hush blockchain stored on your local computer, or what I like to call SDL in the RAW. This is advanced and most user will never need to do this. These instructions are written with the assumption that you do not normally run a web server on your local computer and have some basic understanding. If you are more familiar than you can adjust these instructions to your needs.

**NOTE: YOU WOULD NEVER DO THIS ON AN ACTUAL SERVER. IT WOULD BE INSECURE, BUT HERE IT IS RUNNING LOCALLY so it's alright.**

## SDL-RAW Setup

### Setup hushd

We need to install the hush daemon (hushd) and then completely download the Hush blockchain.

1. Setup and get your hushd functional on [Linux](../hushd-desktop-linux.md), [Windows](../hushd-desktop-windows.md), or Mac.

1. Start it up and it will take some time to download the whole blockchain. You can check your download against the latest block height on the [Hush explorer](https://explorer.hush.is/).

### Setup lightwalletd

Once that is done downloading, we setup [Hush lightwalletd](https://git.hush.is/hush/lightwalletd):

1. Clone it ```git clone https://git.hush.is/hush/lightwalletd```

1. Then run
```shell script
cd lightwalletd
sudo go run cmd/server/main.go -bind-addr 127.0.0.1:9067 -conf-file ~/.komodo/HUSH3/HUSH3.conf -no-tls
```

### Download and install SDL

**Make sure you make a paper backup of your seed phrases!**

- For Binary releases, [the download link is here](https://git.hush.is/hush/SilentDragonLite/releases)
- For source code so you can compile your own, [the link is here](https://git.hush.is/hush/SilentDragonLite/src/branch/master/README.md)

### Configure SDL

1. Open SDL, let it do it's thing (whether it connects successfully or not), and then exit it when it's done. This allows SDL to generate a local config file.

1. Locate your SDL config file. On Linux it's usually in ~/.config/Hush/SilentDragonLite.conf. I'm not sure where they reside on Windows or Mac.

1. In the [connection] section, make sure the following is there & we only want one server= line in that section:
```shell script
[connection]
server=http://127.0.0.1:9067
```

### Start SDL 

This should now be working! Yay! When you are done then shut down all the servers...

## For Support

- If it is having trouble connecting, then please go into the [Hush Tech Support Telegram channel](https://t.me/hush8support) but be aware that most support will not be aware of this configuration, so you might have to ping me in there @jahway603 :)

