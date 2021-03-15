# -------------------------------------------------------------
# KOS Complete Toolchain Image (einsteinx2/dcdev-kos-toolchain)
# -------------------------------------------------------------
#
# This image is meant to be used as a fully featured KOS development environment. It includes both the SH4 and
# ARM cross-compiler toolchains, the full KallistiOS library, all included utilities and addons, and some extra
# useful utilities and programs that you might need to compile any Dreamcast project. It's based on Alpine to keep
# the size to a minimum, though convenience is preferred over a slightly larger image, hence the preinstalled tools
# and the inclusion of the KOS and KOS-PORTS source code rather than the compiled binaries and headers only.
# 
# This image contains an entry scipt that automatically loads the KOS environment variables, so it can compile a
# project or use any of the included tools without entering the interactive shell. If no command is provided, an
# interactive bash shell is automatically launched.
#
# Build Commands:
# ---------------
# GCC 9 Toolchain: `docker build -t einsteinx2/dcdev-kos-toolchain:gcc-9 .`
# GCC 4 Toolchain: `docker build --build-arg KOS_GCC_VER=4 -t einsteinx2/dcdev-kos-toolchain:gcc-4 .`
#
# Example GCC 9 Usage (run from your project directory):
# --------------
# Simple Makefile:   `docker run -it --rm -v $PWD:/src einsteinx2/dcdev-kos-toolchain:gcc-9 make`
# Build Script:      `docker run -it --rm -v $PWD:/src einsteinx2/dcdev-kos-toolchain:gcc-9 ./dc_build.sh`
# Interactive Shell: `docker run -it --rm -v $PWD:/src einsteinx2/dcdev-kos-toolchain:gcc-9`
#
# Example GCC 4 Usage (run from your project directory):
# --------------
# Simple Makefile:   `docker run -it --rm -v $PWD:/src einsteinx2/dcdev-kos-toolchain:gcc-4 make`
# Build Script:      `docker run -it --rm -v $PWD:/src einsteinx2/dcdev-kos-toolchain:gcc-4 ./dc_build.sh`
# Interactive Shell: `docker run -it --rm -v $PWD:/src einsteinx2/dcdev-kos-toolchain:gcc-4`
#
# Notes
# -----
# More specific tags are provided so you can pin your builds to an exact version of this image if needed. For
# example, you can specify GCC 9.3 by using the einsteinx2/dcdev-kos-toolchain:gcc-9.3 image, or you can specify 
# a certain GCC 9.3 image by using a version tag like the einsteinx2/dcdev-kos-toolchain:gcc-9.3__v1.0.0 image.
# This is mostly because while the GCC toolchains don't change very often and are usually compatible as long as
# you're using the same GCC major version (e.g. 9.2 and 9.3), however the KOS library changes more frequently 
# and may prevent your project from compiling on a newer version without making some (usually minor) changes.


#
# Global Arguments
#

# Defaults to verbose output off as it should build faster without all of the console printing.
# To debug build problems, add the flag `--build-arg VERBOSE=true` when building the image to 
# have it print all output from each download and compilation step.
# NOTE: If you are using the latest Docker versions with buildx, you'll also need to increase
#       the log limit (https://github.com/docker/buildx/issues/484) and add optionally include 
#       the flag `--progress=plain` if you want to see the full output.
ARG VERBOSE=false

# For maximum speed, set this to the number of CPU threads you have
# When THREADS=0, `$(getconf _NPROCESSORS_ONLN)` will be used to automatically build using the max available threads
# i.e. if you have a 4 core / 8 thread CPU, choose 8 (can be set using `--build-arg THREADS=8` when building)
ARG THREADS=0

# KOS git repository and branch to use
# NOTE: To use a different repo or branch, for example to use your own fork instead of upstream,
#       add the flag `--build-arg KOS_REPO="https://github.com/your_username/KallistiOS.git"` and/or
#       add the flag `--build-arg KOS_BRANCH="your_branch"
ARG KOS_REPO="https://github.com/KallistiOS/KallistiOS.git"
ARG KOS_BRANCH="master"

# KOS-PORTS git repository and branch to use
# NOTE: To use a different repo or branch, for example to use your own fork instead of upstream,
#       add the flag `--build-arg PORTS_REPO="https://github.com/your_username/kos-ports.git"` and/or
#       add the flag `--build-arg PORTS_BRANCH="your_branch"
ARG PORTS_REPO="https://github.com/KallistiOS/kos-ports.git"
ARG PORTS_BRANCH="master"

# Choose GCC major version. Supported values are 9 (SH4 9.3/ARM 8.4), 4 (both 4.7.4)
# Can be set using `--build-arg KOS_GCC_VER=4` when building, default is 9)
# NOTE: You can technically use any available tag number from the `einsteinx2/dcdev-gcc-toolchain` image here
ARG KOS_GCC_VER=9

#
# First stage: Pre-built GCC cross-compiler toolchain to build KOS 
#

FROM einsteinx2/dcdev-gcc-toolchain:gcc-$KOS_GCC_VER as toolchain
FROM einsteinx2/dcdev-gcc-base:latest as builder
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"

