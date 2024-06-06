### This guide is based on existing ROS2, ROS documentation as well as
### Raspi Docs, multiple SO threads, and miscellaneous sources otherwise undocumented.



# ROS2 SETUP


### SKIP FIRST PART IF THE RASPBERRY PI IS NOT TAKEN RIGHT OUT OF THE BOX

-----------------------------------------------------------

Preliniary steps:

+ Mount and install the radiators. Your Raspberry Pi might need to compile the code from the source, and
radiators are necessary to help with termal throttling.
+ You can use any recent Raspberry Pi Image. For minimal ram usage, it is recommended to install the headless version 
 (Raspberry Pi OS Lite), as opposed to the desktop version.
+ It is possible to use the netinstall to quickly set up a RPI, but it requires a network
without DNS server blocks and bogus network configuration. (anti-example:eduroam)


-----------------------------------------------------------

### FLASHING THE SYSTEM

0.) Insert the SD card slot into a computer with an SD card reader.
1.) Download the raspi lite from the official website:
<!> DO NOT DOWNLOAD ANYTHING OFF OF EDUROAM IF YOU DON'T WANT TO HAVE BROKEN PACKAGES

	a.) Go to https://www.raspberrypi.com/software/operating-systems/ , Download the 'Lite' version.

	b.) Verify the SHA256 Checksum:
	
	i You can use multiple commands to accomplish it ( `openssl` or `shasum`, depending on which
	you have currently installed on your machine:

	shasum -a 256 2024-03-15-raspios-bookworm-arm64-lite.img.xz | grep (shasum on the website)

2.) unzip the image and flash the card:

```
xzcat 2024-03-15-raspios-bookworm-arm64-lite.img.xz | dd of=/dev/sdX bs=4M status=progress
```

If the file is corrupted xz utilities might throw some errors like "unrecognized file format" 
or flash it instantly with 0/0 bytes written — be sure to check.

3.) Connect the RaspberryPi Peripherals to set up the connection

	Warning: Do not connect the RPI to HEI networks like Eduroam, the network is extermely obtrusive as it blocks
	RPI/Debian repositories as well as StackOverflow and StackExchange. It is impossible to netinstall
	please check whether your network administator is blocking Cloudflare's DNS via 1.1.1.1 debug website

4)  Set up ssh, internet connection and camera using raspi-config:

	Set up your user account if you have not already done so.

	a)Set up your wifi connection using raspi-config

	System options -> Wireless LAN -> Enter SSID and Password

	b) Enable VNC and SSH in raspi-config

	The second option requires internet connection.

	c) Configure the options

	Reboot the raspi, connect the camera and enable it.
	Enable X11 forwarding either in the raspi-config or 
	through /etc/sshd_config/ file.


	d) Verify SSH connectivity

	Connect to your rasbian:

	ssh -Y univangers@IP_ADR 

	, then input your password.

	i. If you would like to see X11 applications, append -Y
	to the command to forward the applications to your X server.
	Otherwise omit the option.

	e) Disconnect the peripherals (camera, monitor, keyboard)

	From now on, you will be able to connect to your Pi through the
	network from any computer with ssh installed without the need to connect
	the monitor and external peripherals.


--------------------------------------------------------------------------------
ROS 2 SETUP
--------------------------------------------------------------------------------

We will use instructions mixed from articles as there is no official guide for  installing it from source
on Rasbian. 
Additionally, it is possible to install ROS2 on any distro provided one uses python's venv to install nessecary packges 
from pip aside from system packages.

Further reading:
https://docs.ros.org/en/crystal/Installation/Linux-Install-Binary.htmlSecond part
https://docs.ros.org/en/foxy/Installation/Alternatives/Ubuntu-Development-Setup.html



sudo apt update && sudo apt install curl gnupg2 lsb-release
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.key | sudo apt-key add -

Add the repos:
```
sudo sh -c 'echo "deb [arch=$(dpkg --print-architecture)] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2-latest.list'

sudo apt update
```
!! Don't skip ahead here as going ahead and installing dependencies might break your package system. !!

Install ROS dev tools and flake8 lint extensions:

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


--------------------------------------------------------------------------------------
COMPILATION
--------------------------------------------------------------------------------------

Approach a) on the Raspi

! Requires a better disk or a swap space. !

Change the raspberry Pi's space to above 4GB.

sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile

CONF_SWAPSIZE=100 #change this to 4096

And override the MAX SWAPSIZE limit:
https://forums.raspberrypi.com/viewtopic.php?t=353964

CONF_MAXSWAP=2048 # change this to 4096


Clone the source repositiories for the ROS2 version of choice, then:

colcon build --symlink-install

It's not recommended as the potential Read/Writes during the compilation to 
could wear out the SD card — this storage medium is not resistant.
It is possible to get an official NVMe drive for the Raspberry Pi ( or just plug in a 
USB drive) with a swapfile on it to use it as swap space. This has not been tested by me.

Also do note that QT libraries will fail if you don't have QT framework installed.


Approach b) DOCKER

This approach is harder, but significantly faster in its compilation.
Be aware that if you are missing any dependencies, you will be unable to continue —
You will need to install the appriopriate dependencies on the raspberry pi.

#1.Install the cross-compilation toolchain:


#sudo apt-get install libc6-dev-armhf-cross g++-arm-linux-gnueabihf gcc-arm-linux-gnueabihf


1. Download the cROScompile script
(i) This is a WIP script of mine that is supposed to automate the docker build process.

The script either works in online or offline mode. Online mode detects the architecture as well
as rsyncs the libraries on the pi through ssh. Offline mode works when you have the apriopriate libraries mounted,
but is less error-prone.

<!> For whatever reason, rsync might bug out and enter the symlink loop. This error has roughly
one result in the entire stack exchange and it might be linked to using mismatched versions of rsync.
In that case it's just easier to do a sneakernet approach and copy the files over locally.

<!> Check the dependencies if broken/not installed, and sync your changes with rsync inside of the docker.


```
git clone
```

2. Execute it in online mode, OR
Execute it in offline mode, sync the packages in docker. 





FINALIZING INSTALLATION: (both approaches)

Source the setup file.

. /ros2_iron/install/setup.bash

Add it to your .bashrc file as to have it sourced on startup.


Additional dependencies:
OpenCV
libboost-python-dev