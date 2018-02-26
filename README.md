# Westeros WPE Image for HiKey

This documents contains instructions to build westeros-wpe image for hikey(32bit userspace on 64bit kernel) based on OE RPB buildsystem. This image contains Westeros Compositor along with sample application and Metrological Webkit for Wayland browser on latest Yocto Morty(2.2) branch.

# Prerequisite

The basic Yocto build environment should be available in the host system. See the below links for setting up Yocto build  environment on your host and other requirements


http://www.yoctoproject.org/docs/2.1/yocto-project-qs/yocto-project-qs.html#yp-resources 

http://www.yoctoproject.org/docs/2.1/ref-manual/ref-manual.html#intro-requirements 


# Downloading the code

1) Setup OE RPB system

$ mkdir ~/bin

$ PATH=~/bin:$PATH

$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

$ chmod a+x ~/bin/repo

Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory. To check out the current branch, specify it with -b:


$ repo init -u https://github.com/96boards/oe-rpb-manifest.git -b morty

When prompted, configure Repo with your real name and email address.

A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.


2) Add Westeros-WPE overlay

$ cd .repo

$ git clone https://github.com/psivasubramanian/westeros_wpe_manifest.git local_manifests

$  cd ..


3) Sync

To pull down the metadata sources to your working directory from the repositories as specified in the default manifest, run


$ repo sync

When only downloading from behind a proxy (which is common in some corporate environments), it might be necessary to explicitly specify the proxy that is then used by repo:

$ export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

$ export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

More rarely, Linux clients experience connectivity issues, getting stuck in the middle of downloads (typically during "Receiving objects"). It has been reported that tweaking the settings of the TCP/IP stack and using non-parallel commands can improve the situation. You need root access to modify the TCP setting:

$ sudo sysctl -w net.ipv4.tcp_window_scaling=0

$ repo sync -j1

# Building Westeros WPE Image
4) Setup Environment

$ MACHINE=hikey-32

$ DISTRO=rpb-wayland

$ source setup-environment

This command will setup the build environment like which machine it's build for, which distrubution layer to use. 

5) Configure BBLAYERS and local.conf

$ bitbake-layers add-layer ../layers/meta-lhg/meta-lhg-wpe

$ bitbake-layers add-layer ../layers/meta-metrological/

This commands will add the required layers to the build.


6) Building image

We can build the image in two ways i.e either build both kernel and rootfs images together in single build command or seperately with different build commands.

a) To build 64bit kernel and 32bit rootfs images seperately, first build the 32bit roofts as 

$ MACHINE=hikey-32 DISTRO=rpb-wayland source setup-environment

$ bitbake lhg-westeros-wpe-image   		
	(This will build the 32bit rootfs image alone with Westeros and WPE components i.e lhg-westeros-wpe-image-hikey-32.ext4.gz in build-rpb-wayland/tmp-rpb_wayland-glibc/deploy/images/hikey-32)

	and then build the 64bit image with MACHINE=hikey as 

$ MACHINE=hikey source setup-environment

$ bitbake rpb-weston-image
	(This image will be available in build-rpb-wayland/tmp-rpb_wayland-glibc/deploy/images/hikey)


b) To build both 64bit kernel and 32bit rootfs images together, use the below command

$ MACHINE=hikey-32 DISTRO=rpb-wayland source setup-environment

$ bitbake_secondary_image --extra-machine hikey lhg-westeros-wpe-image

This will build 64bit rpb-minimal-image and 32bit lhg-westeros-wpe-image together in respective path namely hikey and hikey-32 inside build-rpb-wayland/tmp-rpb_wayland-glibc/deploy/images/ path, next we need to copy the 64bit kernel modules into 32bit rootfs image as mentioned in following section.


# Copying 64bit Kernel modules

Copy the 64bit kernel modules(/boot and /lib/modules) into 32bit rootfs before flashing the image to hikey board as below

$ gunzip rpb-weston-image-hikey.ext4.gz	(64bit kernel)

$ gunzip lhg-westeros-wpe-image-hikey-32.ext4.gz		(32bit rootfs)

$ sudo resizefs lhg-westeros-wpe-image-hikey-32-20161117131102.rootfs.ext4 500M	(resizing rootfs image to 500MB)

$ sudo mount -o loop,sync,rw rpb-weston-image-hikey.ext4 rootfs-64 (mount the 64bit image into a directory)

$ sudo mount -o loop,sync,rw lhg-westeros-wpe-image-hikey-32.ext4 rootfs-32 (mount the 32bit image into a directory)

$ sudo rsync -avz rootfs-64/boot rootfs-32/ (rsync boot files from 64bit image)

$ sudo rsync -avz rootfs-64/lib/modules rootfs-32/lib/ (rsync kernel modules from 64bit image)

$ sudo umount rootfs-64 (unmount the kernel image)

$ sudo umount rootfs-32 (unmount the rootfs)

$ ext2simg lhg-westeros-wpe-image-hikey-32.ext4 lhg-westeros-wpe-image-hikey-32.img


# Flashing Instructions for HiKey

Follow the link https://github.com/96boards/documentation/wiki/HiKeyUEFI#flash-binaries-to-emmc- for standard flashing procedure. To flash the above build rootfs alone, use the below command with pins 5-6 closed

$ sudo fastboot flash system lhg-westeros-wpe-image.img


# Running Westeros and WPE

Boot the HiKey with HDMI out connected to the display TV and USBs connected to the keyboard, mouse(use USB hub) and USB-to-Ethernet adaptor. Connect the UART serial output to host using USB-to-OTG cable for running this commands.


8) Launching Westeros compositor

$ export XDG_RUNTIME_DIR=/run/user/

$ LD_PRELOAD=/usr/lib/libwesteros_gl.so.0 westeros –renderer /usr/lib/libwesteros_render_gl.so.0 --enableCursor --display westeros-1-0 &

This will launch the Westeros compositor(in the background) as you can see blank screen with mouse cursor on the display. The option --enableCursor is to enable the mouse pointer while --display displayName is used to set the display name.


9) Running westeros test application

$ export XDG_RUNTIME_DIR=/run/user/

$ export WAYLAND_DISPLAY=westeros-1-0

$ westeros_test --display=westeros-1-0

This will run the westeros_test application which displays the continously rotating color triangle on the screen.


10) Running WPELauncher with westeros backend

$ export XDG_RUNTIME_DIR=/run/user/

$ export WAYLAND_DISPLAY=westeros-1-0

$ ./WPELauncher <URL>

This will launch the webpage of the provided URL using WPELauncher. It will launch youtube if no URL is provided. Make sure HiKey is connected to internet using USB-to-Ethernet adaptor and internet is working with


ping google.com

Eg:
$ ./WPELauncher http://www.google.com

You can use keyboard to input any text.


# Updating the sandbox

If you need to bring changes from upstream then use following commands

$ repo sync

Rebase your local committed changes

$ repo rebase


# Maintainer

Sivasubramanian Patchaiperumal sivasubramanian.patchaiperumal@linaro.org
