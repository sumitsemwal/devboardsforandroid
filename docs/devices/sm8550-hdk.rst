..
 # Copyright (c) 2024, Linaro Ltd.
 #
 # SPDX-License-Identifier: MIT

SM8550-HDK
==========

.. note::
    sm8x50-userdebug is an AOSP build target for Snapdragon 8 Gen devboards.
    It is supported only on v6.8+ kernel versions. Primarily supported and
    tested on sm8550-hdk. Other devboards (sm8550-qrd and sm8650-qrd) are
    supported on the best efforts basis.


`SM8550-HDK <https://www.lantronix.com/products/snapdragon-8-gen-2-mobile-hardware-development-kit/>`_
is a Snapdragon 8 Gen 2 Mobile Hardware Development Kit.


Getting started with SM8550-HDK
-------------------------------

The power-up sequence for sm8550-hdk is similar to that of RB3 and RB5. The wall
power need to be switched ON first and then connect the USB-C to boot
automatically without pressing any physical button on the board. Make sure that
switch <7, 8> of S5702 is ON before powering ON the device to enable HDMI out.

sm8x50-userdebug boots to UI with v6.8+ android-mainline and upstream kernel
using software rendering and linux-firmware binaries for now.
`lunch` target is not added yet and will be added as soon the firmware binaries
land in linaro-vendor / linux-firmware repo to make sure that the device has
more features enabled and is in a more usable state. Boot image header v2 is
used for now, while we are working on releasing ABL with header v4 support.
Also make sure you are running upstream kernel friendly ABL on SM8550-HDK.


Download and Build Kernel and AOSP images from source for SM8x50 devices
------------------------------------------------------------------------

#. Download and build the AOSP source tree for sm8x50-userdebug build target:

.. code::

   mkdir $AOSP
   cd $AOSP
   repo init -u https://android.googlesource.com/platform/manifest -b master
   repo sync -j`nproc`
   ./device/linaro/dragonboard/fetch-vendor-package.sh
   source ./build/envsetup.sh
   lunch sm8x50-trunk_staging-userdebug
   make -j`nproc`


#. Flash sm8x50-userdebug AOSP images:

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


#. Download and build upstream tracking kernel for SM8x50 devices.

.. code::

   git clone https://git.linaro.org/people/amit.pundir/linux -b rbX-mainline
   cd linux
   make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rbX_aosp_defconfig
   make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DTC_FLAGS=-@ -j`nproc`
   make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=./modules/ INSTALL_MOD_STRIP=1 modules_install -j`nproc`
   cp arch/arm64/boot/Image.gz arch/arm64/boot/dts/qcom/qrb5165-rb5.dtb arch/arm64/boot/dts/qcom/sdm845-db845c.dtb arch/arm64/boot/dts/qcom/sm8550-hdk.dtb arch/arm64/boot/dts/qcom/sm8550-qrd.dtb arch/arm64/boot/dts/qcom/sm8650-qrd.dtb $AOSP/device/linaro/dragonboard-kernel/android-upstream/
   find ./modules/lib/ -iname \*.ko -exec cp {} $AOSP/device/linaro/dragonboard-kernel/android-upstream/ \;


#. Rebuild sm8x50-userdebug AOSP images with local kernel build:

.. code::

   cd $AOSP
   source ./build/envsetup.sh
   lunch sm8x50-trunk_staging-userdebug
   make TARGET_KERNEL_USE=upstream -j`nproc`


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
