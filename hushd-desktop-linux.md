# Setup Hush full node with hushd on Linux

In this example, we are using Ubuntu 18.04 (64 bits). This will be different on Mac and Windows, so refer to those OS's docs.

## Setup hushd

### Setup hush user account

1) Log in as user account with sudo access and add a user 'hush' under which the daemon (hushd) will run :

    ```
    $ sudo useradd -r -m -s /bin/bash -d /home/hush hush
    ```

2) Assign a password to the 'hush' user and add to sudo group:

    ```
    $ sudo passwd hush
    $ sudo adduser hush sudo
    ```

3) Switch to user 'hush'
su - hush

4) Update your system

    ```
    $ sudo apt-get update
    $ sudo apt-get upgrade -y
    ```

### Hush binary or compile yourself?

The next step is up to you. I personally like to compile from source and recommend trying that if you experience any issues.

#### binary OR...

If you just want to install an "exe" file, run it, and go, then I would recommend trying the binary install.

On Ubuntu 18.04/20.04 (Debian?), try this:
$ wget https://github.com/MyHush/hush3/releases/download/v3.5.2/hush-3.5.2-amd64.deb
$ sudo dpkg -i hush-3.5.2-amd64.deb

#### compile yourself

The choice is up to you, but if the binary does not work then try compiling it yourself.

For Ubuntu 18.04 or 20.04, use the following:

    ```
    # Install build depedencies
    $ sudo apt-get install build-essential pkg-config libc6-dev m4 g++-multilib autoconf libtool ncurses-dev unzip git python zlib1g-dev wget bsdmainutils automake curl unzip nano libsodium-dev

    # Pull
    $ git clone https://git.hush.is/hush/hush3.git
    $ cd hush3

    # Build
    $./build.sh -j$(nproc)			replace $(nproc) by the number of processor (example : 4)
    ```

For Arch Linux, there are [hush3](https://aur.archlinux.org/packages/hush3/) and [hush3-bin](https://aur.archlinux.org/packages/hush3-bin/) AUR packages available. If you'd rather compile yourself, then use the following:

    ```
    # Install build depedencies
    $ sudo pacman -S libsodium lib32-zlib unzip wget git python rust curl
    
    # Pull
    $ git clone https://git.hush.is/hush/hush3.git
    $ cd hush3
    
    # Build
    $./build.sh -j$(nproc)			replace $(nproc) by the number of processor (example : 4)
    ```

### Setup HUSH3.conf

1) The data (blockchain, configuration, etc.) will be stored in '/home/.komodo/HUSH3' (on Linux) by default.

    ```
    $ mkdir -p ~/.komodo/HUSH3
    ```
1) Open the configuration file in your favorite text editor (nano, vim, etc). I would change the values below with CHANGETHIS appended and you can change the rpcport if you'd like:

    ```
    rpcuser=user-CHANGETHIS
    rpcpassword=pass-CHANGETHIS
    rpcport=18031
    server=1
    daemon=1
    txindex=1
    rpcworkqueue=256
    rpcallowip=127.0.0.1
    rpcbind=127.0.0.1

    addressindex=1
    spentindex=1
    timestampindex=1

    showmetrics=1

    addnode=explorer.myhush.org
    addnode=stilgar.leto.net
    addnode=dnsseed.bleuzero.com
    addnode=dnsseed.hush.quebec
    ```

### Now we can start hushd daemon 

Run this and it will take some time to sync with the network.

    ```
    $ which hushd   # then run from where it's installed
    $ hushd
    ```

