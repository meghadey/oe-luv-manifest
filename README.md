Linux UEFI Validation Build (OE LUV)
===================================
This README file contains information on a reference implementation of an OpenEmbedded based LUV distro.

Maintainers
===========
* Megha Dey <mailto:megha.dey@intel.com>

Support
=======
If you find any bugs please report them here
	https://github.com/intel/oe-luv-manifest/issues

If you have questions or feedback, please subscribe to
	https://lists.01.org/mailman/listinfo/luv


Table of Contents
=================

    I. Introduction
   II. Host package Dependencies
  III. OE LUV Layers
   IV. oe-luv-manifest repository
    V. Setup Environment
   VI. Using GIT with a proxy
  VII. Building LUV

This document provides instructions to get started with building the LUV project.

I. Introduction
===============

This is not an introduction on OpenEmbedded or Yocto Project. If you are not familiar with OpenEmbedded and the Yocto Project, it is very much recommended to read the appropriate documentation first. For example, you can start with:
* http://openembedded.org/wiki/Main_Page
* http://yoctoproject.org/
* https://www.yoctoproject.org/documentation

In this wiki, we assume that the reader is familiar with basic concepts of OpenEmbedded.

II. Host package Dependencies
=============================
In order to successfully set up your build environment, you will need to install the following package dependencies.

1. You will need git installed on your Linux host machine

	`$ sudo apt-get install git`

2. Visit the OpenEmbedded (Getting Started) wiki to see which distribution specific dependencies you will need

	http://www.openembedded.org/wiki/Getting_started

3. Setting up the build environment will first search for `whiptail`, if it is not present then it will search for `dialog`. You only need one of the following packages to ensure your setup-environement runs correctly:

	`$ sudo apt-get install whiptail`

	or

	`$ sudo apt-get install dialog`

4. **Please Note**: If you are running Ubuntu 16.04 you will need to add the following line to your `/etc/apt/sources.list`

	`deb http://archive.ubuntu.com/ubuntu/ xenial universe`

All required dependencies should now be installed on your host environment.

When downloading from behind a proxy (which is common in some corporate environments), it might be necessary to explicitly specify the proxy that is then used by repo:

export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

More rarely, Linux clients experience connectivity issues, getting stuck in the middle of downloads (typically during "Receiving objects"). It has been reported that tweaking the settings of the TCP/IP stack and using non-parallel commands can improve the situation. You need root access to modify the TCP setting:

sudo sysctl -w net.ipv4.tcp_window_scaling=0
repo sync -j1


III. OE LUV Layers
=================

|   Layer    |   Description                    |
|:----------:|:----------------------|
| meta-luv   |  This is the LUV specific layer which has all the test-suite recipes and scripts used to build and run the LUV images |
| meta-oe    |  LUV needs this layer as it houses the plymouth and ndctl recipes |
| meta       |  This layer is used to get the base recipes for the toochain. |

IV. oe-luv-manifest repository
==============================

These are the setup scripts required for the OE LUV buildsystem. If you want to (re)build packages or images for OE RPB, this is the repo to use.
	git clone https://github.com/meghadey/oe-luv-manifest.git

The OE LUV buildsystem is using various components from openembedded, most importantly the Openembedded buildsystem and the bitbake task executor.

To configure the scripts and download the build metadata, do:

1. mkdir ~/bin

2. PATH=~/bin:$PATH

3. curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

4. chmod a+x ~/bin/repo

5. Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included will be placed within your working directory. To check out the current branch, specify it with -b:
	repo init -u https://github.com/meghadey/oe-luv-manifest.git -b master
When prompted, configure Repo with your real name and email address.
A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.

6. To pull down the metadata sources to your working directory from the repositories as specified in the default manifest, run
	repo sync

You are ready to begin your build environment setup.


V. Setup Environment
====================
It is recommended to use the `setup-environment` script. Among several things, it will prompt the user for the list of supported machines and distros. The setup script will also create (if they don't exist) the local build configuration files.

To run the setup script:

    $ . setup-environment

And follow the instructions. If you already know the value for MACHINE and DISTRO, it is possible to set them as environment variables, e.g.

    $ MACHINE=qemux86-64 DISTRO=luv source setup-environment


VI. Using GIT with a proxy
==========================

To use git with a proxy, you must use an external git proxy command.

To do so:
1. Uncomment the following in site.conf (<oe-luv-manifest repo>/build-<DISTRO>/conf)
            GIT_PROXY_COMMAND ?= "oe-git-proxy"
2. Replace "oe-git-proxy" with the git proxy command script on your host system.


VII. Building LUV
=================

LUV can boot from either a local network or a disk. The process to build
is slightly different for each case.

By default, we build from a physical disk (DISTRO = "luv").
One simply needs to do:

  bitbake luv-live-image

If you intend to build an image to boot from a local network, then it is
necessary to specify the DISTRO as luv-netboot.
e.g.:

  DISTRO = "luv-netboot"

Afterwards, you can simply do:

  bitbake luv-netboot-image

At the end of the build, your build artifacts will be found under `tmp-<DISTRO>/deploy/images/<MACHINE>/`.
