# raspi3-ip-camera
Raspberry 3 as IP Camera with mjpgstreamer

running in our lab on lasercam.lab.fablab.uni-erlangen.de

# Raspberry Pi 3, with camera module, raspbian

# build mjpg-streamer
```
ssh pi@lasercam
git clone https://github.com/jacksonliam/mjpg-streamer/
cd mjpg-streamer/
# according to README:
sudo apt-get install cmake libjpeg8-dev gcc g++
cd mjpg-streamer-experimental
make
```

# install as debian package (to allow for clean uninstallation)
## install fpm (for building the deb package)
```
sudo apt install ruby-full ruby-ffi ruby-dev build-essential
sudo gem install --no-ri --no-rdoc fpm
```

## build and install deb package
```
./makedeb.sh
sudo dpkg -i *.deb
```

## setup system service (autostart)
```
sudo cp /lib/systemd/system/mjpg_streamer@.service /lib/systemd/system/mjpg_streamer_pi.service
sudo -e /lib/systemd/system/mjpg_streamer_pi.service
# change the ExecStart line to:      ExecStart=/usr/bin/mjpg_streamer -i 'input_raspicam.so -x 3000 -y 2100 -quality 99' -o 'output_http.so -w /usr/share/mjpg_streamer/www'
# TODO: the full resolution 3280x2464 does not work... :-(
sudo systemctl start mjpg_streamer_pi
sudo systemctl status mjpg_streamer_pi

# start automatically after reboot:
sudo systemctl enable mjpg_streamer_pi
```


## Usage

Fetch JPG from:  http://lasercam.lab.fablab.uni-erlangen.de:8080/?action=snapshot   (URL is only accessible from internal network)
