# Setup your Hush Innosilicon Miner

This documentation is how you setup an Innosilicon A9 Miner and start mining Hush. In this example, we are using [Luckpool](https://luckpool.net), so change if needed.

## Where to begin

1. Connect your ASIC to power and internet connection.

1. Determine what the IP address is on your Innosilicon ASIC.

1. Open it and login in via your web browser.

1. In the Web UI Menu, click on Settings and then click on Pools.

<img height=100% width=100% src="images/miner-pool-ui.png">

1. Setup your pool as follows and ***Note your required port*** from [this list](https://luckpool.net/hush/connect.html) and we're using port 3761 below as this ASIC is 50K/sol. We're setting up different geographical mining pools here so that the miner can failover from Pool 1 to Pool 2 and from Pool 2 to Pool3. Again, change to your needs but these are for Innosilicon A9 mining Hush.

Under Pool 1
	- Change URL to "stratum+tcp://na.luckpool.net:3761"
	- Change Worker to your z-address
	- Change Password to X

Under Pool 2
	- Change URL to "stratum+tcp://eu.luckpool.net:3761"
	- Change Worker to your z-address
	- Change Password to X

Under Pool 3
	- Change URL to "stratum+tcp://ap.luckpool.net:3761"
	- Change Worker to your z-address
	- Change Password to X

1. Then click on Update Pools at the bottom of the Mining Pools Settings webpage.

## Mining support

The Hush Team does not operate or support any mining pools. If you need support of your mining pool, then please seek your mining pool support.

