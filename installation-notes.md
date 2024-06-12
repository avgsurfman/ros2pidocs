This guide is based on existing ROS2 Documentation, as well as
Raspberry Pi Docs [^1], multiple SO threads, and miscellaneous sources otherwise undocumented.



# ROS2 on Raspberry Pi
A guide to setup and compile from source ROS2 on Debian bookworm.


### SKIP THE FIRST PART IF THE RASPBERRY PI IS NOT TAKEN RIGHT OUT OF THE BOX

-----------------------------------------------------------

First:

+ Mount and install the radiators. Your Raspberry Pi might need to compile the code from the source, and
radiators are necessary to help with termal throttling.
+ You can use any recent Raspberry Pi Image. For minimal ram usage, it is recommended to install the headless version 
 (Raspberry Pi OS Lite), as opposed to the desktop version.
+ It is possible to use the netinstall to quickly set up a RPI, but it requires a network
without DNS server blocks and bogus network configuration. (anti-example:eduroam)


-----------------------------------------------------------

## FLASHING THE SYSTEM

Insert the microSD card slot into a computer with an SD card reader.
You might want to get a microSD to SD adapter. Our kits come with one.
Then, mount the SD card on your filesystem.


Download the Raspberry Pi Lite image from the official website.

> [!TIP]
> It is also possible to netinstall[^2], which is easier than the 
> listed installation method. Follow the instructions on RPI docs.

> [!WARNING]
> DO NOT DOWNLOAD ANYTHING OFF OF EDUROAM IF YOU DON'T WANT TO HAVE BROKEN PACKAGES
> For whatever reason, some Eduroam configurations like to block Debian repositories, as well as 
> (somehow) break tarballs by skipping certain files while installing packages via `apt-get`
> Feel free to scream at your network administrator if that's the case!
> Netinstall is also not available on Eduroam.

1. Go to https://www.raspberrypi.com/software/operating-systems/ , Download the 'Lite' version.

2. Verify the SHA256 Checksum:

	ℹ️ There are multiple ways to do this. You may use `openssl` or `shasum`, depending on which
	you have currently installed:
```
shasum -a 256 2024-03-15-raspios-bookworm-arm64-lite.img.xz | grep (shasum on the website)
```

3. Unzip the image and flash the card:

```bash
xzcat 2024-03-15-raspios-bookworm-arm64-lite.img.xz | dd of=/dev/sdX bs=4M status=progress
```

If the file is corrupted xz utilities might throw some errors like `"unrecognized file format"` ,or 
flash it instantly with `0/0` bytes written — be sure to check.

Connect the Raspberry Pi Peripherals to set up the connection to the internet

## Wireless Network setup

### Set up ssh, internet connection and camera using raspi-config:

Set up your user account if you have not already done so.

1. Set up your wifi connection using `raspi-config`

```System options -> Wireless LAN -> Enter SSID and Password```

2. Enable VNC and SSH in raspi-config

3. Configure the options

	Reboot the raspi, connect the camera and enable it.
	Enable X11 forwarding either in the raspi-config or 
	through /etc/sshd_config/ file.

4. Verify SSH connectivity

	Connect to your rasbian:
	```bash
	ssh -Y univangers@IP_ADR
 	```
	, then input your password.

   	> [!NOTE]
    	> If you would like to use X11 applications through X forwarding, append `-Y` or `-X`
	> to the command to forward the applications to your X server.

	(Optional) Disconnect the peripherals (camera, monitor, keyboard)

	From now on, you will be able to connect to your Pi through the
	network from any computer with ssh installed without the need to connect
	the monitor and external peripherals.


--------------------------------------------------------------------------------
ROS 2 SETUP
--------------------------------------------------------------------------------

This part is based on the following guides below:
https://docs.ros.org/en/crystal/Installation/Linux-Install-Binary.html
https://docs.ros.org/en/foxy/Installation/Alternatives/Ubuntu-Development-Setup.html


Additionally, it is possible to install ROS2 on any distro provided one uses python's venv to install nessecary packges 
from pip aside from system packages.


## Adding the GPG keys

```bash
sudo apt update && sudo apt install curl gnupg2 lsb-release
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.key | sudo apt-key add -
```

Add the repos:
```
sudo sh -c 'echo "deb [arch=$(dpkg --print-architecture)] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2-latest.list'

sudo apt update
```

