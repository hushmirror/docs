# Cross compiling a Hush full node daemon from AMD64 to ARM64(aarch64) CPU architecture with Docker.

**Requirements:** Docker-Engine & arm64(aarch64) architecture with Debian operating system like a <a href="https://www.raspberrypi.org/products/raspberry-pi-400/specifications/">Raspberry Pi 400</a>
*For this documentation, I setup <a href="https://dietpi.com/">DietPi</a> It is a Debian:buster OS build.*

0. ## Summary

Cross compiling a Hush full node daemon from **AMD64** to **ARM64(aarch64)** CPU architecture with Docker. 
The following instructions enables a smoother transition between your desktop/laptop to the raspberry pi device. 

Some issues I encountered *without* the virtual environment was **glibc** version differences on two different machines. Initially, I tested the installation by building & compiling on the **Broadcom BCM2711** quad-core Cortex-A72 (ARM v8) 64-bit SoC @ 1.8GHz device by itself through the normal hush3 instructions provided, but I ran into another set of problems with <a href="https://gcc.gnu.org/">gcc(g++)</a> <a href="https://packages.debian.org/sid/g++-multilib">**g++-multilib**</a> for the arm64(aarch64) architecture; there are no multilib packages for the arm64(aarch64) architecture.

*Note: This should work on all types of arm64(aarch64) devices. By cross compiling (building) on another machine for another machine type, we bypass the ARMv7 requirements. We copy over the binaries built to the Raspberry Pi. Cross compiling is where we compile on one machine to be used on another machine type; this is useful for  machines that don't have lots of hardware resources to compile on or in our case, **G++-multilib** isn't built for ARM64(aarch64) devices (at this point in spacetime) and hushd requires it to compile the fresh binaries.*

1. ### Let's Begin!
- Install & setup Docker-Engine <a href="https://docs.docker.com/engine/install/ubuntu/"> Docker-Engine Installation </a>

`sudo docker run -it --rm -v /tmp:/tmp debian:buster`

*Note: we chose this debian type because it has the same ldd --version (glibc) on it. At the moment, there is no DietPi docker image to pull from.*

This creates a container with the latest version of Debian. A bind mount is created mapping the /tmp directory on the host machine to the /tmp directory on the container. The additional options mean the container will run in interactive mode with a terminal, and the container will be destroyed when you stop(exit) it.

`apt-get -y update && apt-get -y upgrade`

2. ### Install Hush build depedencies
*Note: the dependencies for arm64(aarch64) are slightly different; the extra deps are **libboost-dev**, **libdb++-dev**, **libwolssl-dev** and **g++-aarch64-linux-gnu**; libwolf and g++-arch-linux-gnu will be installed later*

`apt-get -y install build-essential pkg-config libc6-dev m4 g++-multilib autoconf libtool ncurses-dev unzip git python zlib1g-dev wget bsdmainutils automake nano curl unzip libsodium-dev libboost-dev libdb++-dev`

3. ### Add to sources list 
To install a dep *wolfssl-dev* that isn't on the normal *debian:latest*, we must add to our */etc/apt/sources.list*.
`nano /etc/apt/sources.list`

At the bottom of the list, type or copy/paste the following line
#### libwolfssl-dev dependency
`deb http://ftp.us.debian.org/debian buster-backports main`

CTRL + X then Y for Yes to confirm and save changes.

`apt-get -y update`

#### Install mo dependencies:

*Note: We install these deps later because of other dep requirements.*
`apt-get -y install libwolfssl-dev`

- Install GNU C++ compiler for the arm64(aarch64) architecture
`apt-get -y install g++-aarch64-linux-gnu`

4. ### Pull or clone latest hush3 from git.hush.is
`git clone https://git.hush.is/hush/hush3.git && cd hush3`

5. ### Build and compile for aarch64-linux-gnu device
`HOST=aarch64-linux-gnu ./build.sh -j$(nproc)`

Relax and let the code flow  [cat drinking coffee meme] [matrix meme] It normally takes 15-30min to compile successfully for me.
 
6. ### Copy binaries to ARM64(aarch64) device.

On success of building the Hush binaries, we copy them to our ARM64(aarch64) device and exit the Docker container.
 
The following cp command used in the **Docker container terminal** copies from docker container to actual machine environment.

`cp -rv ../hush3/ /tmp/hush-arm64`

On the actual **machine environment terminal** we copy all folders and files to raspberry pi drive. The installation location is up to you, but for simplicity sake, I added **/home/user/hush3-foler-name** as a placeholder. If you have a GUI, you can simply drag / drop or CTRL + C and CTRL + V the folder to the drive if copying between drives is troublesome. Remember though, if copying with GUI, you may run into permission problems.

`sudo cp -rv /tmp/hush-arm64/ /media/username/external-drive/home/username/hush-arm64`

#### Changing permissions recursively for every file and folder
*Tip: chmod 755: Only owner can write, read and execute for everyone.*

`sudo chmod 755 -Rv /tmp/hush-arm64`

#### Changing ownership of file/folder for rpi device
*Tip: for security purposes, I recommend creating a new username for the rpi hush server; using root for any application is bad security practice*

`sudo chown -Rv rpi-username /media/username/external-drive/home/dietpi/hush-arm64/`

`exit` ***Note: on `exit`, the docker container will be completely destroyed!***.

7. ### Launch a fresh hush daemon(node) puppy on ARM64(aarch64)

`./src/hushd --version`

We have successfully cross-compiled a hushd for a arm64(aarch64) from amd64 or other architecture.
*Note: this has only been tested from amd64 architecture; we will update this list later for other tested hardware; the process will almost be identical, but other deps may be needed*

Happy Hacking! [Hooray!]

<img src="https://git.hush.is/hush/memes/raw/branch/master/hush/hush-puppy-rocket.gif">

## Automation with Docker

I will update the steps required to automatically build and run hushd with a Dockerfile. This is useful for testing and also aids in other hush puppies who don't need to deep dive into developer chores and maintainance. Required dep is Docker-Engine.

### Create Hush docker image
0. `git clone https://git.hush.is/hush/hush3.git && cd hush3/`
1. `docker build -t hush:3.8.0 .`
### List docker images & run Hush docker image
2. `docker images` 
3. Look for the docker image ID for hush:3.x.x
4. `docker run imageID`
5. `hushd --version`

## Support, Socials and Licensing

<a href="https://git.hush.is/hush/hush3#support-and-socials"> Support, Socials and Licensing</a>

This documentation, formatting, testing, hacking and many headaches created this after about ~34 hours of tedious focus and work.
