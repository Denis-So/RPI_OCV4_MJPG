Based on https://solarianprogrammer.com/2018/12/18/cross-compile-opencv-raspberry-pi-raspbian/ manual.


# RPI_OCV4_MJPG
If you are trying to use USB camera in MJPG mode on Raspberry 3 with OpenCV and Python here is what helped to me:
1. Build OpenCV in cross-compilation. 
a: Prepare Debian Buster(10.5) virtual machine
b: Prepare VM, and build:

apt upgrade && sudo apt upgrade
sudo dpkg --add-architecture armhf
sudo apt update
sudo apt install qemu-user-static
apt-get install python3-dev
apt-get install python3-numpy
apt-get install python-dev
apt-get install python-numpy
apt-get install libpython2-dev:armhf
apt-get install libpython3-dev:armhf
sudo apt-get install libgtk-3-dev:armhf

apt install libtiff-dev:armhf zlib1g-dev:armhf
apt install libjpeg-dev:armhf libpng-dev:armhf
apt install libavcodec-dev:armhf libavformat-dev:armhf libswscale-dev:armhf libv4l-dev:armhf
apt install libxvidcore-dev:armhf libx264-dev:armhf
apt install crossbuild-essential-armhf
apt install gfortran-arm-linux-gnueabihf
sudo apt install cmake git pkg-config wget

cd ~
mkdir opencv_all && cd opencv_all
wget -O opencv.tar.gz https://github.com/opencv/opencv/archive/4.1.0.tar.gz
tar xf opencv.tar.gz
wget -O opencv_contrib.tar.gz https://github.com/opencv/opencv_contrib/archive/4.1.0.tar.gz
tar xf opencv_contrib.tar.gz
rm *.tar.gz


export PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig:/usr/share/pkgconfig
export PKG_CONFIG_LIBDIR=/usr/lib/arm-linux-gnueabihf/pkgconfig:/usr/share/pkgconfig

apt install gstreamer1.0-tools:armhf libgstreamer1.0-dev:armhf libgstreamer-plugins-base1.0-dev:armhf

cd opencv-4.1.0
mkdir build && cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
     -D CMAKE_INSTALL_PREFIX=/opt/opencv-4.1.0 \
     -D CMAKE_TOOLCHAIN_FILE=../platforms/linux/arm-gnueabi.toolchain.cmake \
     -D OPENCV_EXTRA_MODULES_PATH=~/opencv_all/opencv_contrib-4.1.0/modules \
     -D OPENCV_ENABLE_NONFREE=ON \
     -D ENABLE_NEON=ON \
     -D ENABLE_VFPV3=ON \
     -D BUILD_TESTS=OFF \
     -D BUILD_DOCS=OFF \
     -D PYTHON2_INCLUDE_PATH=/usr/include/python2.7 \
     -D PYTHON2_LIBRARIES=/usr/lib/arm-linux-gnueabihf/libpython2.7.so \
     -D PYTHON2_NUMPY_INCLUDE_DIRS=/usr/lib/python2/dist-packages/numpy/core/include \
     -D PYTHON3_INCLUDE_PATH=/usr/include/python3.7m \
     -D PYTHON3_LIBRARIES=/usr/lib/arm-linux-gnueabihf/libpython3.7m.so \
     -D PYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include \
     -D BUILD_OPENCV_PYTHON2=ON \
     -D BUILD_OPENCV_PYTHON3=ON \
     -D WITH_GSTREAMER=ON \
     -D WITH_V4L=ON\
     -D WITH_GTK=ON\
     -D BUILD_NEW_PYTHON_SUPPORT=ON\
     -D BUILD_EXAMPLES=OFF ..
If ypu have  no errors:
make -j16
sudo make install/strip
cd /opt/opencv-4.1.0/lib/python3.7/dist-packages/cv2/python-3.7/
sudo cp cv2.cpython-37m-x86_64-linux-gnu.so cv2.so
cd /opt
tar -cjvf ~/opencv-4.1.0-armhf.tar.bz2 opencv-4.1.0
cd ~

