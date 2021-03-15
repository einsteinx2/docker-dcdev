# Multi-arch building

These are my personal notes on how to build and push these images to Docker Hub with multi-architecture support using Docker's new Buildx system. This assumes Buildx is already installed, as it is by default on macOS. It can also be installed easily on Linux. 

Currently, I'm only building for 2 platforms: x86_64 (`linux/amd64` in Docker) for all modern Intel and AMD machines, and 64bit armv8 aka aarch64 (`linux/arm64` in Docker) for Apple Silicon Macs, Raspberry Pi 3 and 4 (if they're running a 64bit OS), and other modern 64bit ARM single board computers. 

It is trivial to add support for additional platforms like 32bit x86/armv7, for example, by adding `linux/386` and/or `linux/arm/v7` to the `--platform` argument of each buildx command. However, there is significantly increased build time for each emulated platform, so I'm only building for the two most recent and popular ones.

**NOTE: All commands below are run from the root directory of this repo.**


## Custom Builder

NOTE: The default builder has a very conservative log limit of only 1MB, which is very quickly reached while compiling GCC. In order to be able to see all of the log output, it's necessary to create a custom builder with larger limits. This is not required if building with verbose mode off (`--build-arg VERBOSE=false`, which is the default), but is invaluable for debugging image build issues.

1. Create a new builder with 1GB log limits instead of default 1MB (if not already created)
`docker buildx create --driver-opt env.BUILDKIT_STEP_LOG_MAX_SIZE=1073741824 --driver-opt env.BUILDKIT_STEP_LOG_MAX_SPEED=1073741824 --name dcdev --platform linux/amd64,linux/arm64`

2. Confirm the builder was created and start it
`docker buildx inspect dcdev --bootstrap`

## gcc-base (rarely needs update)

1. Set the image version number
`export DCDEV_GCC_BASE_VERSION=2.0.0`

2. Build and push the gcc base image (if it needs updating)
`docker buildx build --builder dcdev --platform linux/amd64,linux/arm64 --push -t einsteinx2/dcdev-gcc-base:v$DCDEV_GCC_BASE_VERSION ./gcc-base`

3. Test the images

4. Update "latest" tag
`docker buildx imagetools create --tag einsteinx2/dcdev-gcc-base:latest einsteinx2/dcdev-gcc-base:v$DCDEV_GCC_BASE_VERSION`

## gcc-toolchain (rarely needs update)

1. Set the image version number
`export DCDEV_GCC_TOOLCHAIN_VERSION=2.0.0`

2. Build and push the GCC toolchain image (if it needs updating)
`docker buildx build --builder dcdev --platform linux/amd64,linux/arm64 --build-arg KOS_GCC_VER=4 --build-arg THREADS=8 --build-arg VERBOSE=true --push -t einsteinx2/dcdev-gcc-toolchain:gcc-4__v$DCDEV_GCC_TOOLCHAIN_VERSION ./gcc-toolchain`
`docker buildx build --builder dcdev --platform linux/amd64,linux/arm64 --build-arg KOS_GCC_VER=9 --build-arg THREADS=8 --build-arg VERBOSE=true --push -t einsteinx2/dcdev-gcc-toolchain:gcc-9__v$DCDEV_GCC_TOOLCHAIN_VERSION ./gcc-toolchain`

3. Test the images

4. Update the GCC major version tags and latest tag (latest points to GCC 9 version)
`docker buildx imagetools create --tag einsteinx2/dcdev-gcc-toolchain:gcc-4 einsteinx2/dcdev-gcc-toolchain:gcc-4__v$DCDEV_GCC_TOOLCHAIN_VERSION`
`docker buildx imagetools create --tag einsteinx2/dcdev-gcc-toolchain:gcc-9 einsteinx2/dcdev-gcc-toolchain:gcc-9__v$DCDEV_GCC_TOOLCHAIN_VERSION`
`docker buildx imagetools create --tag einsteinx2/dcdev-gcc-toolchain:latest einsteinx2/dcdev-gcc-toolchain:gcc-9__v$DCDEV_GCC_TOOLCHAIN_VERSION`


## kos-toolchain (should be frequently updated to latest KOS)

1. Set the image version number
`export DCDEV_KOS_TOOLCHAIN_VERSION=2.0.0`

2. Build and push the KOS toolchain image
`docker buildx build --builder dcdev --platform linux/amd64,linux/arm64 --build-arg KOS_GCC_VER=4 --build-arg THREADS=8 --build-arg VERBOSE=true --push -t einsteinx2/dcdev-kos-toolchain:kos-4__v$DCDEV_KOS_TOOLCHAIN_VERSION ./kos-toolchain`
`docker buildx build --builder dcdev --platform linux/amd64,linux/arm64 --build-arg KOS_GCC_VER=9 --build-arg THREADS=8 --build-arg VERBOSE=true --push -t einsteinx2/dcdev-kos-toolchain:kos-9__v$DCDEV_KOS_TOOLCHAIN_VERSION ./kos-toolchain`

3. Test the images

4. Update the GCC major version tags and latest tag (latest points to GCC 9 version)
`docker buildx imagetools create --tag einsteinx2/dcdev-kos-toolchain:gcc-4 einsteinx2/dcdev-kos-toolchain:gcc-4__v$DCDEV_KOS_TOOLCHAIN_VERSION `
`docker buildx imagetools create --tag einsteinx2/dcdev-kos-toolchain:gcc-9 einsteinx2/dcdev-kos-toolchain:gcc-9__v$DCDEV_KOS_TOOLCHAIN_VERSION `
`docker buildx imagetools create --tag einsteinx2/dcdev-kos-toolchain:latest einsteinx2/dcdev-kos-toolchain:gcc-9__v$DCDEV_KOS_TOOLCHAIN_VERSION `