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

.. note::
   You can also update bootloader binaries by running flashall script, which is
   a part of the vendor package of RB5 AOSP build target. Boot in fastboot mode
   and run following command from your HOST machine:

.. code::

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

#. Download the AOSP source tree and build db845c-trunk_staging-userdebug build target:

::

   repo init -u https://android.googlesource.com/platform/manifest -b master
   repo sync -j`nproc`
   ./device/linaro/dragonboard/fetch-vendor-package.sh
   cd device/linaro/dragonboard
   git remote add d4a https://source.devboardsforandroid.linaro.org/device/linaro/dragonboard
   git fetch d4a; git checkout d4a/d4a
   cd -
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


Flashing and booting AOSP from mmc-sdcard using Generic Bootloader Library (GBL)
--------------------------------------------------------------------------------

.. note::
   This configuration is tested only on RB3, RB5 and SM8550-HDK.
   Booting AOSP from a mmc sdcard using GBL is an experimental build
   configuration and is only intended to be used in the LKFT lab.

Reboot RB5 in fastboot mode and run following commands to disable verity
and erase AOSP partitions for any stale images to make sure that every
reboot/reset lands at the ABL fastboot mode prompt.

.. code::

   $ $AOSP/external/avb/avbtool.py make_vbmeta_image --flag 2 \
     --padding_size 4096 --output ./vbmeta_disabled.img                                     # prepare vbmeta_disabled.img
   $ fastboot --disable-verity --disable-verification flash vbmeta ./vbmeta_disabled.img    # disable verity
   $ fastboot erase boot erase dtbo erase vendor_boot
   $ fastboot reboot bootloader

We chainload U-Boot from ABL bootloader and flash the, already plugged-in,
mmc-sdcard. For RB5, we are using a `upstream u-boot fork with GBL specific changes
<https://source.devboardsforandroid.linaro.org/platform/external/u-boot/+/refs/heads/wip/rbx-integration/>`_.
Here are the instructions to prepare, flash and boot AOSP images from a mmc-sdcard
using GBL. mmc-sdcard boot configuration require atleast 16GB sdcard.

.. note::
   Following commands that are listed with **=>** prompt means those commands need
   to be run from the u-boot prompt on the device and commands with **$** means
   they need to be run from the HOST machine.

* Build U-Boot for RB5 and boot with it:

.. code::

   $ git clone https://source.devboardsforandroid.linaro.org/platform/external/u-boot -b wip/rbx-integration
   $ cd u-boot
   $ make CROSS_COMPILE=aarch64-linux-gnu- clean qcom_defconfig all
   $ gzip u-boot-nodtb.bin
   $ mkbootimg --os_version 14.0.0 --os_patch_level 2023-10 --header_version 2 \
      --kernel u-boot-nodtb.bin.gz --dtb dts/upstream/src/arm64/qcom/qrb5165-rb5.dtb \
      --pagesize 2048 --cmdline "" --output u-boot.img
   $ fastboot boot u-boot.img                                                               # this will boot U-Boot on RB5

* Build GBL efi app and mcopy it over to gbl.img:

.. code::

   $ repo init -u https://android.googlesource.com/kernel/manifest -b uefi-gbl-mainline
   $ repo sync -j`nproc`
   $ ./tools/bazel run //bootable/libbootloader:gbl_efi_dist \
     --extra_toolchains=@gbl//toolchain:all --sandbox_debug --verbose_failures
   $ dd if=/dev/zero of=gbl.img bs=1048576 count=2
   $ mkfs.vfat gbl.img
   $ mcopy -i gbl.img out/gbl_efi/gbl_aarch64.efi ::gbl_aarch64.efi

* Build AOSP images with GBL support (boot image header v4 support with init_boot):

.. code::

   $ mkdir $AOSP
   $ cd $AOSP
   $ repo init -u https://android.googlesource.com/platform/manifest -b master
   $ repo sync -j`nproc`
   $ ./device/linaro/dragonboard/fetch-vendor-package.sh
   $ cd device/linaro/dragonboard
   $ git remote add d4a https://source.devboardsforandroid.linaro.org/device/linaro/dragonboard
   $ git fetch d4a; git checkout d4a/d4a
   $ cd $AOSP
   $ source ./build/envsetup.sh
   $ lunch db845c-trunk_staging-userdebug
   $ make TARGET_SDCARD_BOOT=true TARGET_USES_GBL=true -j`nproc`

The above instructions will build AOSP images (boot.img, init_boot.img, super.img, userdata.img
and vendor_boot.img) for RB5 devboards under $AOSP/out/target/product/db845c directory.

* Prepare AOSP partition layout, flash and launch GBL and boot AOSP images on the sdcard.
  Make sure that a 16GB+ MMC sdcard is plugged into the board:

.. code::

   => run gpt_mmc_aosp                                                                      # prepare AOSP style GPT partition layout
                                                                                            # on the mmc-sdcard
   => reset                                                                                 # this will reboot in ABL fastboot mode
   $ fastboot boot u-boot.img                                                               # reboot U-Boot on rb5
   => run fastboot                                                                          # starting U-Boot's fastboot command
   $ fastboot erase gbl erase boot_a erase boot_b erase init_boot_a \
     erase init_boot_b erase vendor_boot_a erase vendor_boot_b \
     erase vbmeta_a erase vbmeta_b erase modemst1 erase modemst2 \
     erase fsg erase fsc erase misc erase metadata erase super erase userdata               # erase mmc-sdcard partitions
   $ fastboot flash gbl ./gbl.img flash boot ./boot.img \
     flash init_boot ./init_boot.img flash vendor_boot ./vendor_boot.img \
     flash super ./super.img flash userdata ./userdata.img format:ext4 metadata             # flash GBL and AOSP images
   $ fastboot --disable-verity --disable-verification flash vbmeta_a ./vbmeta_disabled.img  # disable verity
   $ fastboot --disable-verity --disable-verification flash vbmeta_b ./vbmeta_disabled.img  # disable verity on _b slot as well just in case
   $ fastboot reboot
   $ fastboot boot u-boot.img                                                               # reboot U-Boot on rb5
   => run gbl                                                                               # launch GBL

.. note::
   The above "run gbl" command from U-Boot prompt will launch GBL and it will start
   booting the newly flashed AOSP images to UI, unless you press Backspace to
   interrupt the GBL launch and force it into the fastboot mode over USB.


ToDo / Known Issues
-------------------

#. Fastboot over U-Boot takes about 30seconds to initialize on the HOST machine
   after running "run fastboot" command on the U-Boot shell.

#. Factory version of RB5 comes with older version of lt9611uxc firmware flashed
   on it, so in case you do not see display up and running then please upgrade
   the lt9611uxc firmware version to v43 or newer.


Device Maintainer(s)
********************
- Amit Pundir <pundir at #aosp-developers on OFTC IRC>
