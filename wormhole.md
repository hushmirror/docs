# Setup Hush Wormhole Server

**Developmental warning: I have not been able to get the wormhole to function correctly, but I have been able to get closer towards it working. Please feel free to test and contribute. Thanks.**

In this how-to we cover how to implement your own [Hush wormhole server (HWS)](https://git.hush.is/hush/SilentDragonWormhole). The HWS is used with the [Hush Android companion app](https://git.hush.is/hush/SilentDragonAndroid) to send and receive payments from your Hush desktop wallet (both [full node](https://git.hush.is/hush/SilentDragon) and [lite wallet](https://git.hush.is/hush/SilentDragonLite) are supported).

## Setup Wormhole

### Setup system

Install your preferred distro. In this example I am using a VPS running Ubuntu 20.04, so these instructions should be very similar for Debian-based distros.

### Pre-setup to compile

Before compiling, note that there is a wormhole binary already in "build/libs" so you can use that if your system has any issues with compiling it as others have.

1. Install dependencies:

    ```
    $ sudo apt-get install android-sdk gradle
    ```

1. Your system may have a version of gradle that is too old. Gradle 5.6.1 is known
to work well:

    ```
    $ wget https://services.gradle.org/distributions/gradle-5.6.1-bin.zip
    $ unzip gradle-5.6.1-bin.zip
    $ export PATH=$HOME/gradle-5.6.1-bin:$PATH
    ```

1. Clone and compile:

    ```
    $ git clone https://git.hush.is/hush/SilentDragonWormhole
    $ cd SilentDragonWormhole
    $ gradle build
    ```

1. (Optional step) Setup a separate user account to run the wormhole on.

### Run the wormhole

1. If the build was successful, you can execute the binary and start the server. Note: The default port of the Wormhole is 7070. This can be changed in source and recompiled if needed.

    ```
    $ java -jar build/libs/wormhole-1.0-SNAPSHOT.jar
    ```

1. Test if it is working and see if is 7070 (or other if you changed it) port is listening

    ```
    $ ss -lt
    ```

### Nginx Setup

1. Install Nginx & GNU Screen

    ```
    $ sudo apt install nginx screen -y
    ```

1. Enable nginx thru your firewall. Look up more info on ufw if needed.

	```
	$ sudo ufw status
	```

1. Get a TLS certificate. If you running a public-facing server, the easiest way to obtain a certificate is to use a NGINX reverse proxy and get a Let's Encrypt certificate. Since we're using Ubuntu here **I SUGGEST YOU DO NOT USE SNAPD** and just ```sudo apt install certbot``` and then start on [Step 7 of these instructions by the EFF](https://certbot.eff.org/instructions).

1. Create a new section for the NGINX reverse proxy and change port 443 to something else if needed:
   
    ```
    server {
        listen 443 ssl;
        listen [::]:443 ssl;
    
        server_name your_host.net;

        ssl_certificate /etc/letsencrypt/live/your_host.net/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/your_host.net/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
        
        access_log /var/log/nginx/your_host.net.access.log;
        error_log /var/log/nginx/your_host.net.error.log;

        location / {
            # let's try wormhole stuff
            # redirect to our wormhole running on 7070
            proxy_pass http://localhost:7070;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            # magic needed for WebSocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }
    }
    ```

    You might also need these [20.04 specific instructions to setup your tls certificate with Nginx](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

### Test your wormhole

You will need to do the following steps on your computer and phone to test that your new wormhole server is working.

##### On the Desktop computer

1. Open [SilentDragon](https://git.hush.is/hush/SilentDragon) or [SillentDragonLite](https://git.hush.is/hush/SilentDragonLite) wallet application.

1. At the top, click on Apps and then click on "Connect mobile app".

1. On the "Connect mobile app" screen, check the checkbox labeled "Allow connections over the internet via wormhole".

##### On the phone

1. Open your [SilentDragonAndroid app](https://git.hush.is/hush/SilentDragonAndroid).

1. In the top right hand corner, click on the three dots icon and then click on Settings.

1. On the App Settings screen, you will see "Current Wormhole:" which displays the configured wormhole.

1. Just below this, enter your server's hostname and port and then click on "Set Custom Wormhole" to change the configured wormhole.

##### Use phone with computer

1. On phone app, click "Scan QR Code" and then use it to scan the code on the computer screen.

1. Now they should be paired.
