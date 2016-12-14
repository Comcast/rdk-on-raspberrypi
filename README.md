# rdk-on-raspberrypi
Documentation for running RDK profiles ( Video, broadband, Camera ) on Raspberrypi boards

Setting up workspace

```shell
mkdir rpi-yocto
git clone git://github.com/kraj/openembedded-core
git clone git://github.com/kraj/bitbake
cd openemmbedded-core; ln -s ../bitbake;cd -
git clone git://github.com/kraj/meta-openembedded
git clone git://github.com/kraj/meta-raspberrypi
git clone git://github.com/kraj/meta-96boards
git clone git://github.com/kraj/meta-himvis
git clone git://github.com/kraj/meta-qt5 -b rdk/qt-5.1.1
git clone git://github.com/metrological/meta-metrological

source openembedded-core/oe-init-build-env rpi-ml-build

bitbake-layers add-layer ../meta-raspberrypi
bitbake-layers add-layer ../meta-96boards
bitbake-layers add-layer ../meta-metrological
bitbake-layers add-layer ../meta-himvis
bitbake-layers add-layer ../meta-qt5
bitbake-layers add-layer ../meta-openembedded/meta-oe/
bitbake-layers add-layer ../meta-openembedded/meta-multimedia/

```

copy the auto.conf to  conf/ directory

you can use raspberrypi2 as well if you own raspberrypi2 machine.

Use 4.8 kernel make this change in auto.conf

```shell
PREFERRED_VERSION_linux-raspberrypi = "4.8%"
```

Build WPE with Westeros Compositor

```shell
bitbake westeros-wpe-image
```

Flash SD-Card with Rpi Image

Note: Ensure that card is unmounted before below operation. Some OSes will mount the existing partitions
automatically on inserting the card. So umount them manually.

 ```shell
sudo dd if=tmp/deploy/images/raspberrypi3/westeros-wpe-image-raspberrypi3.rpi-sdimg of=/dev/sdX
 ```
 
where X is the letter a,b,c which your box would have mounted the uSD card on you can check that with dmesg | tail -10
when you insert the card into your computer.
 
Once booted login as ‘root’ it has no password
 
Run
```shell
mkdir -p /var/run/user/`id -u`/
chmod 0700 /var/run/user/`id -u`/
export XDG_RUNTIME_DIR=/var/run/user/`id -u`/
westeros --enableCursor --renderer libwesteros_render_dispmanx.so.0.0.0 --framerate 60 --display wayland-0&

export XDG_RUNTIME_DIR=/var/run/user/`id -u`/
export WAYLAND_DISPLAY=wayland-0
/usr/bin/WPELauncher
 ```
This should result in WPE launched on screen and you can try to play a video manually
 
Or you can launch a video like
 ```shell
/usr/bin/WPELauncher https://www.youtube.com/tv#/watch/video/control?v=-YGDyPAwQz0&resume
 ```
which will play one video automatically
 
 
Second test is to run big bunny video launch it like
 ```shell
gst-launch-1.0 souphttpsrc location="http://download.blender.org/peach/bigbuckbunny_movies/big_buck_bunny_720p_h264.mov" ! typefind ! qtdemux name=demux demux. ! queue ! h264parse ! omxh264dec ! glimagesink demux. ! queue ! faad ! autoaudiosink
```

Run QT5 demo (with wayland/westeros)

 ```shell
Qt5_CinematicExperience -platform wayland-egl
```

Run QT5 demo (with eglfs)

 ```shell
Qt5_CinematicExperience -platform eglfs -evdev
```

Building QT5 application on Pi

Copy the qt.conf file into /usr/bin/qt5/qt.conf
secondly put envsetup.sh in $HOME

```shell
source ${HOME}/envsetup.sh
cd <your-app>
qmake
make
```

If the build complains about missing .h files then you will need to install development headers and libraries e.g.
```shell
opkg install qtbase-dev
```

Resize SD-Card

Add in local.conf
```shell
CORE_IMAGE_EXTRA_INSTALL_append = " 96boards-tools "
```
and build the image again and flash it to SD-Card then run the following after firstboot.

```shell

parted /dev/mmcblk0 resizepart 2 100%
resize2fs -p /dev/mmcblk0p2
reboot

```

With systemd if dhcp does not work then you have to set /etc/resolv.conf symlink correctly.

```shell
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```
