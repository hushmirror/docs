# Silent Dragon Lite IN THE RAW (SDL-RAW)

This documentation is how you setup Silent Dragon Lite to use a Hush blockchain stored on your local computer, or what I like to call SDL in the RAW. This is advanced and most user will never need to do this. These instructions are written with the assumption that you do not normally run a web server on your local computer and have some basic understanding. If you are more familiar than you can adjust these instructions to your needs.

**NOTE: YOU WOULD NEVER DO THIS ON AN ACTUAL SERVER. IT WOULD BE INSECURE, BUT HERE IT IS RUNNING LOCALLY so it's alright.**

## SDL-RAW Setup

### Setup hushd

We need install the hush daemon (hushd) and the completely download the Hush blockchain.

1. Setup and get your hushd functional on [Linux](hushd-desktop-linux.md), [Windows](hushd-desktop-windows.md), or Mac.

1. Start it up and then it will take some time to download the whole blockchain. You can check your download against the latest block height on the [Hush explorer](https://explorer.hush.is/).

### Setup lightwalletd

1. Clone it ```git clone https://git.hush.is/hush/lightwalletd```

1. Then run

	```
	$ cd lightwalletd
	$ sudo go run cmd/server/main.go -bind-addr 127.0.0.1:9067 -conf-file ~/.komodo/HUSH3/HUSH3.conf -no-tls
	```

### Setup nginx as reverse proxy

Here we are using a very simple nginx.conf where this web server will not run anything else and will only be started when needed.

1. Install nginx. Look up how to do that as it can be different for each operating system.

1. Backup the nginx.conf to a nginx.conf.original before you change it.

1. Start up nginx and open your computer's web browser to http://127.0.0.1 to see that the nginx test page is working, then continue on.

1. Change the nginx.conf to the following:

	```
	worker_processes  1;

	events {
    	worker_connections  1024;
	}

	http {
		include       mime.types;
		default_type  application/octet-stream;
		sendfile        on;
		keepalive_timeout  65;
		
		server {
	    	listen       80 http2;
	        server_name  localhost;
	
	    	location / {
		   		grpc_pass grpc://localhost:9067;
	    	}
	
	        error_page   500 502 503 504  /50x.html;
		    location = /50x.html {
	    	    root   /usr/share/nginx/html;
	    	}
		}
	}
	```

1. Restart nginx.

### Download and install SDL

**Make sure you make a paper backup of your seed phrases!**

- For Binary releases, [the download link is here](https://git.hush.is/hush/SilentDragonLite/releases)
- For source code so you can compile your own, [the link is here](https://git.hush.is/hush/SilentDragonLite/src/branch/master/README.md)
- For legacy AppImage releases, [the download link is here](https://github.com/MyHush/SilentDragonLite/releases)

### Configure SDL

Configure it before opening it.

1. Locate your SDL config file. On Linux it's usually in ~/.config/Hush/SilentDragonLite.conf

1. Now make sure the following is in there:

	```
	[connection]
	server=http://127.0.0.1:9067
	```

### Start SDL 

This should now be working! Yay! When you are done then shut down all the servers...

## For Support

- If it is having trouble connecting, then please go into the [Hush Tech Support Telegram channel](https://t.me/hush8support) but be aware that most support will not be aware of this configuration, so you might have to ping me in there @jahway603 :)

