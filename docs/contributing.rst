..
 # Copyright (c) 2023, Linaro Ltd.
 #
 # SPDX-License-identifier: MIT

############
Contributing
############

For familiarity and uniformity with the AOSP development process, Gerrit-based
review and merge process is followed for dev-boards that have their source code
hosted here.

To submit contributions for projects hosted here on devboardsforandroid, please
use
`devboardsforandroid Gerrit server <https://gerrit.devboardsforandroid.linaro.org/>`_.

For communities around existing devices that already have a process for their
contributions, reviewing and merging, we can continue to follow the same.

Requirements for adding Devboards
=================================

We are excited to add Devboards here, but it is important to put some
rules for adding and keeping the Devboards in sync.

#. Devboard should have active maintainer(s) that are responsible for them.
#. AOSP/main should be built and boot tested to UI with the Devboard within
   a 30 day cycle.
#. Mandatory availability of a mechanism that shows the health of the Devboard.
   It could be CI loop, or something similar.
#. Source code should be available - same as would be expected if the Devboard
   was in AOSP.
#. Device deprecation: If there is no update or the Devboard is broken with
   AOSP/main for > 3 months, it would be deprecated. Orphaned Devboards are
   not useful for the community.

Interactions
============
We have a couple of IRC channels on OFTC

- **#linaro-android** to interact with the team at Linaro that works on Android
- **#aosp-developers** as a community go-to place

Google Groups
=============

A few Google groups around various topics of interest related to AOSP are
also a good source of information and troubleshooting:

- `Building AOSP <https://groups.google.com/g/android-building>`_
- `AOSP Internals <https://groups.google.com/g/android-platform>`_
- `Porting AOSP <https://groups.google.com/g/android-porting>`_
- `Contributing to AOSP <https://groups.google.com/g/android-contrib>`_
