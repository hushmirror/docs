# Running your own hush lite server
![You have to call me dragon](images/dragon-stepbrothers.gif)

This write up will explain how to setup your own light (lite) wallet server to use with [Hush's SilentDragonLite wallet](https://git.hush.is/hush/SilentDragonLite) or [CLI version](https://git.hush.is/hush/silentdragonlite-cli).

### Install & Setup your Linux server or VPS

Install your preferred distro. In this example I am using a VPS running Ubuntu 20.04, so these instructions should be very similar for any Debian-based distro. This setup requires a VPS with at least 2 vCPUs and 4GB of RAM to compile and run the software.

##### Pre Steps

1. Install GNU screen
    ```shell script
    sudo apt install screen
    ```

1. Install the Go language and NGINX.
    ```shell script
    sudo apt install golang nginx
    ```

1. Enable nginx thru your firewall and open port 443 (HTTPS). Look up more info on ufw if needed.
    ```shell script
    ufw help
    sudo ufw status
    sudo ufw allow 443
    ```

##### Setup Hushd

1. Setup hushd by [following these instructions](hushd-desktop-linux.md) and make sure to grant the hush user sudo access.

1. Start a screen session and change to user hush with ```sudo -u hush -s```. If you are not familiar with GNU screen or need a refresher, then [check out this link](https://linuxize.com/post/how-to-use-linux-screen/).

1. Open hushd port in the firewall.
    ```shell script
    sudo ufw allow 18030
    sudo ufw status
    ```

1. Run hushd at the command line. You should see a bunch of text scrolling.

1. Then check if the Hush blockchain is downloading by noticing if the blockchain directory is increasing.
    ```shell script
    du -h ~/.komodo/HUSH3/blocks/
    ```

1. The blockchain download will take some time, so feel free to take a break and wait or open another terminal (or GNU screen) and continue to install Hush lightwalletd.

##### Setup Lightwalletd

1. Then as user hush ```sudo -u hush -s``` download the [Hush Lightwalletd](https://git.hush.is/hush/lightwalletd)
    ```shell script
    git clone https://git.hush.is/hush/lightwalletd
    ```

1. Install these packages for certbot
    ```shell script
    sudo apt install certbot python3-certbot-nginx
    ```

1. Get a TLS certificate. If you running a public-facing server, the easiest way to obtain a certificate is to use a NGINX reverse proxy and get a Let's Encrypt certificate. Since we're using Ubuntu here **I SUGGEST YOU DO NOT USE SNAPD** and just ```sudo apt install certbot``` and then start on [Step 7 of these instructions by the EFF](https://certbot.eff.org/instructions) and most users would run the following command and follow the prompts:
    ```shell script
    sudo certbot --nginx
    ```

1. Open up your web browser and see that the https template site is working before moving forward. It will appear with the lock icon in your web browser and you can click on it and see that it is valid certificate in your web browser.

1. Make a backup of the nginx's default file located under /etc/nginx/sites-available/default.

1. Modify the above default file to contain only the following (if not using 443, then change that to which port you are using too):
   
    ```
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
    
        server_name your_host.net;

        ssl_certificate /etc/letsencrypt/live/your_host.net/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/your_host.net/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
        
        location / {
            # Replace your_host.net:9067 with the address and port of your gRPC server if using a custom port
            grpc_pass grpc://your_host.net:9067;
        }
    }
    ```

    You might also need these [20.04 specific instructions to setup your tls certificate with Nginx](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

1. Restart nginx to enable the new configuration.
    ```shell script
    sudo systemctl restart nginx.service
    ```

1. Open lightwalletd port in the firewall.
    ```shell script
    sudo ufw allow 9067
    sudo ufw status
    ```

1. Run the lightwalletd frontend with the following and your server's hostname:
    ```shell script
    go run cmd/server/main.go -bind-addr your_host.net:9067 -conf-file /home/YOUR_USERNAME_Running_Hushd/.komodo/HUSH3/HUSH3.conf -no-tls
    ```

    Note: Above we use the "-no-tls" option as we are using NGINX as a reverse proxy and letting it handle the TLS authentication for us.

    Note: You can configure lightwalletd to handle its own TLS authentication, but you will have to consult the [lightwalletd documentation](https://git.hush.is/hush/lightwalletd) for that setup.

1. It will first begin downloading golang dependencies. After that is complete then you should start seeing the frontend ingest and cache the Hush blocks after ~15 seconds. Success!

##### Testing your SDL server 

###### Option 1: Point the SilentDragonLite GUI Desktop wallet to this server

1. Download and install the [SilentDragonLite (SDL) wallet](sdl.md).

1. After opening the SDL wallet, go into the Edit -> Settings toolbar.

1. Enter your https://your_host.net into the Lightwallet Server field.

1. Close SDL and then re-open it.

1. Success!

###### Option 2: Point the command line `silentdragonlite-cli` to this server

1. Ubuntu only has version 1.43.0 or Rustc, so we want to install a newer version. I used the defaults in the script.
    ```shell script
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    rustc --version
    rustc 1.47.0 (18bf6b4f0 2020-10-07)
    ```

1. Now to test if it's working with a client by connecting to your server! Substitute your server below:
    ```shell script
    git clone https://git.hush.is/hush/silentdragonlite-cli
    cd silentdragonlite-cli
    cargo build --release
    ./target/release/silentdragonlite-cli --server https://lite.hush.is
    ```

1. Success!

