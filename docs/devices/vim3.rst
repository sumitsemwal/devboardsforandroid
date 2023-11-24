..
 # Copyright (c) 2023, BayLibre SAS
 #
 # SPDX-License-Identifier: MIT


VIM3
====

VIM3 is one of the few supported dev-boards listed on
`source.android.com <https://source.android.com/docs/setup/create/devices>`_.

The `VIM3 wiki <https://gitlab.baylibre.com/baylibre/amlogic/atv/aosp/device/amlogic/yukawa/-/wikis/Khadas_VIM3>`_
provides links to the documentation for using these devices.


Board status
************

The VIM3 is build-tested every week:

- CI builds are available at `concourse.baylibre.com <https://concourse.baylibre.com/teams/amlogic/pipelines/>`_
- Prebuilt binaries are available at `binaries.baylibre.com <https://public.amlogic.binaries.baylibre.com/ci/build/yukawa/>`_

These prebuilts are regularly boot-tested to home screen using HDMI.

VIM3 is also used for U-Boot development, to test ``fastboot`` and Android boot flow changes.
The VIM3 board is fully upstream in U-Boot.
Documentation can be found on `U-Boot's documentation <https://docs.u-boot.org/en/latest/board/amlogic/khadas-vim3.html>`_
To build for Android, use ``configs/khadas-vim3_android_defconfig``.

Device Maintainer(s)
********************

VIM3 is co-maintained by Mattijs and Guillaume from BayLibre.

To reach out to them by email:

- Mattijs Korpershoek <mkorpershoek@baylibre.com>
- Guillaume La Roque <glaroque@baylibre.com>

A mailing list for support is also available at aosp@baylibre.com

Mattijs is also available on IRC: ``mkorpershoek`` at ``#aosp-developers`` on OFTC IRC.
