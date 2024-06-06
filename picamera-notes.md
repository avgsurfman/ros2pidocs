1. Install the picamera

sudo apt install -y python3-picamera2

(This will also pull in graphical dependencies)


Using the test script

(You might want to install x-11 apps)o


sudo apt install -y python3-opencv opencv-data
sudo apt install -y ffmpeg
sudo apt install x11-apps 

sudo apt install libboost-python-dev

# Preview options:
# QTGL (GPU accelerated)
# DKMS (Without X server)
# QT (Slow, but useful for X forwarding)

try to use NULL preview instead of DKMS

picam2.title_fields = ["ExposureTime", "AnalogueGain"]
to set a title field


transform - whether camera images are horizontally or vertically mirrored, or both (giving a 180 degree rotation).
All three streams (if present) share the same transform.
colour_space - the colour space of the output images. The main and lores streams must always share the same
colour space. The raw stream is always in a camera-speciﬁc colour space.
buffer_count - the number of sets of buffers to allocate for the camera system. A single set of buffers repre‐
sents one buffer for each of the streams that have been requested.
queue - whether the system is allowed to queue up a frame ready for a capture request.
# particularly handy if one is concerned about overhead
sensor - parameters that allow an application to select a particular mode of operation for the sensor
# lower res modes give more fps

display - this names which (if any) of the streams are to be shown in the preview window. It does not actually af‐
fect the camera images in any way, only what Picamera2 does with them.
# specify the stream lores or highres to be displayed

encode - this names which (if any) of the streams are to be encoded if a video recording is started. This too
does not affect the camera images in any way, only what Picamera2 does with them. This parameter only speci‐
ﬁes a default value that can be overridden when the encoders are started.
# speficies the encoding

STREAM params

Encoding

picam2 = Picamera2()
config = picam2.create_preview_configuration(lores={})

Image size
picam = Picamera2()
config = picam2.create_preview_configuration({"size": (808, 606)})


For RAW capture, read the Raspiberry Pi documentation


Setting FPS at runtime

picam2.create_video_configuration()["controls"]
{'NoiseReductionMode': <NoiseReductionMode.Fast: 1>, 'FrameDurationLimits': (33333, 33333)}

We can use a configuration object instead of dictionary to set up our stream

! Conﬁguration objects are persistent !

picam2.preview_configuration.main.size = (800, 600)
picam2.configure("preview")

2. Set up the project

 To use OpenCV with ROS2, you will need CV_bridge, which needs to be downloaded off github:

 https://github.com/ros-perception/vision_opencv/

Create a project folder, then install the package in the source directory.
If you'd like to recompile a single package, use 

colcon build --package-up-to




