# Setup Hush full node with hushd on Windows

In this example, we are using Windows 10 (64 bit).

## Setup hushd

### Hush binary or compile yourself?

On Windows I suggest you use the release binary unless you know what you're doing regarding code and compilers.

1) Download the [release binary here](https://git.hush.is/hush/hush3/releases) and unextract the zip file. (Note: for 3.6.0 please choose the larger file)

1) The data (blockchain, configuration, etc.) will be stored in 'C:\Users\<YOUR_USERNAME>/AppData/Roaming/Komodo/HUSH3' (on Windows) by default.

1) Open the configuration file in your favorite text editor. I would change the values below with CHANGETHIS appended and you can change the rpcport if you'd like:

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

    addnode=199.247.28.148
    ```

### Now we can start hushd daemon 

1) Open the directory you extracted to in a command prompt window. I use cmd on Windows, but some user PowerShell.

1) Run hushd.bat and a **Windows Security Alert should immediately pop-up** regarding "komodod".

1) We want this, so we check the two checkboxes and Accept. This will take some time to sync and you will see a bunch go across the screen. Congratulations as now you have Hushd running on your Windows system!

