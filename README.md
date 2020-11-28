# rpi-kernel-build

rpi-kernel-build is a script which builds Raspberry Pi kernel tarballs.  It was originally built as a pseudo-fork of [sakaki-](https://github.com/sakaki-)'s kernel builds to build Raspberry Pi 4 64-bit kernels, but is currently capable of building all variants: 32-bit or 64-bit, bcm2709/bcm2711/bcmrpi/bcmrpi3, multiple tracked upstream branches (e.g. rpi-5.4.y, rpi-5.10.y), and feature-enhanced config patches.

## Binary images

There is a [scheduled weekly GitHub workflow](https://github.com/rfinnie/rpi-kernel-build/actions) which builds most combinations.  The following parameters are available:

* Architecture: `arm` (32-bit) or `arm64` (64-bit).  Note that a 64-bit kernel will work with a 32-bit userland provided by Raspberry Pi OS (on 64-bit CPUs that is; Raspberry Pi 3 and later).
* Platform: `bcmrpi` (Raspberry Pi 0/1/2), `bcm2709`/`bcmrpi3` (Raspberry Pi 3), `bcm2711` (Raspberry Pi 4).  Note that the Raspberry Pi 3 platform is called `bcm2709` for `arm` (32-bit) kernels, but `bcmrpi3` for `arm64` (64-bit) kernels, but are otherwise the same platform.
* Branch: `rpi-5.4.y` is the current branch which Raspberry Pi itself builds, and is the latest LTS branch from kernel.org.  As of this writing, `rpi-5.10.y` is the latest overall branch.
* Bis: Same as their non-bis releases, but with a few extra kernel config features enabled. ("Bis" is Latin for second, secondary, etc.)

## Installation

Download the desired tarball, understanding the options above.  For this example, we'll assume `bcm2711` (Raspberry Pi 4), `arm64`, `rpi-5.4-y`, no bis (Raspberry Pi-developer kernel config).

Unfortunately, GitHub actions wrap the build artifacts in a zip file, so it will need to be unzipped to get the tarball:
```
$ unzip images-build-174991421.2-rpi-5.4.y-bcm2711-bis0.zip
```

Then, extract the resulting tarball to the root directory of the Raspberry Pi OS installation:
```
$ sudo tar axvf rpi-kernel-5.4.51-20200717-g9d49ae69a144-arm64-bcm2711.tar.xz -C /
```

Modify `/boot/config.txt`:
```
[all]
# Needed for arm64 kernels; omit for arm kernels
arm_64bit=1
# Adjust architecture and platform based on what was chosen
kernel=kernel.img-arm64-bcm2711
```

Reboot, and hopefully everything has gone correctly.

## Building from source

A clone of [raspberrypi/linux](https://github.com/raspberrypi/linux) is needed, in the ```linux/``` subdirectory by default (configurable).  Beware that a full clone is needed; --depth=1 will not work as raspberrypi will often rebase rather far back.  I currently have a mega centralized linux tree on my build machine, with several remotes, including an "rpi" remote corresponding to the raspberrypi repository.  rpi-kernel-build supports alternative remote and branch names.

You can build directly on an arm64 machine, such as a Raspberry Pi 4 with an arm64 kernel, and kernel builds will take about an hour.  Minimum build environment on a Debian/Ubuntu system should be:

```
sudo apt-get install build-essential flex bison libssl-dev
```

If you want to cross-compile on an x86-64 system, you also want:

```
sudo apt-get install \
  gcc-aarch64-linux-gnu libc6-dev-arm64-cross \
  gcc-arm-linux-gnueabi libc6-dev-armel-cross
```

And then set `CROSS_COMPILE=yes` in your rpi-kernel-build.rc config.  On an AMD Ryzen 2700X, kernel builds take about 5 minutes.  (Normally `CROSS_COMPILE` takes a cross compiler prefix such as `CROSS_COMPILE="aarch64-linux-gnu-"`, but `CROSS_COMPILE=yes` is a special case which sets the prefix based on the target architecture.)

## Sources

If built with `ASSEMBLE_SOURCE=1` (default), a source tarball is also built.  To use, extract it to `/usr/src/`, install the build prerequisites above, then run:

```
make -C /lib/modules/$(uname -r)/build prepare
```

(The kernel tarball's `/lib/modules/$(uname -r)/build` is a symlink to `/usr/src/linux-$(uname -r)`.)

This source tree may be used to build DKMS modules.  An example DKMS module is [hello-dkms](https://github.com/rfinnie/hello-dkms).