# Set build arguments
ARG VERBOSE
ARG THREADS
ARG KOS_REPO
ARG KOS_BRANCH
ARG PORTS_REPO
ARG PORTS_BRANCH

# Set environment variables
ENV DCTOOLCHAIN="/opt/toolchains/dc"
ENV KOS="$DCTOOLCHAIN/kos"
ENV PORTS="$DCTOOLCHAIN/kos-ports"

# Use the bash shell instead of sh for the build process
SHELL ["/bin/bash", "-c"]

# Copy the prebuilt GCC toolchain
COPY --from=toolchain $DCTOOLCHAIN/sh-elf $DCTOOLCHAIN/sh-elf
COPY --from=toolchain $DCTOOLCHAIN/arm-eabi $DCTOOLCHAIN/arm-eabi

# Validate THREADS argument and set build variables
RUN if [ "$THREADS" -eq 0 ] 2>/dev/null; then \
        # If THREADS is 0, default to max possible threads
        printf "export DOCKER_THREADS=$(getconf _NPROCESSORS_ONLN)\n" >> /tmp/dockerenv.sh; \
    elif ! [[ -z "${THREADS//[0-9]}" ]]; then \
        # If THREADS is a negative number or any non-integer, print an error and exit
        printf "Invalid THREADS specified, please enter a positive integer value\n"; \
        exit 1; \
    else \
        # If THREADS is a positive integer, use that value
        printf "export DOCKER_THREADS=$THREADS\n" >> /tmp/dockerenv.sh; \
    fi \
    && . /tmp/dockerenv.sh \
    && printf "Building using $DOCKER_THREADS threads\n"

# Build KOS and patched GCC cross-compiler toolchain for SH4 (Main CPU) and ARM7DI (AICA Control CPU)
RUN \
    # Clone repositories
    . /tmp/dockerenv.sh \
    && printf "Cloning latest mainline KOS...$NL" \
    && git clone --single-branch --branch $KOS_BRANCH $KOS_REPO $KOS &> $REDIRECT \
    && printf "Done.\nCloning latest mainline KOS PORTS...$NL" \
    && git clone --single-branch --branch $PORTS_BRANCH $PORTS_REPO $PORTS &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Configure environment
    . /tmp/dockerenv.sh \
    && printf "Configuring environment...$NL" \
    && cp $KOS/doc/environ.sh.sample $KOS/environ.sh \
    # NOTE: This is in single quotes on purpose so that ${KOS_BASE} is not evaluated
    && printf '. ${KOS_BASE}/environ_dreamcast.sh' >> $KOS/environ.sh \
    # NOTE: This is in double quotes on purpose so that $KOS is evaluated
    && printf ". /etc/profile\n. $KOS/environ.sh\n" > /root/.bashrc \
    && printf "Done.\n"
RUN \
    # Build KOS
    . /tmp/dockerenv.sh && . $KOS/environ.sh \
    && printf "Building KOS...$NL" \
    && make -j$DOCKER_THREADS -C $KOS &> $REDIRECT \
    && printf "Done.\nCleaning KOS...$NL" \
    && make -j$DOCKER_THREADS -C $KOS clean &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Build KOS Ports
    . /tmp/dockerenv.sh && . $KOS/environ.sh \
    && printf "Building KOS Ports...$NL" \
    && make -C $KOS kos-ports_all &> $REDIRECT \
    && printf "Done.\nCleaning KOS Ports...$NL" \
    && make -C $KOS kos-ports_clean &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Build KOS addons
    . /tmp/dockerenv.sh && . $KOS/environ.sh \
    && printf "Building KOS addons...$NL" \
    && make -j$DOCKER_THREADS -C $KOS/addons &> $REDIRECT \
    && printf "Done.\nCleaning KOS addons...$NL" \
    && find $KOS/addons -name "*.o" -delete \
    && printf "Done.\n"
