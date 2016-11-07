# westeros_wpe_manifest
This repository contains a local manifest that can be used to build westeros-wpe image for hikey based on OE RPB buildsystem.

# 1) Setup OE RPB system

$ mkdir ~/bin

$ PATH=~/bin:$PATH

$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

$ chmod a+x ~/bin/repo

Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory. To check out the current branch, specify it with -b:

$ repo init -u https://github.com/96boards/oe-rpb-manifest.git -b krogoth

When prompted, configure Repo with your real name and email address.

A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.

# 2) Add Westeros-WPE overlay

$ cd .repo

$ git clone https://github.com/psivasubramanian/westeros_wpe_manifest.git local_manifests

$  cd ..

# 3) Sync
To pull down the metadata sources to your working directory from the repositories as specified in the default manifest, run

$ repo sync

When downloading from behind a proxy (which is common in some corporate environments), it might be necessary to explicitly specify the proxy that is then used by repo:

$ export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

$ export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

More rarely, Linux clients experience connectivity issues, getting stuck in the middle of downloads (typically during "Receiving objects"). It has been reported that tweaking the settings of the TCP/IP stack and using non-parallel commands can improve the situation. You need root access to modify the TCP setting:

$ sudo sysctl -w net.ipv4.tcp_window_scaling=0

$ repo sync -j1

# 4) Setup Environment

MACHINE values can be:
hikey-32

DISTRO values can be:
rpb-wayland

$ source setup-environment

# 5) Configure BBLAYERS and local.conf

$ bitbake-layers add-layer ../layers/meta-lhg

$ bitbake-layers add-layer ../layers/meta-metrological/

Append the following to conf/local.conf

DISTRO_FEATURES_append = " westeros"

BBMASK = "meta-metrological/recipes-multimedia/gstreamer/gstreamer1.0-plugins-bad_1.6.3.bbappend meta-lhg/recipes-graphics"

# 6) Build

$ bitbake lof-westeros-wpe-image

This will build the 32bit rootfs image with Westeros and WPE components. Need to copy 64bit kernel modules(/boot and /lib/modules) into this rootfs before flashing the image to hikey board.

# Updating the sandbox

If you need to bring changes from upstream then use following commands

$ repo sync

Rebase your local committed changes

$ repo rebase
