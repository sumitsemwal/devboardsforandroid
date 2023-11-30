..
 # Copyright (c) 2023, Linaro Ltd.
 #
 # SPDX-License-Identifier: MIT


Hikey960
========

Hikey960 until recently was a devboard listed on the AOSP reference boards
page. It is now in maintenance phase mostly used to test legacy builds. No
further development activity is planned for this devboard.

We are hosting it here as an example of how the devboardsforandroid
infrastructure can be used to support devboards.

More details about the devboard can be found at the `96boards page 
<https://www.96boards.org/product/hikey960/>`_.

- `Documentation <https://www.96boards.org/documentation/consumer/hikey/hikey960/hardware-docs/hardware-user-manual.md.html>`_
- Kernel source code - will be shared soon

**Artifacts hosted here**

- `device config <https://source.devboardsforandroid.linaro.org/device/linaro/hikey960/>`_
    - This also includes vendor binaries for Hikey960.
- `prebuilt kernels <https://source.devboardsforandroid.linaro.org/device/linaro/hikey960-kernel/>`_
- `Local manifest <https://source.devboardsforandroid.linaro.org/platform/manifest/>`_

Build Instructions

::

$ repo init -u https://android.googlesource.com/platform/manifest -b main
$ git clone https://gerrit.devboardsforandroid.linaro.org/platform/manifest .repo/local_manifests/
$ repo sync -j`nproc`
$ source build/envsetup.sh
$ lunch linaro_hikey960-userdebug
$ make -j$(nproc)

**Artifacts hosted at AOSP (available as of 24 Oct, 2023)**

- `device config at AOSP <https://android.googlesource.com/device/linaro/hikey/>`_
- `prebuilt kernels at AOSP <https://android.googlesource.com/device/linaro/hikey-kernel/>`_
- `Vendor binary blobs <http://releases.devboardsforandroid.linaro.org/vendor-packages>`_
    - This is required as a binary package when using the AOSP based artifacts.

Build instructions

::

$ repo init -u https://android.googlesource.com/platform/manifest -b main
$ repo sync -j`nproc`
$ ./device/linaro/hikey/fetch-vendor-package.sh
$ source build/envsetup.sh
$ lunch hikey960-userdebug
$ make -j`nproc`


Device Maintainer(s)
********************
- Yongqin Liu - <liuyq at #aosp-developers on OFTC IRC>