RUN \
    # Build extra included utilities
    . /tmp/dockerenv.sh && . $KOS/environ.sh \
    && printf "Building extra included utilities...$NL" \
    && sed -i 's/^DIRS.*/DIRS = bin2c bincnv dcbumpgen genromfs isotest kmgenc makejitter rdtest scramble vqenc wav2adpcm/' $KOS/utils/Makefile \
    && make -j$DOCKER_THREADS -C $KOS/utils &> $REDIRECT \
    && printf "Done.\nSymlinking extra utilities to $DCTOOLCHAIN/bin so they're in the PATH...$NL" \
    && mkdir -p $DCTOOLCHAIN/bin \
    && ln -s $KOS/utils/bin2c/bin2c $DCTOOLCHAIN/bin/bin2c \
    && ln -s $KOS/utils/bin2o/bin2o $DCTOOLCHAIN/bin/bin2o \
    && ln -s $KOS/utils/dcbumpgen/dcbumpgen $DCTOOLCHAIN/bin/dcbumpgen \
    && ln -s $KOS/utils/genexports/genexports.sh $DCTOOLCHAIN/bin/genexports.sh \
    && ln -s $KOS/utils/genexports/genexportstubs.sh $DCTOOLCHAIN/bin/genexportstubs.sh \
    && ln -s $KOS/utils/genromfs/genromfs $DCTOOLCHAIN/bin/genromfs \
    && ln -s $KOS/utils/ipload/ipload.py $DCTOOLCHAIN/bin/ipload.py \
    && ln -s $KOS/utils/isotest/isotest $DCTOOLCHAIN/bin/isotest \
    && ln -s $KOS/utils/kmgenc/kmgenc $DCTOOLCHAIN/bin/kmgenc \
    && ln -s $KOS/utils/makejitter/makejitter $DCTOOLCHAIN/bin/makejitter \
    && ln -s $KOS/utils/rdtest/rdtest $DCTOOLCHAIN/bin/rdtest \
    && ln -s $KOS/utils/scramble/scramble $DCTOOLCHAIN/bin/scramble \
    && ln -s $KOS/utils/vqenc/vqenc $DCTOOLCHAIN/bin/vqenc \
    && ln -s $KOS/utils/wav2adpcm/wav2adpcm $DCTOOLCHAIN/bin/wav2adpcm \
    && printf "Done.\nCleaning extra included utilities...$NL" \
    && find $KOS/utils -name "*.o" -delete \
    && printf "Done.\n"
RUN \
    # Build extra external utilities
    . /tmp/dockerenv.sh && . $KOS/environ.sh \
    && printf "Building extra external utilities...$NL" \
    && mkdir -p $KOS/utils/makeip \
    && cd $KOS/utils/makeip \
    && curl --progress-bar -O http://www.boob.co.uk/files/makeip.tar.gz &> $REDIRECT \
    && tar -xf makeip.tar.gz &> $REDIRECT \
    && rm makeip.tar.gz &> $REDIRECT \
    && gcc makeip.c -o makeip &> $REDIRECT \
    && ln -s $KOS/utils/makeip/makeip $DCTOOLCHAIN/bin/makeip \
    && cd $KOS/utils \ 
    && git clone https://github.com/einsteinx2/img4dc.git &> $REDIRECT \
    && cd $KOS/utils/img4dc \
    && cmake . &> $REDIRECT \
    && make cdi4dc &> $REDIRECT \
    && ln -s $KOS/utils/img4dc/cdi4dc/cdi4dc $DCTOOLCHAIN/bin/cdi4dc \
    && cd $KOS/utils \ 
    && git clone https://github.com/sizious/dcload-ip.git &> $REDIRECT \
    && cd $KOS/utils/dcload-ip \
    && make install \
    && printf "Done.\n"
RUN \
    # Prepare files to copy in next stage
    . /tmp/dockerenv.sh \
    && printf "Preparing files to copy to next stage...$NL" \
    && mkdir /tmp/copy \
    && mv $DCTOOLCHAIN/sh-elf $DCTOOLCHAIN/arm-eabi $DCTOOLCHAIN/kos $DCTOOLCHAIN/kos-ports $DCTOOLCHAIN/bin /tmp/copy \
    && mv /tmp/dockerenv.sh /root/.bashrc /tmp/copy \
    && printf "Done.\n"

#
# Second stage: Final Alpine image with the GCC cross-compiler toolchain, KOS, KOS-PORTS, Addons, Utils, and some other useful stuff
#

FROM alpine:latest
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"

# Set build arguments
ARG VERBOSE

# Set a local environment variable based on the global arg
ENV DCTOOLCHAIN="/opt/toolchains/dc"

# Copy only the compiled toolchains from the build image to save space
COPY --from=builder /tmp/copy $DCTOOLCHAIN

# Install tools and create entry script that loads the KOS environment so make commands can be given directly
RUN \
    # Move copied files into place
    . $DCTOOLCHAIN/dockerenv.sh \
    && rm -f $DCTOOLCHAIN/dockerenv.sh \
    && mv $DCTOOLCHAIN/.bashrc /root/.bashrc \
\
    # Install useful tools for building Dreamcast projects
    && printf "Installing useful tools for building Dreamcast projects...$NL" \
    && apk --update add --no-cache \
       bash bison bzip2 cdrkit cmake colordiff coreutils curl flex gawk git linux-headers \
       musl-dev make patch python2 sed subversion tar texinfo tree vim wget &> $REDIRECT \
	&& apk --update add --no-cache libelf-dev --repository=http://dl-cdn.alpinelinux.org/alpine/v3.9/main \
    && rm -rf /var/cache/apk/* \
    && printf "Done.\n" \
\
    # Create entry point script to automatically load KOS environment
    && printf "Create entry point script to automatically load KOS environment...$NL" \
    && printf '#!/bin/bash\n. /opt/toolchains/dc/kos/environ.sh\nexec "$@"' > /usr/local/bin/entry.sh \
    && chmod a+x /usr/local/bin/entry.sh \
    && printf "Done.\n"

# Default to /src directory
WORKDIR /src

# Run any command passed in a bash shell with the KOS environment loaded
ENTRYPOINT ["/usr/local/bin/entry.sh"]

# If no command is passed, run an interactive bash shell
CMD ["/bin/bash"]
