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

.. note::
   You can also update bootloader binaries by running **flashall** script, which is
   a part of the vendor package of the RB3 AOSP build target. Boot in fastboot mode
   and run following command from your HOST machine:

.. code::

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

#. Download the AOSP source tree and build db845c-trunk_staging-userdebug build target:

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


Booting AOSP from MMC Sdcard
----------------------------

Booting AOSP on DB845c from a mmc sdcard is an experimental build configuration
and is only intended to be used in the LKFT lab. Regular users should not enable
this build flag and should flash and boot from the UFS instead.

Booting from external sdcards will help prevent the internal emmc/ufs wear off
in the long run and extend the lab-life of most of our devboards. To avoid
flashing anything on internal UFS and boot solely from a sdcard, we are
switching to chainloading U-Boot from ABL bootloader. For now we are using a WIP
`upstream u-boot fork <https://source.devboardsforandroid.linaro.org/platform/external/u-boot/+/refs/heads/rbx-integration>`_.

.. note::
   In the long run we plan to switch to AOSP/external/u-boot project to catch up
   with the Android bootloader features.

Set ``TARGET_SDCARD_BOOT=true`` at build time to build and boot AOSP from a mmc
sdcard. This device configuration need atleast 16GB sdcard to boot from. Here are
the instructions to prepare and flash AOSP images on a MMC sdcard:

.. note::
   Following commands that are listed with **=>** prompt means those commands need
   to be run from the u-boot prompt on the device and commands with **$** means
   they need to be run from the HOST machine.

* Boot DB845c in the fastboot mode as mentioned above and erase the boot
  partition to make sure that every reboot/reset lands at the ABL fastboot mode
  prompt.

::

   $ fastboot erase boot

* Build U-Boot for DB845c and boot with it:

::

   $ git clone https://source.devboardsforandroid.linaro.org/platform/external/u-boot -b rbx-integration
   $ cd u-boot
   $ source envsetup.sh
   $ mu qcom_defconfig
   $ budt dragonboard845c
   $ fastboot boot /tmp/u-boot.img   # this will boot U-Boot on DB845c

* Prepare AOSP partition layout on the sdcard from the U-Boot prompt. Make sure
  that a 16GB+ MMC sdcard is plugged into the board:

::

   => run gpt_mmc_aosp
   => reset                          # this will reboot in ABL fastboot mode
   $ fastboot boot /tmp/u-boot.img
   => run fastboot                   # starting U-Boot's fastboot command
   $ fastboot erase boot erase init_boot erase vendor_boot erase modemst1 erase modemst2 erase fsg erase fsc erase misc erase metadata erase super erase userdata
   $ fastboot reboot                 # rebooting in ABL fastboot mode
   $ fastboot boot /tmp/u-boot.img
   => run fastboot

* Build AOSP target db845c-userdebug with MMC sdcard support and flash images on
  the MMC sdcard. Make sure we run U-Boot's fastboot command on the device
  before running the flash commands:

::

   $ make TARGET_SDCARD_BOOT=true -j`nproc`
   $ cd out/target/product/db845c
   $ fastboot flash super ./super.img flash userdata ./userdata.img format:ext4 metadata reboot
   $ fastboot boot ./boot.img

.. note::
   We do not flash **boot.img** on the sdcard; instead, we load the boot image from
   device RAM by running ``fastboot boot boot.img``.


Device Maintainer(s)
********************
- Amit Pundir <pundir at #aosp-developers on OFTC IRC>
