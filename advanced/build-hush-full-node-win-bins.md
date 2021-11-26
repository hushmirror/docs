# Cross-compile & Package Hush Full Node Binaries for Windows

These instructions are how to cross-compile and package fresh Hush Windows executables from Ubuntu 18.04. In addition to creating fresh win bins, this tutorial will also shine light on how to use a virtual machine with docker engine. This process also aids in keeping your local machine free of unwanted packages and on exit of the interactive docker terminal, the image is destroyed.

## Requirements
- [Docker Engine](https://docs.docker.com/engine/install)

### Launch a docker image in interactive mode, rm on exit of container and bind tmp directory to actual tmp
```
sudo docker run -it --rm -v /tmp:/tmp ubuntu:bionic
```

### Verify building on correct OS version
```
cat /etc/*-release
```
### Install dependencies
```
apt update && apt-get install build-essential pkg-config libc6-dev m4 g++-multilib \
      autoconf libtool ncurses-dev unzip git python zlib1g-dev wget \
      bsdmainutils automake curl unzip nano libsodium-dev g++-mingw-w64-x86-64 git zip
```
 
### Clone Hush source
```   
git clone https://git.hush.is/hush/hush3.git && cd hush3
```

### Prepare directory for zipping
```
mkdir mkdir hush-vX.X.X-win
```

### Compile fresh Hush Windows binaries
```
./zcutil/build-win.sh
```

### Copy binaries to hush-vX.X.X-win folder
```
cd src/ && cp hushd.exe hush-tx.exe hush-cli.exe ../hush-vX.X.X-win/ && cd ..
```

### Copy sapling params to hush-vX.X.X-win folder
```
cp sapling-output.params sapling-spend.params hush-vX.X.X-win/
```

### Copy asmap to hush-vX.X.X-win folder
```
cp asmap.dat hush-vX.X.X-win/
```


### Create README.txt
```
nano hush-vX.X.X-win/README.txt
```

#### Edit / Copy & Paste 

```
                     ___====-_  _-====___
               _--~~~#####// '  ` \\#####~~~--_
             -~##########// (    ) \\##########~-_
           -############//  |\^^/|  \\############-
         _~############//   (O||O)   \\############~_
        ~#############((     \\//     ))#############~
       -###############\\    (oo)    //###############-
      -#################\\  / `' \  //#################-
     -###################\\/  ()  \//###################-
    _#/|##########/\######(  (())  )######/\##########|\#_
    |/ |#/\#/\#/\/  \#/\##|  \()/  |##/\#/  \/\#/\#/\#| \|
    `  |/  V  V  `   V  )||  |()|  ||(  V   '  V /\  \|  '
       `   `  `      `  / |  |()|  | \  '      '<||>  '
                       (  |  |()|  |  )\        /|/
                      __\ |__|()|__| /__\______/|/
                     (vvv(vvvv)(vvvv)vvv)______|/


HUSH vX.X.X Windows Binaries

Download new binaries from: https://git.hush.is/hush/hush3/releases

Thanks to all the people who made this possible, including:
	
	-DukeLeto
	-oDinZu
	-Jahway603
	-Onryo
	-Fekt0r
	-Hush Community

INSTRUCTIONS:

0.) Verify checksum matches downloaded file with: certutil -hashfile .\hush-vX.X.X-win.zip SHA256
1.) Extract archive: right-click extract here
2.) Open up powershell or command prompt by pressing WIN+R, type powershell or cmd.
3.) Launch hushd full node by typing: ./hushd.exe or hushd if in command prompt; you will see the hush full node begin downloading the hush blockchain.
4.) In another powershell or command prompt window, you can use hushd -help and hush-cli help for managing your wallet/full-node with command line interface (CLI). As an example, to create a new z-addr, type: hush-cli z_getnewaddress OR .\hush-cli.exe getalldata.
5.) Press WIN+R, type %APPDATA% and open up your Hush folder, then see your debug.txt file and your HUSH3.conf file. The user/pass is anonymously generated, you can change these if you desire so.

Notes: 

- If you are running out of memory syncing blockchain and hushd is silently exiting without any errors. Run hushd without using other resources. 

- Always close hushd with hush-cli stop; an incorrect shutdown may result in a corrupt wallet. 

Community & Socials: 

See our new YouTube channel for more info: https://hush.is/yt
And also https://videos.hush.is
Explorer: https://explorer.hush.is

Join Telegram main channel for announcements: https://hush.is/tg
Join Telegram for official support: https://hush.is/tg_support
```

#### Save README.txt
```
CTRL+X
SHIFT+Y
ENTER
```

### Compress hush-vX.X.X-win folder and all its files
```
zip -r hush-vX.X.X-win.zip hush-vX.X.X-win/
```

### Create SHA256 Checksum for integrity
```
sha256sum hush-vX.X.X-win.zip 
```
I.E. b558c52c4bdbf3290f28b2a7beaa3cd1a93aa6b110ef2db59c2ad17faff60314  hush-3.9.0-win.zip

### Copy hush-vX.X.X-win.zip to actual machine directory /tmp/
```
cp -v hush-vX.X.X-win.zip
```

### Test fresh Hush Windows binaries
*You will need a Windows 10 OS to test these binaries*
**Follow the README.txt**

### Upload ZIP to Hush Gitea via [Releases](https://git.hush.is/hush/hush3/releases page)
```
cp /tmp/hush-vX.X.X-win.zip /home/username/Downloads
```
#### Add checksum to Gitea releases page
```
b558c52c4bdbf3290f28b2a7beaa3cd1a93aa6b110ef2db59c2ad17faff60314  hush-3.9.0-win.zip
```

### Exit docker container
```
exit
```

### Congratulations :party 

## Support, Socials and Licensing

<a href="https://git.hush.is/hush/hush3#support-and-socials"> Support, Socials and Licensing</a>