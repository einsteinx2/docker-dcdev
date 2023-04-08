# Optimized Dreamcast homebrew development Docker images

## Overview
This repository contains a set of 3 Dockerfiles that can be used to build a fully self-contained and optimized Dreamcast development environment to cross-compile Dreamcast homebrew games and applications based on either the [KallistiOS aka KOS](https://github.com/KallistiOS/KallistiOS) framework or "bare metal" if required.

All images are based on Alpine linux  and they are split into 3 images for size and build time optimization: `gcc-base`, `gcc-toolchain`, and `kos-toolchain`. 

If you only need the SH4 and ARM cross-compilers to use for "bare metal" programming or as a base for your own KOS images, then the only image you need is `ghcr.io/octoate/dcdev-gcc-toolchain`. 

If you just want a complete ready to go KOS environment, the only image you need is `ghcr.io/octoate/dcdev-kos-toolchain`, though to build that image, the other two are required.

## Quick Start

### Example GCC 12 Usage (run from your project directory):
**Simple Makefile:**   `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-12 make`<br/>
**Build Script:**      `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-12 ./dc_build.sh`<br/>
**Interactive Shell:** `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-12`


### Example GCC 9 Usage (run from your project directory):
**Simple Makefile:**   `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-9 make`<br/>
**Build Script:**      `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-9 ./dc_build.sh`<br/>
**Interactive Shell:** `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-9`

### Example GCC 4 Usage (run from your project directory):
**Simple Makefile:**   `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-4 make`<br/>
**Build Script:**      `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-4 ./dc_build.sh`<br/>
**Interactive Shell:** `docker run -it --rm -v $PWD:/src ghcr.io/octoate/dcdev-kos-toolchain:gcc-4`

These will mount the current folder to `/src` in the container and run whatever command is passed inside a bash shell with all KOS environment variables already set. When passing no command, an interactive bash shell with all KOS variables set is provided if preferred.

You may execute other commands, of course. There are various utilities included in the image in the `/opt/toolchains/dc/bin` directory that is in the `$PATH` that can do various things like create a IP.BIN file, scramble/unscramble, convert audio and video files to the Dreamcast's format, etc. These commands can be run by simply appending the command to the end of the `docker run` command of the image you want to use.

Note: These image probably will not compile Dreamshell as that requires a specially patched KOS install. Eventually I'll create images based on these that include the Dreamshell patches.

## Images Overview

All images are optimized using multi-stage builds to keep the final image size and number of layers to a minimum, and only include the minimum set of required tools to have a fully functional development environment.

### ghcr.io/octoate/dcdev-gcc-base
A stock Alpine Linux image that contains a regular x86_64 GCC compiler toolchain and all necessary tools like make, sed, git, etc that are required to compile the SH4 and ARM cross-compilers as well as the KOS framework.

### ghcr.io/octoate/dcdev-gcc-toolchain
A stock Alpine Linux image that contains only the SH4 and ARM cross-compiler binaries based on various versions of GCC depending on the image tag, including the SH4 GDB debugger binary, but nothing else. It can be used to directly compile SH4 and ARM code for the Dreamcast that is not based on the KOS framework, such as projects coding "bare metal" using something like [DreamHAL](https://github.com/sega-dreamcast/dreamhal).

### ghcr.io/octoate/dcdev-kos-toolchain
A stock Alpine Linux image that contains the SH4 and ARM cross-compiler binaries based on various versions of GCC depending on the image tag, the full KOS framework binaries and source code, all KOS-PORTS libraries for things like mp3, ogg, opus, png, jpeg, and SDL 1.2 support, all KOS included utilities and addons, and some external utilities.

These utilities include things like `bin2c` and `bin2o` to include binary files in your code as C headers or linkable objects; and `genromfs` to generate KOS romdisks; `scramble`, `makeip`, and `cdi4dc` to create `CDI` disc images from your homebrew software. All of these utilities are located in the `$PATH` so can be used by simply appending the command to the `docker run` command.

Also preinstalled are various `apk` packages that are useful for development such as `cdrecord`, `cmake`, `colordiff`, `curl`, `dos2unix`, `gawk`, `git`, `make`, `mkisofs`, `perl`, `python2.7`, `sed`, `svn`, `tree`, `unix2dos`, `wget`, `vim`, etc.

However, it does not include the standard x86_64 GCC tools as they are not required for Dreamcast homebrew compilation and only increase the image size.

## Build Instructions

### ghcr.io/octoate/dcdev-gcc-base
`docker build -f gcc-base/Dockerfile -t ghcr.io/octoate/dcdev-gcc-base:latest .`

### ghcr.io/octoate/dcdev-gcc-toolchain
**GCC 12:** `docker build -f gcc-toolchain/Dockerfile -t ghcr.io/octoate/dcdev-gcc-toolchain:gcc-12 .`<br/>
**GCC 9:** `docker build -f gcc-toolchain/Dockerfile -t ghcr.io/octoate/dcdev-gcc-toolchain:gcc-9 .`<br/>
**GCC 4:** `docker build --build-arg KOS_GCC_VER=4 -f gcc-toolchain/Dockerfile -t ghcr.io/octoate/dcdev-gcc-toolchain:gcc-4 .`

### ghcr.io/octoate/dcdev-kos-toolchain
**GCC 12:** `docker build -f kos-toolchain/Dockerfile -t ghcr.io/octoate/dcdev-kos-toolchain:gcc-12 .`<br/>
**GCC 9:** `docker build -f kos-toolchain/Dockerfile -t ghcr.io/octoate/dcdev-kos-toolchain:gcc-9 .`<br/>
**GCC 4:** `docker build --build-arg KOS_GCC_VER=4 -f kos-toolchain/Dockerfile -t ghcr.io/octoate/dcdev-kos-toolchain:gcc-4 .`

### Additional Build Arguments (only for `gcc-toolchain` and `kos-toolchain`)
There are also a variety of other build arguments you can use when building these images. To see the full details, refer to the comments at the top of each Dockerfile, but here are some of the most useful ones.

#### ARG VERBOSE=false
Defaults to verbose output off as it should build faster without all of the console printing and is easier to follow the build progress.<br/>
To debug build problems, add the flag `--build-arg VERBOSE=true` when building the image to have it print all output from each download and compilation step.

### ARG THREADS=0
For maximum speed, set this to the number of CPU threads you have.<br/>
When THREADS=0, `$(getconf _NPROCESSORS_ONLN)` will be used to automatically build using the max available threads
i.e. if you have a 4 core / 8 thread CPU, choose 8 (can be set using `--build-arg THREADS=8` when building)

### ARG KOS_GCC_VER=9
Choose GCC major version. Supported values are 9 (SH4 9.3/ARM 8.4), 4 (both 4.7.4).<br/>
Can be set using `--build-arg KOS_GCC_VER=4` when building, default is 9.

### ARG KOS\_REPO, KOS\_BRANCH, PORTS\_REPO, PORTS\_BRANCH
These allow you to specify a git repository URL and branch to use for KOS and/or KOS-PORTS if you do not want to use the current master branch from the upstream KOS repositories, for example if you'd like to build your own fork or work-in-progress branch.

## Author

These images were developed and by [Ben Baron aka einsteinx2](https://github.com/einstein2x) based on the cross-compiler build scripts and framework from KallistiOS by [Cryptic Allusion (Dan Potter and Lawrence Sebald aka BlueCrab)](http://gamedev.allusion.net/softprj/kos).
They are currently maintained by [Tim Riemann aka Octoate](https://github.com/octoate).
