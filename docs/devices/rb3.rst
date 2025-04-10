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
   cd device/linaro/dragonboard
   git remote add d4a https://source.devboardsforandroid.linaro.org/device/linaro/dragonboard
   git fetch d4a; git checkout d4a/d4a
   cd -
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
`upstream u-boot fork <https://source.devboardsforandroid.linaro.org/platform/external/u-boot/+/refs/heads/wip/rbx-integration>`_.

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

   $ git clone https://source.devboardsforandroid.linaro.org/platform/external/u-boot -b wip/rbx-integration
   $ cd u-boot
   $ make CROSS_COMPILE=aarch64-linux-gnu- clean qcom_defconfig all
   $ gzip u-boot-nodtb.bin
   $ mkbootimg --os_version 14.0.0 --os_patch_level 2023-10 --header_version 2 \
      --kernel u-boot-nodtb.bin.gz --dtb dts/upstream/src/arm64/qcom/sdm845-db845c.dtb \
      --pagesize 2048 --cmdline "" --output u-boot.img
   $ fastboot boot u-boot.img                                                               # this will boot U-Boot on RB3


* Prepare AOSP partition layout on the sdcard from the U-Boot prompt. Make sure
  that a 16GB+ MMC sdcard is plugged into the board:

::

   => run gpt_mmc_aosp
   => reset                          # this will reboot in ABL fastboot mode
   $ fastboot boot u-boot.img                                                               # this will boot U-Boot on RB3
   => run fastboot                   # starting U-Boot's fastboot command
   $ fastboot erase boot erase init_boot erase vendor_boot erase modemst1 erase modemst2 erase fsg erase fsc erase misc erase metadata erase super erase userdata
   $ fastboot reboot                 # rebooting in ABL fastboot mode
   $ fastboot boot u-boot.img                                                               # this will boot U-Boot on RB3
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


Flashing and booting AOSP from mmc-sdcard using Generic Bootloader Library (GBL)
--------------------------------------------------------------------------------

.. note::
   This configuration is tested only on RB3, RB5 and SM8550-HDK.
   Booting AOSP from a mmc sdcard using GBL is an experimental build
   configuration and is only intended to be used in the LKFT lab.

Reboot RB3 in fastboot mode and run following commands to disable verity
and erase AOSP partitions for any stale images to make sure that every
reboot/reset lands at the ABL fastboot mode prompt.

.. code::

   $ $AOSP/external/avb/avbtool.py make_vbmeta_image --flag 2 \
     --padding_size 4096 --output ./vbmeta_disabled.img                                     # prepare vbmeta_disabled.img
   $ fastboot --disable-verity --disable-verification flash vbmeta ./vbmeta_disabled.img    # disable verity
   $ fastboot erase boot erase dtbo erase vendor_boot
   $ fastboot reboot bootloader

We chainload U-Boot from ABL bootloader and flash the, already plugged-in,
mmc-sdcard. For RB3, we are using a `upstream u-boot fork with GBL specific changes
<https://source.devboardsforandroid.linaro.org/platform/external/u-boot/+/refs/heads/wip/rbx-integration/>`_.
Here are the instructions to prepare, flash and boot AOSP images from a mmc-sdcard
using GBL. mmc-sdcard boot configuration require atleast 16GB sdcard.

.. note::
   Following commands that are listed with **=>** prompt means those commands need
   to be run from the u-boot prompt on the device and commands with **$** means
   they need to be run from the HOST machine.

* Build U-Boot for RB3 and boot with it:

.. code::

   $ git clone https://source.devboardsforandroid.linaro.org/platform/external/u-boot -b wip/rbx-integration
   $ cd u-boot
   $ make CROSS_COMPILE=aarch64-linux-gnu- clean qcom_defconfig all
   $ gzip u-boot-nodtb.bin
   $ mkbootimg --os_version 14.0.0 --os_patch_level 2023-10 --header_version 2 \
      --kernel u-boot-nodtb.bin.gz --dtb dts/upstream/src/arm64/qcom/sdm845-db845c.dtb \
      --pagesize 2048 --cmdline "" --output u-boot.img
   $ fastboot boot u-boot.img                                                               # this will boot U-Boot on RB3

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
and vendor_boot.img) for RB3 devboards under $AOSP/out/target/product/db845c directory.

* Prepare AOSP partition layout, flash and launch GBL and boot AOSP images on the sdcard.
  Make sure that a 16GB+ MMC sdcard is plugged into the board:

.. code::

   => run gpt_mmc_aosp                                                                      # prepare AOSP style GPT partition layout
                                                                                            # on the mmc-sdcard
   => reset                                                                                 # this will reboot in ABL fastboot mode
   $ fastboot boot u-boot.img                                                               # reboot U-Boot on rb3
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
   $ fastboot boot u-boot.img                                                               # reboot U-Boot on rb3
   => run gbl                                                                               # launch GBL

.. note::
   The above "run gbl" command from U-Boot prompt will launch GBL and it will start
   booting the newly flashed AOSP images to UI with software rendering support,
   unless you press Backspace to interrupt the GBL launch and force it into the
   fastboot mode over USB.


Device Maintainer(s)
********************
- Amit Pundir <pundir at #aosp-developers on OFTC IRC>
