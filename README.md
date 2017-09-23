# rdk-on-raspberrypi
Documentation for running RDK profiles ( Video, broadband, Camera ) on Raspberrypi boards

Setting up workspace

```shell
mkdir rpi-yocto
git clone git://git.yoctoproject.org/poky
git clone git://git.openembedded.org/meta-openembedded
git clone git://git.yoctoproject.org/meta-raspberrypi
git clone git://github.com/WebPlatformForEmbedded/meta-wpe
git clone git://github.com/96boards/meta-96boards

source poky/oe-init-build-env rpi-ml-build

bitbake-layers add-layer ../meta-openembedded/meta-oe
bitbake-layers add-layer ../meta-openembedded/meta-multimedia
bitbake-layers add-layer ../meta-openembedded/meta-python
bitbake-layers add-layer ../meta-raspberrypi
bitbake-layers add-layer ../meta-96boards
bitbake-layers add-layer ../meta-wpe
```

Edit conf/local.conf
Set Machine

```shell
MACHINE = "raspberrypi3"
```

you can use raspberrypi2 as well if you own raspberrypi2 machine.

Ignore QT
```shell
BBMASK = "recipes-qt"
```
Use 4.9 kernel
```shell
PREFERRED_VERSION_linux-raspberrypi = "4.9%"
```

Build WPE with Westeros Compositor

```shell
bitbake wpe-westeros-image
```

For westeros-wpe-image to runtime test. Here are steps, please document them publicly so folks using this image
Can try them out. These are validated on RaspberryPI3
 
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
