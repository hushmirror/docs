# Cross-Compiling SilentDragon for Windows with MXE

This document explains how to build Windows binaries from a Linux system. This was tested and created on Debian 11 (Bullseye).

## Install Build dependencies
```
sudo apt-get -y install clang g++ build-essential make mingw-w64 git pkg-config libc6-dev m4 g++-multilib autoconf libncurses-dev libtool-bin unzip python zlib1g-dev wget curl bsdmainutils automake libgl1-mesa-dev libglu1-mesa-dev libfontconfig1-dev autopoint libssl-dev
```

## Install QT packages
```
sudo apt-get -y install qtdeclarative5-dev qt5-qmake libqt5websockets5-dev qtcreator
``` 

## Install MXE dependencies
Current list of dependencies is here: http://mxe.cc/#requirements-debian

```
sudo apt-get install \
    autoconf \
    automake \
    autopoint \
    bash \
    bison \
    bzip2 \
    flex \
    g++ \
    g++-multilib \
    gettext \
    git \
    gperf \
    intltool \
    libc6-dev-i386 \
    libgdk-pixbuf2.0-dev \
    libltdl-dev \
    libssl-dev \
    libtool-bin \
    libxml-parser-perl \
    lzip \
    make \
    openssl \
    p7zip-full \
    patch \
    perl \
    python \
    ruby \
    sed \
    unzip \
    wget \
    xz-utils
```

## Clone MXE
```    
git clone https://github.com/mxe/mxe.git
``` 
    
## Build MXE

Add `MXE_TARGETS` so we get both 32-bit and 64-bit Windows binaries.
```
cd mxe
make MXE_TARGETS='x86_64-w64-mingw32.static i686-w64-mingw32.static' openssl qtbase qtwebsockets
```
## Add path to MXE and /usr/bin
`export PATH=$PATH:`_/your/mxe/path_`/usr/bin`

## Clone SilentDragon source
```
git clone https://git.hush.is/hush/SilentDragon.git
cd SilentDragon
```

## Build latest translations
Either
`
./build.sh linguist
`
Or
`
./win-build.sh linguist
`
## Build SilentDragon Windows binaries
Within your SilentDragon repository run build script:
```
./win-build.sh release
```
Executable should be created and located within SilentDragon repository, IE: SilentDragon/release/silentdragon.exe

**Note:** Options for `win-build.sh` include: `clean`, `cleanbuild`, `linguist`, and `release`. No option specified will build debug version by default. `winbuild.sh` compiles 64-bit binary only, but can be modified to compile 32-bit. 

**Targeting 32-bit or 64-bit**
Modify qbuild_release target in `winbuild.sh` to specify 32-bit or 64-bit

To make 32-bit Windows GUI executable
`
i686-w64-mingw32.static-qmake-qt5 $CONF CONFIG+=release
`

To make 64-bit Windows GUI executable (default)
`
x86_64-w64-mingw32.static-qmake-qt5 $CONF CONFIG+=release
`
