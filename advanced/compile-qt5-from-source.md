## Configure, Build, Install & Setup Qt 5.15.2 Source on Ubuntu 18.04 with X11 Support

This tutorial will teach you how to compile from source via CLI (command-line interface) and enable customization for building Qt5 apps on different operating systems andor hardware architectures like RPI400. 

When compiling Qt5 yourself, you are empowered with a configuration that is constant throughout the Linux Universe. This process is also very similar with compiling Qt apps on your (AArch64) ARM64 devices like RPi 400, Rock64 & PineBook64.

*Note: This tutorial doesn't include json Qtwebengine*

## What is Qt?
"Qt is a cross-platform application development framework for desktop, embedded and mobile." [Qt Wiki]

## Let's Begin!

### Remove & purge all Qt packages
```
sudo apt -y remove qt5* libqt5* qtcreator && sudo autoremove
```

### Download Qt 5.15.2 Source to qt5-sources folder
```
mkdir qt5-sources && cd qt5-sources && mkdir build-shadow

wget https://download.qt.io/official_releases/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz
```

#### Verify MD5 hash
```
md5sum qt-everywhere-src-5.15.2.tar.xz
```
```
e1447db4f06c841d8947f0a6ce83a7b5  qt-everywhere-src-5.15.2.tar.xz
```

#### Un-tar Qt5 archive
```
tar xf qt-everywhere-src-5.15.2.tar.xz
```

#### Move into build-shadow directory to configure your Qt 5.15.2
```
cd build-shadow
```

### Install Qt5 Minimal Dependencies
```
sudo apt update

sudo apt install build-essential libfontconfig1-dev libdbus-1-dev libfreetype6-dev libicu-dev libinput-dev libxkbcommon-dev libsqlite3-dev libssl-dev libpng-dev libjpeg-dev libglib2.0-dev
```

#### (Optional) Install VC4 Drivers for RPi 4 type devices (i.e. cortex-a53 & cortex-a72)
```
sudo apt install libgles2-mesa-dev libgbm-dev libdrm-dev
```
### Install X11 Support Dependencies
```
sudo apt install libx11-dev libxcb1-dev  libxext-dev libxi-dev libxcomposite-dev libxcursor-dev libxtst-dev libxrandr-dev libfontconfig1-dev libfreetype6-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev  libxcb-glx0-dev  libxcb-keysyms1-dev libxcb-image0-dev  libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev  libxcb-randr0-dev  libxcb-render-util0-dev  libxcb-util0-dev  libxcb-xinerama0-dev  libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev
```

### Configure Qt 5.15.2
```
../qt-everywhere-src-5.15.2/configure -v -bundled-xcb-xinput -opensource -confirm-license -release -ssl -glib -no-feature-geoservices_mapboxgl -qt-pcre -nomake examples -no-compile-examples -nomake tests -reduce-exports -system-freetype -fontconfig -qpa xcb
```
#### Make the configuration *(-j 4 is number of cpus you want to use)*
```
make -j 4
```

#### Install Qt5 into: *(default) /usr/local/*
```
sudo make install
```
*Note: Check and make sure ./configure has required deps; if you installed the deps, but still receiving errors, remove config.cache and ./configure again.*

**Tip:** *If you want to view your configuration summary, you may do so via nano config.summary from build-shadow directory*

### Update profile to know where Qt5.15.2 bins are
```
nano ~/.bashrc
```

#### Add this at the bottom of your .bashrc file.
```
# set PATH for Qt 5.15.2
export PATH="/usr/local/Qt-5.15.2/bin:$PATH"
```

#### Reload your ~/.bashrc file & create new shell window.
```
source ~/.bashrc
CTRL+SHIFT+T
ALT+1
exit
```
#### Verify Qt 5.15.2 has been installed
```
qmake --version
```

```
QMake version 3.1
Using Qt version 5.15.2 in /usr/local/Qt-5.15.2/lib
```

### Build your happy Qt5 app

```
./build.sh
```

![happy little apps](../images/bob-ross-happy.gif)

### References
[tal.org/rpi](https://www.tal.org/tutorials/building-qt-515-raspberry-pi)

[doc.qt.io/qtmodules](https://doc.qt.io/qt-5/qtmodules.html)

[wiki.qt.io/about](https://wiki.qt.io/About_Qt)

## Support
[Hush Tech Support Telegram channel](https://t.me/hush8support) 