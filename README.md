# pi-hole-kas

KAS configuration files for pi-hole image built in yocto for RPi4
The build takes place in a KAS container running within a virtualenv

On a clean ubuntu 22.04LTS minimal image python3 is installed by default.

## Initialisation of build machine

Install docker following instructions at https://docs.docker.com/engine/install/ubuntu/ including the reboot after adding ${USER} to docker group.


## Install other packages

```
sudo apt-get install git python3-pip python3-virtualenv
```

## Build the image
Download the kas files that define the yocto recipes. Check out the pi-hole 6.0.6 version that is based on Scarthgap version of yocto

```
git clone https://github.com/HunterEmbedded/kas-pi-hole.git
cd kas-pi-hole
git checkout scarthgap-pi-hole-6.0.6
```

Install virtualenv and call it pi-hole-build
```
virtualenv pi-hole-build
```
and activate it.
```
source pi-hole-build/bin/activate
```
and install kas
```
pip3 install kas
```
Now create the kas container with the rpi4 yocto configuration
```
kas-container shell kas/rpi4.yaml
```

## Subsequent builds only need

```
cd kas-pi-hole
source pi-hole-build/bin/activate
kas-container shell kas/rpi4.yaml
```


The image and update bundle are built with
```
bitbake pi-hole-bundle
```

## Flash the Image

### RaspberryPi 4
Use the Raspberry Pi Imager to programme an SD card. 

In the GUI under Operating System chose the "Use custom" option and select the file 
`build/tmp/deploy/images/raspberrypi4-64/pi-hole-image-6.0.6-raspberrypi4-64.rootfs-<timestamp>.img`

Once programmed and the SD card is inserted in the RPi4 power it on.
The pi-hole application will start automatically. Use your favourite method to find the IP address it has been allocated on your network.

To log into the pi-hole UI 
`http://<IP>/admin`

To log in via ssh or a serial terminal the username is "admin" and the password "pihole"



## Add a new bundle to the other partition
RAUC adds into the .img file a pair of rootfs partitions A and B as well as a /data partition.
After programming the .img file to the SD card the system will boot using rootfs A and then move all the pi-hole configuration files from /etc to /data/etc using symlinks.

If a new RAUC bundle is available it can be installed into rootfs B.

From a another PC copy the new .raucb file to /data/update on the RPi4. This directory is owned by admin and so is writeable.

```
scp build/tmp/deploy/images/raspberrypi4-64/pi-hole-image-6.0.6-raspberrypi4-64-<timestamp>.raucb admin@<RPI4 IP>:/data/update/pi-hole-bundle.raucb
```

Then it can be installed from a shell running on the RPi 4. The install operation will write the contents of the bundle to rootfs B. Implicit in the install operation is an update of the u-boot variables that on the next boot the other partition (ie B) should be used.

```
rauc install /data/update/pi-hole-bundle.raucb
rm /data/update/pi-hole-bundle.raucb
sudo sytemctl reboot
```

After reboot once pi-hole service has successfully started the partition B will be marked as good and so used for subsequent boots.
On rootfs B the pi-hole configuration files are symlinked to the /data partition and so any customisations are maintained.
