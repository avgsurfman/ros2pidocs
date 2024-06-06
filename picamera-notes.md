## PiCamera2 Setup

0. Install picamera2

```bash
sudo apt install -y python3-picamera2
```
(This will also pull in graphical dependencies)

1. Install additional depenencies:
* OpenCV
* OpenCV-data
* libboost-python-dev
* (Optional) x11-apps

```bash
sudo apt install -y python3-opencv opencv-data
sudo apt install -y ffmpeg
sudo apt install x11-apps 
sudo apt install libboost-python-dev
```

Preview options:
- QTGL (GPU accelerated) - does **not** work with RPI LITE
- DKMS (Without X server)
- QT (Slow, but useful for X forwarding)

You can play around with the examples in the [offical repo for PiCamera2](https://github.com/raspberrypi/picamera2/tree/main/examples).



## Set up the project

 To use OpenCV with ROS2, you will need CV_bridge, which needs to be downloaded off github:

 https://github.com/ros-perception/vision_opencv/

Create a project folder, then install the package in the source directory.
If you'd like to recompile a single package, use 

```
colcon build --package-up-to
```

## Clone the test nodes
Clone the test nodes from my repository:
```bash
https://github.com/avgsurfman/PiCam2ROS
```
, then move them to your project's folder.

(Refer to the PiCamera2 manual in case of issues.)[https://datasheets.raspberrypi.com/camera/picamera2-manual.pdf]




