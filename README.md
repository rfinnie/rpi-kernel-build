# rpi-kernel-build

This program aims to replicate the excellent arm64 Raspberry Pi 4 kernel builds [sakaki-](https://github.com/sakaki-) is doing ([bcm2711-kernel](https://github.com/sakaki-/bcm2711-kernel) and [bcm2711-kernel-bis](https://github.com/sakaki-/bcm2711-kernel-bis)).  It was created so I could compile kernels from raspberrypi's [rpi-5.4.y tree](https://github.com/raspberrypi/linux/tree/rpi-5.4.y), but is currently compatible with either rpi-4.19.y (default) or rpi-5.4.y.  At the moment I am not uploading any built kernels.

## Setup

A clone of [raspberrypi/linux](https://github.com/raspberrypi/linux) is needed, in the ```linux/``` subdirectory by default (configurable).  Beware that a full clone is needed; --depth=1 will not work as raspberrypi will often rebase rather far back.  I currently have a mega centralized linux tree on my build machine, with several remotes, including an "rpi" remote corresponding to the raspberrypi repository.  rpi-kernel-build supports alternative remote and branch names.

You can build directly on an arm64 machine, such as a Raspberry Pi 4 with an arm64 kernel, and kernel builds will take about an hour.  Minimum build environment on a Debian/Ubuntu system should be:

```
sudo apt-get install build-essential flex bison openssl-dev
```

If you want to cross-compile on an x86-64 system, you also want:

```
sudo apt-get install gcc-aarch64-linux-gnu libc6-dev-arm64-cross
```

And then set ```CROSS_COMPILE="aarch64-linux-gnu-"``` in your rpi-kernel-build.rc config.  On an AMD Ryzen 2700X, kernel builds take about 5 minutes.

As an example, here's my full rpi-kernel-build.rc for cross-compilation of rpi-5.4.y:

```
GIT_CHECKOUT="/home/ryan/git/linux"
GIT_REMOTE="rpi"
GIT_BRANCH="rpi-5.4.y"
IMAGE_DIR="/home/ryan/rpi/images"
CROSS_COMPILE="aarch64-linux-gnu-"
JOBS=16
```

## -bis kernels

When run with environment variable ```BIS=1```, rpi-kernel-build produces a kernel with some additional config changes.  At the moment, they mirror sakaki-'s  bcm2711-kernel-bis changes.

## Usage

This program produces tarballs which are compatible with sakaki-'s tarball layouts.  [bcm2711-kernel](https://github.com/sakaki-/bcm2711-kernel)'s README.md has a good usage guide for converting a Raspbian installation to use a 64-bit kernel, using with Gentoo, etc.
