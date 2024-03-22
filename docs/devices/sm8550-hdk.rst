..
 # Copyright (c) 2024, Linaro Ltd.
 #
 # SPDX-License-Identifier: MIT

SM8550-HDK
==========

.. note::
    sm8x50-userdebug is a **WIP** AOSP build target for Snapdragon 8 Gen
    devboards. Primarily supported and tested on sm8550-hdk but also smoke
    tested on sm8550-qrd and sm8650-qrd devboards.


`SM8550-HDK <https://www.lantronix.com/products/snapdragon-8-gen-2-mobile-hardware-development-kit/>`_
is a Snapdragon 8 Gen 2 Mobile Hardware Development Kit.


Getting started with SM8550-HDK
-------------------------------

The power-up sequence for sm8550-hdk is similar to that of RB3 and RB5. The wall
power need to be switched ON first and then connect the USB-C to boot
automatically without pressing any physical button on the board.

Make sure you are running upstream kernel friendly ABL(bootloader) on SM8550-HDK
Latest bootloader binaries are hosted here <URL>.

You can also update bootloader binaries by running **flashall** script, which is
a part of the vendor package of the SM8x50 AOSP build target. Boot the device in
fastboot mode and run the following commands from your HOST machine:

.. code::

   <WIP>


Download and Build Kernel and AOSP images from source for SM8x50 devices
------------------------------------------------------------------------

#. Download the AOSP source tree and prepare sm8x50-userdebug build target:

.. code::

   mkdir $AOSP
   cd $AOSP
   repo init -u https://android.googlesource.com/platform/manifest -b master
   repo sync -j`nproc`
   ./device/linaro/dragonboard/fetch-vendor-package.sh
   cd $AOSP/device/linaro/dragonboard/
   git remote add d4a https://source.devboardsforandroid.linaro.org/device/linaro/dragonboard
   git fetch d4a
   git checkout d4a/d4a
   mkdir $AOSP/device/linaro/dragonboard-kernel/android-upstream


#. Download and build upstream tracking kernel for SM8x50 devices, while we are
   still working on supporting these devices in android-mainline kernel.

.. code::

   git clone https://git.linaro.org/people/amit.pundir/linux -b rbX-mainline
   cd linux
   make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rbX_aosp_defconfig
   make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DTC_FLAGS=-@ -j`nproc`
   make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=./modules/ INSTALL_MOD_STRIP=1 modules_install -j`nproc`
   cp arch/arm64/boot/Image.gz arch/arm64/boot/dts/qcom/qrb5165-rb5.dtb arch/arm64/boot/dts/qcom/sdm845-db845c.dtb arch/arm64/boot/dts/qcom/sm8550-hdk.dtb arch/arm64/boot/dts/qcom/sm8550-qrd.dtb arch/arm64/boot/dts/qcom/sm8650-qrd.dtb $AOSP/device/linaro/dragonboard-kernel/android-upstream/
   find ./modules/lib/ -iname \*.ko -exec cp {} $AOSP/device/linaro/dragonboard-kernel/android-upstream/ \;


#. Build and flash sm8x50-userdebug AOSP images:

.. code::

   cd $AOSP
   source ./build/envsetup.sh
   lunch sm8x50-trunk_staging-userdebug
   make TARGET_KERNEL_USE=upstream -j`nproc`


Reboot sm8x50 in fastboot mode and run following commands to flash and boot AOSP:

Disable verity and and erase AOSP partitions for any stale images:

.. code::

   $AOSP/external/avb/avbtool.py make_vbmeta_image --flag 2 --padding_size 4096 --output ./vbmeta_disabled.img
   fastboot --disable-verity --disable-verification flash vbmeta ./vbmeta_disabled.img
   fastboot erase boot dtbo init_boot vendor_boot
   fastboot reboot bootloader


Flash AOSP images:

.. code::

   cd $AOSP/out/target/product/sm8x50/
   fastboot flash super ./super.img flash userdata ./userdata.img format:ext4 metadata boot ./boot.img


It should boot sm8x50 devices to UI with software rendering support.


Known issues and Troubleshooting on sm8550-hdk
----------------------------------------------

#. UFS is not stable. There are a series of known issues that are currently
   being worked upon.

   For example: Probability of UFS probe running into a hard crash during boot
   time is very high and it needs a power reset to recover.

   And then there is a run time UFS crash which leaves the device unusable.
   v6.9-rc1 kernel will hopefully be more stable.

#. At times `fastboot boot` run into the following failure:

.. code::

   $ fastboot boot ./boot.img
   Sending 'boot.img' (19988 KB)                      OKAY [  0.460s]
   Booting                                            FAILED (remote: 'Failed to load/authenticate boot image: Load Error')
   fastboot: error: Command failed


Run the following set of commands to recover from the above error:

.. code::

   $ fastboot reboot bootloader
   $ fastboot set_active a boot ./boot.img


Device Maintainer(s)
********************
- Amit Pundir <pundir at #aosp-developers on OFTC IRC>
