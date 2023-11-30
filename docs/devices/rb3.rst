..
 # Copyright (c) 2023, Linaro Ltd.
 #
 # SPDX-License-Identifier: MIT

RB3
===

RB3 is one of the supported dev-boards listed on
`source.android.com <https://source.android.com/docs/setup/create/devices>`_.

The vendor-packages required to build RB3 images with AOSP are
listed `here <http://releases.devboardsforandroid.linaro.org/vendor-packages>`_.


Getting started with RB3 (also known as DB845c)
-----------------------------------------------

Learn about your RB3 board as well as how to prepare and set up for basic
use from the
`96boards DB845c getting started page <https://www.96boards.org/documentation/consumer/dragonboard/dragonboard845c/getting-started/rb3-kit/>`_.

Make sure you are running AOSP (ptable compatible) bootloader on DB845c. Latest
bootloader binaries (build #97 and above) are `hosted here
<http://snapshots.linaro.org/96boards/dragonboard845c/linaro/rescue/>`_.

For flashing instructions checkout
`96boards DB845c recovery page <https://www.96boards.org/documentation/consumer/dragonboard/dragonboard845c/installation/board-recovery.md.html>`_.

NOTE:

You can also update bootloader binaries by running **flashall** script, which is
a part of the vendor package of the RB3 AOSP build target. Boot in fastboot mode
and run following command from your HOST machine::

   git clone https://gerrit.devboardsforandroid.linaro.org/linaro-vendor-package
   cd src/db845c/dragonboard-845c-bootloader-ufs-aosp/
   ./flashall


Install pre-built AOSP images on RB3
------------------------------------

Linaro create daily AOSP builds for DB845c that user can download, flash and
boot from. If you are interested in prebuilt AOSP images for DB845c and want to
avoid compiling your own, please download and flash boot.img, vendor_boot.img,
super.img and userdata.img from
`the snapshot here <http://snapshots.linaro.org/96boards/dragonboard845c/linaro/aosp-master>`_.

Flash downloaded AOSP images by running following commands, while booted
in fastboot mode::

   fastboot flash userdata userdata.img
   fastboot flash super super.img
   fastboot flash vendor_boot vendor_boot.img
   fastboot flash boot boot.img


Compile AOSP from sources for RB3
---------------------------------

#. Download the AOSP source tree and build db845c-userdebug build target:

::

   repo init -u https://android.googlesource.com/platform/manifest -b master
   repo sync -j`nproc`
   ./device/linaro/dragonboard/fetch-vendor-package.sh
   source ./build/envsetup.sh
   lunch db845c-trunk_staging-userdebug
   make -j`nproc`


#. Install:  `Boot DB845c into fastboot mode <https://www.96boards.org/documentation/consumer/dragonboard/dragonboard845c/installation/board-recovery.md.html#booting-into-fastboot>`_ and run following command:

::

   ./device/linaro/dragonboard/installer/db845c/flash-all-aosp.sh

You can also perform QDL board recovery by running following script after
booting DB845c in `USB flashing mode <https://www.96boards.org/documentation/consumer/dragonboard/dragonboard845c/installation/board-recovery.md.html#connecting-the-board-in-usb-flashing-mode-aka-edl-mode>`_:

::

   ./device/linaro/dragonboard/installer/db845c/recovery.sh


Building the kernel for RB3
---------------------------

The **Preferred** option is to build DB845c Android GKI kernel artifacts using
official Bazel build. Run the following commands to clone the kernel source,
prebuilt Android toolchains and build scripts.

::

   mkdir repo-db845c
   cd repo-db845c
   repo init -u https://android.googlesource.com/kernel/manifest -b common-android-mainline
   repo sync -j`nproc`
   tools/bazel clean
   tools/bazel run //common:db845c_dist

Now delete all the objects in
$(AOSP_TOPDIR)device/linaro/dragonboard-kernel/android-mainline/, then copy
build artifacts from out/db845c/dist/ to
$(AOSP_TOPDIR)/device/linaro/dragonboard-kernel/android-mainline/

If you want to properly test the GKI kernel, you should

* grab the latest kernel_aarch64 build from
  https://ci.android.com/builds/branches/aosp_kernel-common-android-mainline/grid?

* under artifacts, download the Image.gz and copy it to
  $(AOSP_TOPDIR)/device/linaro/dragonboard-kernel/android-mainline/

Then rebuild AOSP using:

::

   make TARGET_KERNEL_USE=mainline -j`nproc`'

