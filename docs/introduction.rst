..
 # Copyright (c) 2023, Linaro Ltd.
 #
 # SPDX-License-identifier: MIT

############
Introduction
############

There are many dev-boards available in the market, and a great number of
interested folks that would like to have Android Open Source Project (AOSP)
running on these devices, for various reasons.

The **dev-boards for Android** community initiative is to enable a collaborative
space for developers desirous of keeping AOSP running with relevant kernels. 
The dev-boards can then be used for new feature development for Android,
testing and validating them with Android tests like CTS and VTS.
They can also be instrumental in sharing and co-developing features like
HALs across multiple devices.

We would aim for the dev-boards to boot to UI with AOSP, be testable and usable
as development platforms.

It would be great to have device-owner(s) for each device that is enabled in
this collaboration.

In the future, we would like to enable as many dev-boards as possible with CI
via LAVA, which can be then used to track board status and quality on a rolling
basis.

Goals
*****
- Be an umbrella project for collaboration on dev-boards for Android.
- Help foster a healthy community around AOSP.
- Provide a space for feature-sharing across devices.
- Enable CI testing via LAVA, increasing level of confidence in these devices.

Software Components
*******************
For each supported device, links are made available to the kernel source, local
manifests, device specific files and binaries (like bootloader, firmware, HALs)
and documentation.


Community Maintainer(s)
***********************

- Sumit Semwal <sumit.semwal@linaro.org>

