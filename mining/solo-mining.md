# Setup your Solo Miner for Hush

<img height=100% width=100% src="../images/hushsolo.jpg">

This documentation is how to start solo mining Hush with an Innosilicon A9 (ASIC) Miner and be your own "Hush Solo".

## Overview

For this setup we will have our computer running hushd and we will have our ASIC Miner connecting to it. In this write-up we are using a Linux computer. This might work on Windows or Mac, but that is beyond the scope of this document. If you do successfully test with these surveillance operating systems, then feel free to do a pull request to add your input in here.

### Pre-setup

1. Make sure you know the IP address of your ASIC miner. If you're not sure how to figure that out then do a [Startpage search](https://startpage.com) to learn how. It is outside the scope of this document.

1. Make sure you know your desktop computer's IP address.

1. Follow compilation or installation of the [Hush daemon](https://git.hush.is/hush/hush3/) onto your desktop computer, which is outside of the scope of this document. 

### Desktop computer setup

First we need to setup the Hush configuration on our computer.

1. Open your HUSH3.conf, which should be located in your home directory at ~/.komodo/HUSH3/HUSH3.conf.

1. Make sure you add the IP address of your ASIC miner into the conf file. The following is an example using 192.168.33.66 as the ASIC miner's IP:
	```
	rpcuser=user-choose-your-own
	rpcpassword=PASSWORD0123456-make-it-custom
	rpcport=18031
	server=1
	txindex=1
	rpcworkqueue=256
	rpcallowip=127.0.0.1
	rpcallowip=192.168.33.66
	rpcbind=127.0.0.1
	```

	Change the ```rpcuser``` and ```rpcpassword``` above to something unique as it will be used later.
	Note: If you had more than 1 ASIC, then each one would get it's own rpcallowip line item.

1. Then start ```hushd``` at the command line. It needs to be on your desktop computer with the Hush blockchain completely downloaded before you continue. You can check its status with the following command ```hush-cli getinfo | grep synced``` and you want the value to be true before continuing.

### Setup a NOMP

I didn't know what NOMP meant, so I looked it up and it stands for Node Open Mining Protocol. This software will let our ASIC miner connect to the hushd running on our computer, so here we go...

1. We'll use [KNOMP](https://github.com/webworker01/knomp), which is a NOMP based stratum for Komodo based Equihash coins like Hush.

1. We git clone it and change to the ```knomp``` directory.
	```
	mkdir ~/hushsolo
	cd ~/hushsolo
	git clone https://github.com/webworker01/knomp
	cd knomp
	```

1. Then copy an example config file to be used by us.
	```
	cp config_example.json config.json
	```

1. Then edit config.json and save it.
	```
	only things changed from default were 
	"forks": "auto"
	"stratumHost": "stratum.hush.puppy"
	```

1. Next we create a "coin" config for Hush. Change to the ```coins``` directory, create "hush.json", and configure as follows:
	```
	{
		"name": "hush",
		"symbol": "hush",
		"algorithm": "equihash",
		"peerMagic": "f9eee48d",
		"txfee": 0.0001,
		"privateChain": true,
		"burnFees": true,
		"sapling": true
	}
	```

1. Then copy the poolconfig template into the ```knomp\pool_configs``` directory 
	```
	cd ~\hushsolo\knomp
	cp poolconfigs.template pool_configs/hushsolo.json
	cd pool_configs
	```

1. We need some t-addresses (yes, transparent garbage) for the mining software, so generate some new t-addresses with this command:
	```
	hush-cli getnewaddress
	```

1. Now edit hushsolo.json and configure it as follows. 

	** IMPORTANT: the zAddress and tAddress below need to come from the same wallet**

	```
	{
	"enabled": true,
	"coin": "hush.json",
	"address": "FIRST-newly-generated-t-address-you-own",
	"zAddress": "the-z-address-you-want-to-mine-to",
	"tAddress": "SAME-AS-address-above-as-not-sure-if-need-to-be-different-or-not",
	"walletInterval": 1,
	"rewardRecipients": {},
  	"tlsOptions": { "enabled": false },
	"paymentProcessing": { "enabled": false, "daemon": false },
	"trackShares": { "disable": true },
	"ports": {
    	"3333": {
        	"tls":false,
        	"diff": 0.05,
        	"varDiff": {
            	"minDiff": 0.04,
            	"maxDiff": 16,
            	"targetTime": 15,
            	"retargetTime": 60,
            	"variancePercent": 30
        	}
    	}
	},
	"daemons": [{
    	"host": "127.0.0.1",
    	"port": 18031,
    	"user": "rpcuser value from HUSH3.conf",
    	"password": "rpcpassword value from HUSH3.conf"
	}],
	
	"p2p": {
        	"enabled": true,
        	"host": "127.0.0.1",
        	"port": 19333,
        	"disableTransactions": true
	},
	"mposMode": { "enabled": false }
	}
	```

1. Now change back to the knomp directory.
	```
	cd ~\hushsolo\knomp
	```

1. We run ```docker-compose up``` at the command line in the KNOMP directory once we configure as above.

1. Double check by opening 127.0.0.1:8080 in your web browser to check.

### ASIC Miner setup

1. Open the web UI and login in via your web browser.

1. In the Web UI Menu, click on Settings and then click on Pools.

<img height=100% width=100% src="../images/miner-pool-ui.png">

### Configure Solo Mining

1. Setup your "pool" as follows with this example using 192.168.33.101 as the desktop computer's IP address.

	- Under Pool 1
		- Change URL to "http://stratum+192.168.33.101:3333"
		- Change Worker to a z-address from your Hush wallet. It's a good idea to create and use new addresses every so often for better privacy.
		- Change Password to whatever you want, some just use "X"

	- Under Pool 2 & Pool 3 we leave as-is

1. Then click on Update Pools at the bottom of the Mining Pools Settings webpage.

1. Success as you hear your miner spin up! Check the Miner Status in the web UI to verify that it is actually mining.

<img height=100% width=100% src="../images/knomp-hush-miner.png">

1. Also check the status of 192.168.33.101:8080 in your web browser and click around to see that it is working!

## To-Do Items to Clarify

* Not sure if the address & tAddress are the same t-address in the pool_config file.
* Not sure if the ASIC miner "leaving" and "re-joining" the pool constantly is normal or a setting that needs to be changed.
* Have to test this further...

## Mining support

Join us in our Hush mining channel on Telegram, which can [be found here](https://t.me/minersgonnamine). Most miners are not rocking "Hush Solo" style, so make sure you mention that too.

##### Credits

Hush puppies like giving credit when it's due,

1. [This resource](https://github.com/zone117x/node-open-mining-portal) was a great help with writing this as it had in-depth explanation of each option in all of the config files we configured. If you need any other info on certain options, then take a look thru their write-up.

1. [This repository](https://github.com/z-classic/z-nomp/tree/master/pool_configs) is an old Z-NOMP with pool configurations, so these were helpful to see other options, like those special _comments.
