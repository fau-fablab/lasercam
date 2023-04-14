# Lasercam
Raspberry 4 as IP Camera with mjpgstreamer

running in our lab on lasercam.lab.fablab.uni-erlangen.de

# Hardware

Raspberry Pi 4, with camera module


# OS

32bit Raspi OS lite

DO NOT use the 64bit stuff, it cost me two hours of angry disappointment because some things don't work yet there (as of 2023-04).

2023-02-21-raspios-bullseye-armhf-lite.img


User/PW: see brain:/mnt/secrets/passwords/lasercam.fablab.fau.de.txt

optionally EEPROM update via raspi-config - only if necessary if later something doesnt work

Enable SSH

Start `sudo raspi-config`
go to Interface - enable "Legacy Camera"
(this won't work in the future, but in the future hopefully the raspi cam will work like a normal webcam and not require special patched mjpg-streamer)

# build mjpg-streamer
```
ssh pi@lasercam
# according to README of mjpg-streamer:
sudo apt-get update
sudo apt-get -y install git cmake libjpeg9-dev gcc g++ libraspberrypi-dev
git clone https://github.com/jacksonliam/mjpg-streamer/
cd mjpg-streamer/
# currently not helpful, leads to compile error. but may be useful once they fix the errors: apt install libraspberrypi-dev 
cd mjpg-streamer-experimental
sudo ln -s /usr /opt/vc # Workaround according to https://github.com/jacksonliam/mjpg-streamer/issues/259, may no longer be needed in the future
make
```

# install as debian package (to allow for clean uninstallation)
## install fpm (for building the deb package)
```
sudo apt install ruby-full ruby-ffi ruby-dev build-essential
sudo gem install fpm
```

## build and install deb package
```
./makedeb.sh
sudo dpkg -i *.deb
```

## Debugging notes
```
# PLEASE IGNORE - ONLY FOR DEBUGGING
# sudo apt-get -y install libcamera-apps v4l-utils # testing, may no longer be needed in the future
# v4l2-ctl --list-devices
# ...
```

## setup system service (autostart)
```
sudo cp /lib/systemd/system/mjpg_streamer@.service /lib/systemd/system/mjpg_streamer_pi.service
sudo -e /lib/systemd/system/mjpg_streamer_pi.service
# change the ExecStart line to:      ExecStart=/usr/bin/mjpg_streamer -i 'input_raspicam.so -x 3000 -y 2100 -quality 99' -o 'output_http.so -w /usr/share/mjpg_streamer/www'
sudo systemctl daemon-reload
sudo systemctl start mjpg_streamer_pi
sudo systemctl status mjpg_streamer_pi

# start automatically after reboot:
sudo systemctl enable mjpg_streamer_pi
```


## Usage

Fetch JPG from:  http://lasercam.lab.fablab.uni-erlangen.de:8080/?action=snapshot   (URL is only accessible from internal network)
