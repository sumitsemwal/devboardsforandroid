..
 # Copyright (c) 2023, Linaro Ltd.
 #
 # SPDX-License-Identifier: MIT

RB5
===

RB5 is one of the supported dev-boards listed on
`source.android.com <https://source.android.com/docs/setup/create/devices>`_.

The vendor-packages required to build RB5 images with AOSP are
listed `here <http://releases.devboardsforandroid.linaro.org/vendor-packages>`_


Getting started with RB5
------------------------

Learn about your RB5 as well as how to prepare and set up for basic use from the
`96boards RB5 getting started page <https://www.96boards.org/documentation/consumer/dragonboard/qualcomm-robotics-rb5>`_.

Make sure you are running AOSP (ptable compatible) bootloader on RB5. Latest
bootloader binaries (build #29 and above) are `hosted here
<http://snapshots.linaro.org/96boards/qrb5165-rb5/linaro/rescue>`_.

For flashing instructions checkout `96boards RB5 recovery page <https://www.96boards.org/documentation/consumer/dragonboard/qualcomm-robotics-rb5/installation/board-recovery.md.html>`_.

NOTE:

You can also update bootloader binaries by running flashall script, which is
a part of the vendor package of RB5 AOSP build target. Boot in fastboot mode
and run following command from your HOST machine::

   git clone https://gerrit.devboardsforandroid.linaro.org/linaro-vendor-package
   cd src/rb5/rb5-bootloader-ufs-aosp/
   ./flashall


Install pre-built AOSP images on RB5
------------------------------------

Linaro creates daily AOSP builds for RB5 that user can download, flash and boot
from. If you are interested in prebuilt AOSP images for RB5 and want to avoid
compiling your own, please download and flash boot.img, vendor_boot.img,
super.img and userdata.img from
`the snapshot here <https://snapshots.linaro.org/96boards/dragonboard845c/linaro/aosp-master/>`_.

Flash downloaded AOSP images by running following commands, while booted
in fastboot mode::

   fastboot flash userdata userdata.img
   fastboot flash super super.img
   fastboot flash vendor_boot vendor_boot.img
   fastboot flash boot boot.img


Compile AOSP from sources for RB5
---------------------------------

#. Download the AOSP source tree and build rb5-userdebug build target:

::

   repo init -u https://android.googlesource.com/platform/manifest -b master
   repo sync -j`nproc`
   ./device/linaro/dragonboard/fetch-vendor-package.sh
   source ./build/envsetup.sh
   lunch db845c-trunk_staging-userdebug #DB845c builds boot on RB5 as well.
   make -j`nproc`


#. Install:  `Boot RB5 into fastboot mode <https://www.96boards.org/documentation/consumer/dragonboard/qualcomm-robotics-rb5/installation/board-recovery.md.html#booting-into-fastboot>`_ and run following command:

::

   ./device/linaro/dragonboard/installer/rb5/flash-all-aosp.sh

You can also perform QDL board recovery by running following script after
booting RB5 in `USB flashing mode <https://www.96boards.org/documentation/consumer/dragonboard/qualcomm-robotics-rb5/installation/board-recovery.md.html#connecting-the-board-in-usb-flashing-mode-aka-edl-mode>`_:

::

   ./device/linaro/dragonboard/installer/rb5/recovery.sh


Building the kernel for RB5
---------------------------

The **Preferred** option is to build RB5 Android GKI kernel artifacts using
Bazel build. Run the following commands to clone the kernel source, prebuilt
Android toolchains and build scripts.

::

   mkdir repo-rb5
   cd repo-rb5
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

ToDo / Known Issues
-------------------

* Factory version of RB5 comes with older version of lt9611uxc firmware flashed
  on it, so in case you do not see display up and running then please upgrade
  the lt9611uxc firmware version to v43 or newer.

* WiFi-BT drivers are WIP and not upstreamed yet. You can find the patches here
  https://git.linaro.org/people/amit.pundir/linux.git/log/?h=rbX-mainline

