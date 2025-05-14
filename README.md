# pi-hole-kas

KAS configuration files for pi-hole image built in yocto for RPi4
The build takes place in a KAS container running within a virtualenv

On a clean ubuntu 22.04LTS minimal image python3 is installed by default.

##Initialisation of build machine

Install docker following instrctions at https://docs.docker.com/engine/install/ubuntu/ including the reboot after adding ${USER} to docker group.


##Install other packages

```
sudo apt-get install git python3-pip python3-virtualenv
```

Download the kas files that define the yocto recipes

```
git clone http://<gitserver>/pi-hole-kas
cd pi-hole-kas
```

Install virtualenv and call it pi-hole-build
```
virtualenv pi-hold-build
```
and activate it.
```
source pi-hold-build/bin/activate
```
and install kas
```
pip3 install kas
```
Now create the kas container with the rpi4 yocto configuration
```
kas-container shell kas/rpi4.yaml
```

##Subsequent builds only need

```
cd pi-hole-kas
source pi-hold-build/bin/activate
kas-container shell kas/rpi4.yaml
```


The image is built with
```
bitbake pi-hole-image
```


