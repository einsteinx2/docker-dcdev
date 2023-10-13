# Optimized Dreamcast homebrew development Docker images

## Overview

This repository contains a set of 3 Dockerfiles that can be used to build a fully self-contianed and optimized Dreamcast development environment to cross-compile Dreamcast homebrew games and applications based on either the [KallistiOS aka KOS](https://github.com/KallistiOS/KallistiOS) framework or "bare metal" if required.

All images are based on Debian linux and are split into 3 images for size and build time optimization: `gcc-base`, `gcc-toolchain`, and `kos-toolchain`. 

If you only need the SH4 and ARM cross-compilers to use for "bare metal" programming or as a base for your own KOS images, then the only image you need is `einsteinx2/dcdev-gcc-toolchain`. 

If you just want a complete ready to go KOS environment, the only image you need to pull is `einsteinx2/dcdev-kos-toolchain`, though to build that image, the other two are required.

## Quick Start

### Example GCC 13 Usage (run from your project directory)

#### Simple Makefile
`docker run -it --rm -v $PWD:/src ghcr.io/einsteinx2/dcdev-kos-toolchain:gcc-stable make`
#### Build Script
`docker run -it --rm -v $PWD:/src ghcr.io/einsteinx2/dcdev-kos-toolchain:gcc-stable ./dc_build.sh`

#### Interactive Shell
`docker run -it --rm -v $PWD:/src ghcr.io/einsteinx2/dcdev-kos-toolchain:gcc-stable`

### Example GCC 4 Usage (run from your project directory)

*NOTE: It is highly recommended to use the latest GCC toolchain above, but this legacy toolchain is provided if needed for compatibility with older projects.*

#### Simple Makefile
`docker run -it --rm -v $PWD:/src ghcr.io/einsteinx2/dcdev-kos-toolchain:gcc-legacy make`
#### Build Script
`docker run -it --rm -v $PWD:/src ghcr.io/einsteinx2/dcdev-kos-toolchain:gcc-legacy ./dc_build.sh`
#### Interactive Shell
`docker run -it --rm -v $PWD:/src ghcr.io/einsteinx2/dcdev-kos-toolchain:gcc-legacy`

These will mount the current folder to `/src` in the container and run whatever command is passed inside a bash shell with all KOS environment variables already set. When passing no command, an interactive bash shell with all KOS variables set is provided if preferred.

You may execute other commands, of course. There are various utilities included in the image in the `/opt/toolchains/dc/bin` directory that is in the `$PATH` that can do various things like create a IP.BIN file, scramble/unscramble, convert audio and video files to the Dreamcast's format, etc. These commands can be run by simply appending the command to the end of the `docker run` command of the image you want to use.

NOTE: These image probably will not compile Dreamshell as that requires a specially patched KOS install. Eventually I'll create images based on these that include the Dreamshell patches.

## Images Overview

All images are optimized using multi-stage builds to keep the final image size and number of layers to a minimum, and only include the minimum set of required tools to have a fully functional development environment.

### einsteinx2/dcdev-gcc-base

A stock Debian Linux image that contains a regular x86_64/ARM64 GCC compiler toolchain and all necessary tools like make, sed, git, etc that are required to compile the SH4 and ARM cross-compilers as well as the KOS framework.

### einsteinx2/dcdev-gcc-toolchain

A stock Debian Linux image that contains only the SH4 and ARM cross-compiler binaries based on various versions of GCC depending on the image tag, including the SH4 GDB debugger binary, but nothing else. It can be used to directly compile SH4 and ARM code for the Dreamcast that is not based on the KOS framework, such as projects coding "bare metal" using something like [DreamHAL](https://github.com/sega-dreamcast/dreamhal).

### einsteinx2/dcdev-kos-toolchain

A stock Debian Linux image that contains the SH4 and ARM cross-compiler binaries based on various versions of GCC depending on the image tag, the full KOS framework binaries and source code, all KOS-PORTS libraries for things like mp3, ogg, opus, png, jpeg, and SDL 1.2 support, all KOS included utilities and addons, and some external utilities.

These utiltiies include things like `bin2c` and `bin2o` to include binary files in your code as C headers or linkable objects; and `genromfs` to generate KOS romdisks; `scramble`, `makeip`, and `cdi4dc` to create `CDI` disc images from your homebrew software. All of these utilities are located in the `$PATH` so can be used by simply appending the command to the `docker run` command.

Also preinstalled are various `apt` packages that are useful for development such as `cdrecord`, `cmake`, `curl`, `dos2unix`, `gawk`, `make`, `mkisofs`, `python3`, `sed`, `tree`, `unix2dos`, `wget`, etc.

However, it does not include the standard x86_64/ARM64 GCC tools as they are not required for Dreamcast homebrew compilation and only increase the image size.

## Build Instructions

### einsteinx2/dcdev-gcc-base
`docker build -f gcc-base/Dockerfile -t einsteinx2/dcdev-gcc-base:latest .`

### einsteinx2/dcdev-gcc-toolchain

#### GCC 13.2.0
`docker build -f gcc-toolchain/Dockerfile -t einsteinx2/dcdev-gcc-toolchain:stable .`
#### GCC 4.7.4
`docker build --build-arg KOS_GCC_VER=4 -f gcc-toolchain/Dockerfile -t einsteinx2/dcdev-gcc-toolchain:legacy .`

### einsteinx2/dcdev-kos-toolchain

#### GCC 13.2.0
`docker build -f kos-toolchain/Dockerfile -t einsteinx2/dcdev-kos-toolchain:gcc-stable .`
#### GCC 4.7.4
`docker build -f kos-toolchain/Dockerfile --build-arg KOS_GCC_VER=4 -t einsteinx2/dcdev-kos-toolchain:gcc-legacy .`

### Additional Build Arguments (only for `gcc-toolchain` and `kos-toolchain`)

There are also a variety of other build arguments you can use when building these images. To see the full details, refer to the comments at the top of each Dockerfile, but here are some of the most useful ones.

#### VERBOSE

Defaults to verbose output off as it should build faster without all of the console printing and is easier to follow the build progress.  
To debug build problems, add the flag `--build-arg VERBOSE=true` when building the image to have it print all output from each download and compilation step.

### THREADS

For maximum speed, set this to the number of CPU threads you have.  
When THREADS=0, `$(getconf _NPROCESSORS_ONLN)` will be used to automatically build using the max available threads
i.e. if you have a 4 core / 8 thread CPU, choose 8 (can be set using `--build-arg THREADS=8` when building)

### KOS_GCC_VER

Choose GCC major version. Supported values are stable (SH4 13.2.0/ARM 8.5.0), legacy (both 4.7.4).  
Can be set using `--build-arg KOS_GCC_VER=legacy` when building, default is stable (legacy is not recommended for use in new projects).

### KOS\_REPO, KOS\_BRANCH, PORTS\_REPO, PORTS\_BRANCH

These allow you to specify a git repository URL and branch to use for KOS and/or KOS-PORTS if you do not want to use the current master branch from the upstream KOS repositories, for example if you'd like to build your own fork or work-in-progress branch.

## Author

These images are developed and maintained by [Ben Baron aka einsteinx2](https://github.com/einsteinx2) based on the cross-compiler build scripts and framework from KallistiOS by [Cryptic Allusion (Dan Potter and Lawrence Sebald aka BlueCrab)](http://gamedev.allusion.net/softprj/kos).