## Install ROS dev tools and flake8 lint extensions:

```
sudo apt update && sudo apt install -y \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest-cov \
  python3-flake8-blind-except \
  python3-flake8-builtins \
  python3-flake8-class-newline \
  python3-flake8-comprehensions \
  python3-flake8-deprecated \
  python3-flake8-import-order \
  python3-flake8-quotes \
  python3-pytest-repeat \
  python3-pytest-rerunfailures \
  ros-dev-tools
```

## Installing depenencies:

Run:
```
sudo apt upgrade
```

Then run `rosdep`:

```bash
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
```

You might need to skip some packages and install them manually[^4].



--------------------------------------------------------------------------------------
COMPILATION
--------------------------------------------------------------------------------------


## Approach A: on the Raspi

! Requires a better disk or a swap space. !

Change the raspberry Pi's space to above 4GB.
```bash
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile
```

```
CONF_SWAPSIZE=100 # change this to 4096
```

And override the MAXSWAPSIZE limit[^5]:
```
CONF_MAXSWAP=2048 # change this to 4096
```

Clone the source repositiories for the ROS2 version of choice, then:
```bash
colcon build --symlink-install
```
It's not recommended as the potential Read/Writes during the compilation to 
could wear out the SD card — this storage medium is not resistant.
It is possible to get an official NVMe drive for the Raspberry Pi ( or just plug in a 
USB drive) with a swapfile on it to use it as swap space. 

> [!NOTE] Note that QT libraries will fail to compile as you don't have QT framework installed.


## Approach B: DOCKER

This approach is harder, but significantly faster in its compilation.
Be aware that if you are missing any dependencies, you will be unable to continue —
You will need to install the appriopriate dependencies on the raspberry pi.

0. Copy the libraries over the network.
You can either `rsync` libraries or mount them using `sshfs`.

You can also just use [sneakernet](https://en.wikipedia.org/wiki/Sneakernet) and just copy the files manually.


1. Download the cROScompile script
(i) This is a WIP script of mine that is supposed to automate the docker build process.

The script either works in online or offline mode. Online mode detects the architecture as well
as rsyncs the libraries on the pi through ssh. Offline mode works when you have the apriopriate libraries mounted,
but is less error-prone.

<!> For whatever reason, rsync might bug out and enter the symlink loop. This error has roughly
one result in the entire stack exchange and it might be linked to using mismatched versions of rsync.
In that case it's just easier to do a sneakernet approach and copy the files over locally.

<!> If compilation fails, check dependencies first. Compilation can also fail if you are using musl instead of gcc coreutils.


``` bash
git clone https://github.com/avgsurfman/crospile.git
```
Then:
```bash
./cROSpile.py --offline --arch=aarch64
```

In docker, if you haven't copied the libraries yet:

```
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.47:/{lib,usr} /home/develop/rootfs
```
Consider using a lighter cypher to give RPI less load.

## FINALIZING INSTALLATION: (both approaches)

Source the setup file.

```bash
. /ros2_iron/install/setup.bash
```
Add it to your .bashrc file as to have it sourced on startup.




## 📖 Further reading 

### ROS2 Installation on Debian
* https://docs.ros.org/en/crystal/Installation/Linux-Install-Binary.htmlSecond part
* https://docs.ros.org/en/foxy/Installation/Alternatives/Ubuntu-Development-Setup.html

### Cross-compilation
* https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/
* https://tttapa.github.io/Pages/Raspberry-Pi/C++-Development-RPiOS/Development-setup.html
* https://medium.com/@stonepreston/how-to-cross-compile-a-cmake-c-application-for-the-raspberry-pi-4-on-ubuntu-20-04-bac6735d36df

### CMAKE
* https://cmake.org/cmake/help/book/mastering-cmake/index.html





[^1]:Links to main sources:
https://www.raspberrypi.com/documentation/
https://docs.ros.org/en/iron/index.html

[^2]:https://www.raspberrypi.com/documentation/computers/getting-started.html#install-over-the-network

[^3]:https://github.com/cyberbotics/epuck_ros2/blob/master/installation/cross_compile

[^4]:https://stackoverflow.com/questions/77899289/how-to-solve-the-error-cannot-locate-rosdep-definition-for-pcl

[^5]:https://forums.raspberrypi.com/viewtopic.php?t=353964




