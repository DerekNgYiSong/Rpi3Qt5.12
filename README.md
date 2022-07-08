# Raspberry Pi 3 with Qt5.12.2



This steps compiles QT5.12.2 into raspberry pi 3 with Buster OS and cross compile to Rpi3 program.

Compiled successfully with system below:  
Raspberry PI OS: Buster  
Host machine: ubuntu 20.2 LTS  
IP setup in raspberry pi OS = 192.168.58.2  


Reference:  
https://www.interelectronix.com/qt-on-the-raspberry-pi-4.html  
https://github.com/PhysicsX/QTonRaspberryPi/blob/main/qt5.14.2onRaspberrypi  


--------------------------------------------
Raspberry Pi 3 Setup
--------------------------------------------
Setup raspberry pi OS into memory card and boot to RPi3  

Raspberry Pi IP setting
--------------------------------------------

#Add static ip to raspberry pi  
sudo nano /etc/dhcpcd.conf  
edit lines:  
 interface eth0  
 static ip_address=192.168.58.2/24  
//note: with dhcpcd.conf setting, internet will not forcefully go through LAN port, but from wifi connection as setup in next steps  

#add wifi connection to raspberry pi   
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf  
add these  
 network={  
  ssid="your ssid"  
  psk="password"  
 }  

Raspberry Pi Enable GL Desktop
--------------------------------------------

sudo raspi-config >>> enable KMS GL Desktop in advance option  

Raspberry Pi Fetch dependencies
--------------------------------------------
sudo nano /etc/apt/sources.list >>> uncomment deb_src  on last line  

sudo apt-get update  
sudo apt-get dist-upgrade  
sudo reboot  
sudo rpi-update  
sudo reboot  

sudo apt-get build-dep qt5-qmake

sudo apt-get build-dep libqt5webengine-data

sudo apt-get install libboost1.58-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev 

sudo apt-get install libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev

sudo apt-get install libasound2-dev libpulse-dev gstreamer1.0-omx libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa

sudo apt-get install libvpx-dev libsrtp0-dev libsnappy-dev libnss3-dev

sudo apt-get install "^libxcb.*"

sudo apt-get install flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1

sudo apt-get install libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev libavcodec-dev libavformat-dev libswscale-dev 

sudo apt-get install libgstreamer0.10-dev gstreamer-tools libraspberrypi-dev libx11-dev libglib2.0-dev 

sudo apt-get install freetds-dev libsqlite0-dev libpq-dev libiodbc2-dev firebird-dev libjpeg9-dev libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 

sudo apt-get install libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev 

sudo apt-get install libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev 

sudo apt-get install libxcb-glx0-dev libxi-dev libdrm-dev libssl-dev libxcb-xinerama0 libxcb-xinerama0-dev

sudo apt-get install libatspi-dev libssl-dev libxcursor-dev libxcomposite-dev libxdamage-dev libfontconfig1-dev 

sudo apt-get install libxss-dev libxtst-dev libpci-dev libcap-dev libsrtp0-dev libxrandr-dev libnss3-dev libdirectfb-dev libaudio-dev



sudo mkdir /usr/local/qt5pi

sudo chown pi:pi /usr/local/qt5pi


--------------------------------------------
Host Machine Setup:
--------------------------------------------

sudo apt-get update

sudo apt-get upgrade

sudo apt-get install gcc git bison python gperf pkg-config gdb-multiarch


sudo apt-get install make

sudo apt-get install libclang-dev

sudo apt-get install build-essential

mkdir /opt/qt5pi

chown meha:meha /opt/qt5pi



cd /opt/qt5pi/

mkdir tools

mkdir sysroot

mkdir sysroot/usr

mkdir sysroot/opt



wget https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz

tar xf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz 

mv -dr gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf ./tools/


wget https://download.qt.io/official_releases/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz

tar xf qt-everywhere-src-5.15.2.tar.xz 


wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py

chmod +x sysroot-relativelinks.py 



export PATH=$PATH:/opt/qt5pi/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin

nano ~/.bashrc


 
--------------------------------------------
Sync Raspberry pi files to Host for cross compile
--------------------------------------------
rsync -av --rsync-path="sudo rsync" pi@192.168.58.2:/lib sysroot

rsync -av --rsync-path="sudo rsync" pi@192.168.58.2:/usr/include sysroot/usr

rsync -av --rsync-path="sudo rsync" pi@192.168.58.2:/usr/lib sysroot/usr 

rsync -av --rsync-path="sudo rsync" pi@192.168.58.2:/opt/vc sysroot/opt

./sysroot-relativelinks.py sysroot



--------------------------------------------
Build Qt5 for raspberry pi
--------------------------------------------
mkdir build

cd build/

../qt-everywhere-src-5.15.2/configure -release -opengl es2  -eglfs -device linux-rasp-pi3-vc4-g++ -device-option CROSS_COMPILE=/opt/qt5pi/tool/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- -sysroot /opt/qt5pi/sysroot -prefix /usr/local/qt5.15 -extprefix /opt/qt5pi/qt5.15 -opensource -confirm-license -skip qtscript -skip qtwayland -skip qtwebengine -nomake tests -make libs -pkg-config -no-use-gold-linker -v -recheck

make -j8 -s

make install


//qt5.15 folder will be created

--------------------------------------------
Sync QT5 to raspberry pi
--------------------------------------------
cd /opt/qt5pi/

rsync -avz --rsync-path="sudo rsync" qt5pi pi@192.168.58.2:/usr/local


--------------------------------------------
Raspberry pi ssh deployment for QT5.12
--------------------------------------------
export QT_QPA_PLATFOMRTHEME=qt5ct

export DISPLAY=:0.0

export XAUTHORITY=/home/pi/.Xauthrity

export XDG_SESSION_TYPE=x11

Qt creator environment batch edit
--------------------------------------------
Go to project > run > system environment > batch edit > add these:
  
QT_QPA_PLATFOMRTHEME=qt5ct

DISPLAY=:0.0

XAUTHORITY=/home/pi/.Xauthrity

XDG_SESSION_TYPE=x11


-------------------------------------------
QT example cross compile
-------------------------------------------

cd /opt/qt5pi

cp -dr /opt/qt5pi/qt-everywhere-src-5.15.2/qtbase/examples/opengl/qopenglwidget/ ./

cd ./qopenglwidget

/opt/qt5pi/qt5.15/bin/qmake

make 



scp qopenglwidget pi@192.168.58.2:~/

ssh pi@192.168.58.2

./qopenglwidget



//at raspiberry pi screen, a opengl GUI is shown with ~60 calls/s




-------------------------------------------
QT GDB debug
-------------------------------------------

sudo apt-get install gdbserver

sudo apt-get install gdb-multiarch



in Qt creator, set Kits>>debugger>> new debugger Path = /usr/bin/gdb-multiarch

Add debugger into Rpi Kits.