c: Move to Raspberry
Copy opencv-4.1.0-armhf.tar.bz2 and opencv.pc from your home folder to your Raspberry Pi.
For the next part of the article, I’ll assume that you are on a RPi.
Make sure your RPi has all the development libraries we’ve used. Like before, if you don’t plan to use GTK+, ignore the first line from the next commands. Most of these libraries should be already installed if you are using the full version of Raspbian:
sudo apt install libgtk-3-dev libcanberra-gtk3-dev
sudo apt install libtiff-dev zlib1g-dev
sudo apt install libjpeg-dev libpng-dev
sudo apt install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
Uncompress and move the library to the /opt folder of your RPi:
tar xfv opencv-4.1.0-armhf.tar.bz2
sudo mv opencv-4.1.0 /opt
Optionally, you can erase the archive:
rm opencv-4.1.0-armhf.tar.bz2
Next, let’s also move opencv.pc where pkg-config can find it:
1 sudo mv opencv.pc /usr/lib/arm-linux-gnueabihf/pkgconfig
In order for the OS to find the OpenCV libraries we need to add them to the library path:
echo 'export LD_LIBRARY_PATH=/opt/opencv-4.1.0/lib:$LD_LIBRARY_PATH' >> .bashrc
source .bashrc
This step not worked  for me and i have to: 
nano /etc/ld.so.conf.d/cv.conf and put the path:
/opt/opencv-4.1.0/lib
inside of cv.conf

Sudo ldconfig

Log out and log in or restart the Terminal.
Next, let’s create some symbolic links that will allow Python to load the newly created libraries:
1 sudo ln -s /opt/opencv-4.1.0/lib/python2.7/dist-packages/cv2 /usr/lib/python2.7/dist-packages/cv2
2 sudo ln -s /opt/opencv-4.1.0/lib/python3.7/dist-packages/cv2 /usr/lib/python3/dist-packages/cv2


d: Get to know about your camera:
v4l2-ctl --list-devices will output of all dimensiona of all connected cameras:
[0]: 'MJPG' (Motion-JPEG, compressed)
		Size: Discrete 1600x1200
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
		Size: Discrete 2592x1944
			Interval: Discrete 0.067s (15.000 fps)
		Size: Discrete 2048x1536
			Interval: Discrete 0.067s (15.000 fps)
		Size: Discrete 1920x1080
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x1024
			Interval: Discrete 0.067s (15.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1600x1200
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
	[1]: 'YUYV' (YUYV 4:2:2)
		Size: Discrete 1600x1200
			Interval: Discrete 0.200s (5.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 2592x1944
			Interval: Discrete 0.333s (3.000 fps)
		Size: Discrete 2048x1536
			Interval: Discrete 0.250s (4.000 fps)
		Size: Discrete 1920x1080
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x1024
			Interval: Discrete 0.111s (9.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.050s (20.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1600x1200
			Interval: Discrete 0.200s (5.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
      
 e: v4l2-ctl -d /dev/video0 --list-ctrls
Output all v4l camera fuctions:
                     brightness 0x00980900 (int)    : min=-64 max=64 step=1 default=0 value=0
                       contrast 0x00980901 (int)    : min=0 max=64 step=1 default=32 value=32
                     saturation 0x00980902 (int)    : min=0 max=128 step=1 default=60 value=60
                            hue 0x00980903 (int)    : min=-40 max=40 step=1 default=0 value=0
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
                          gamma 0x00980910 (int)    : min=72 max=500 step=1 default=100 value=100
                           gain 0x00980913 (int)    : min=0 max=100 step=1 default=0 value=0
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=1 value=1
error 110 getting ctrl White Balance Temperature
                      sharpness 0x0098091b (int)    : min=0 max=6 step=1 default=2 value=2
         backlight_compensation 0x0098091c (int)    : min=0 max=2 step=1 default=1 value=1
                  exposure_auto 0x009a0901 (menu)   : min=0 max=3 default=3 value=3
              exposure_absolute 0x009a0902 (int)    : min=1 max=5000 step=1 default=157 value=157 flags=inactive
         exposure_auto_priority 0x009a0903 (bool)   : default=0 value=1
                 focus_absolute 0x009a090a (int)    : min=1 max=1023 step=1 default=600 value=1023 flags=inactive
                     focus_auto 0x009a090c (bool)   : default=1 value=1

f: Now you can connect to camera:
cam = cv2.VideoCapture()

cam.open(0, apiPreference=cv2.CAP_V4L2)

cam.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc('M', 'J', 'P', 'G'))
cam.set(cv2.CAP_PROP_FRAME_WIDTH, 800)
cam.set(cv2.CAP_PROP_FRAME_HEIGHT, 600)
cam.set(cv2.CAP_PROP_FPS, 30.0)

 



