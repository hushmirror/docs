# Running your own hush lite server

This write up will explain how to setup your own light (lite) wallet server to use with [Hush's SilentDragonLite wallet](https://git.hush.is/hush/SilentDragonLite) or [CLI version](https://git.hush.is/hush/silentdragonlite-cli).

### Install & Setup your Linux server or VPS

Install your preferred distro. In this example I am using a VPS running Ubuntu 20.04, so these instructions should be very similar for Debian-based distros. This setup requires a VPS with at least 2 vCPUs and 4GB of RAM to compile and run the software.

##### Pre Steps

1. Install GNU screen

    ```
    $ sudo apt install screen
    ```

1. Install the Go language and NGINX.

    ```
    $ sudo apt install golang nginx
    ```

1. Enable nginx thru your firewall and open port 443 (HTTPS). Look up more info on ufw if needed.

    ```
    $ sudo ufw status
    ```

##### Setup Hushd

1. Setup hushd by [following these instructions](hushd-desktop-linux.md) and make sure to grant the hush user sudo access.

1. Start a screen session and change to user hush with ```sudo -u hush -s```.

1. Run hushd at the command line. You should see a bunch of text scrolling.

1. Then check if the Hush blockchain is downloading by noticing if the blockchain directory is increasing.

    ```
    $ du -h ~/.komodo/HUSH3/blocks/
    ```

1. The blockchain download will take some time, so open another terminal (or screen) and continue to install Lightwalletd.

##### Setup Lightwalletd

1. Then as user hush ```sudo -u hush -s``` download the [Hush Lightwalletd](https://git.hush.is/hush/lightwalletd)

    ```
    $ git clone https://git.hush.is/hush/lightwalletd
    ```

1. Get a TLS certificate. If you running a public-facing server, the easiest way to obtain a certificate is to use a NGINX reverse proxy and get a Let's Encrypt certificate. Since we're using Ubuntu here **I SUGGEST YOU DO NOT USE SNAPD** and just ```sudo apt install certbot``` and then start on [Step 7 of these instructions by the EFF](https://certbot.eff.org/instructions).

1. Create a new section for the NGINX reverse proxy and change port 443 to something else if needed:
   
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

1. Run the lightwalletd frontend with the following:

    ```
    $ sudo go run cmd/server/main.go -bind-addr your_host.net:9067 -conf-file ~/.komodo/HUSH3/HUSH3.conf -no-tls
    ```

    Note: Above we use the "-no-tls" option as we are using NGINX as a reverse proxy and letting it handle the TLS authentication.

    Note: You can configure lightwalletd to handle its own TLS authentication, but you will have to consult the lightwalletd documentation for that setup.

1. You should start seeing the frontend ingest and cache the Hush blocks after ~15 seconds. Success!

##### Point the `silentdragonlite-cli` to this server

1. Ubuntu only has version 1.43.0 or Rustc, so we want to install a newer version. I used the defaults in the script.

    ```
    $ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    $ rustc --version
    rustc 1.47.0 (18bf6b4f0 2020-10-07)
    ```

1. Now to test if it's working with a client by connecting to your server! Substitute your server below:
    
    ```
    $ git clone https://git.hush.is/hush/silentdragonlite-cli
    $ cd silentdragonlite-cli
    $ git checkout origin/dev
    $ cargo build --release
    $ ./target/release/silentdragonlite-cli --server https://lite.hush.is
    ```

1. Success!

