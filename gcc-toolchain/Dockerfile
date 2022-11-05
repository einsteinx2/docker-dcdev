# --------------------------------------------------------
# KOS GCC Toolchain Image (octoate/dcdev-gcc-toolchain)
# --------------------------------------------------------
#
# This image is meant to be a minimally sized image to be used as a base image to create the final fully functional 
# KOS toolchain images as KOS changes much more often than the GCC cross-compiler toolchain and takes much less time to compile.
# 
# Build Commands:
# ---------------
# GCC 9 Toolchain: `docker build -t octoate/dcdev-gcc-toolchain:gcc-9 .`
# GCC 4 Toolchain: `docker build --build-arg KOS_GCC_VER=4 -t octoate/dcdev-gcc-toolchain:gcc-4 .`


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

#
# First stage: Alpine-based builder image to compile the GCC toolchain
#

FROM ghcr.io/octoate/gcc-base:latest as builder
LABEL maintainer "Tim Riemann <triemann@octoate.de>"

# Choose GCC major version. Supported values are 9 (SH4 9.3/ARM 8.4), 4 (both 4.7.4)
# Can be set using `--build-arg KOS_GCC_VER=4` when building, default is 9)
ARG KOS_GCC_VER=9

# Set build arguments
ARG TARGETPLATFORM
ARG VERBOSE
ARG THREADS
ARG KOS_REPO
ARG KOS_BRANCH

# Set environment variables
ENV DCTOOLCHAIN="/opt/toolchains/dc"
ENV KOS="$DCTOOLCHAIN/kos"
ENV DC_CHAIN="$KOS/utils/dc-chain"
ENV DC_CHAIN_CONFIG_MK="$KOS/utils/dc-chain/config.mk"

# Use the bash shell instead of sh for the build process 
SHELL ["/bin/bash", "-c"]

# Validate KOS_GCC_VER argument and set build variables
RUN if [ "$KOS_GCC_VER" -eq 9 ]; then \
        # Versions for SH4 toolchain
        printf "export DOCKER_SH4_BINUTILS_VER=2.34\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GCC_VER=9.3.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_NEWLIB_VER=3.3.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GDB_VER=9.2\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_INSIGHT_VER=6.8-1\n" >> /tmp/dockerenv.sh; \
        # Tarball extensions for SH4 toolchain
        printf "export DOCKER_SH4_BINUTILS_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GCC_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_NEWLIB_EXT=gz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GDB_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_INSIGHT_EXT=bz2\n" >> /tmp/dockerenv.sh; \
        # Versions for ARM toolchain
        printf "export DOCKER_ARM_BINUTILS_VER=2.34\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_ARM_GCC_VER=8.4.0\n" >> /tmp/dockerenv.sh; \
        # Tarball extensions for ARM toolchain
        printf "export DOCKER_ARM_BINUTILS_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_ARM_GCC_EXT=xz\n" >> /tmp/dockerenv.sh; \
    elif [ "$KOS_GCC_VER" -eq 4 ]; then \
        # Versions for SH4 toolchain
        printf "export DOCKER_SH4_BINUTILS_VER=2.34\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GCC_VER=4.7.4\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_NEWLIB_VER=2.0.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GDB_VER=9.2\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_INSIGHT_VER=6.8-1\n" >> /tmp/dockerenv.sh; \
        # Tarball extensions for SH4 toolchain
        printf "export DOCKER_SH4_BINUTILS_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GCC_EXT=bz2\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_NEWLIB_EXT=gz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_GDB_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_SH4_INSIGHT_EXT=bz2\n" >> /tmp/dockerenv.sh; \
        # Versions for ARM toolchain
        printf "export DOCKER_ARM_BINUTILS_VER=2.34\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_ARM_GCC_VER=4.7.4\n" >> /tmp/dockerenv.sh; \
        # Tarball extensions for ARM toolchain
        printf "export DOCKER_ARM_BINUTILS_EXT=xz\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_ARM_GCC_EXT=bz2\n" >> /tmp/dockerenv.sh; \
    else \
        printf "Invalid KOS_GCC_VER specified, supported values are 9 (SH4 9.3/ARM 8.4) and 4 (both 4.7.4)."; \
        exit 1; \
    fi \
    && . /tmp/dockerenv.sh \
    && printf "Building SH4 GCC $DOCKER_SH4_GCC_VER and ARM GCC $DOCKER_ARM_GCC_VER\n"

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

# Build the KOS patched GCC cross compiler toolchain for SH4 (Main CPU) and ARM7DI (AICA Control CPU)
RUN \
    # Clone repositories
    . /tmp/dockerenv.sh \
    && printf "Cloning latest mainline KOS...$NL" \
    && mkdir -p $DCTOOLCHAIN \
    && git clone --single-branch --branch $KOS_BRANCH $KOS_REPO $KOS &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Patch the dc-chain config.mk file to set job threads and remove Obj-C support 
    . /tmp/dockerenv.sh \
    && printf "Copying sample dc-chain config.mk file...$NL" \
    && cp $DC_CHAIN/config.mk.stable.sample $DC_CHAIN_CONFIG_MK \
    && printf "Patching dc-chain config.mk file to set threads and languages...$NL" \
    && sed -i "/^makejobs/c\makejobs=-j$DOCKER_THREADS" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^pass2_languages/c\pass2_languages=c,c++" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^erase/c\erase=0" $DC_CHAIN_CONFIG_MK \
    && printf "Done.\n"
RUN \
    # Patch dc-chain config.mk file to set GCC versions
    . /tmp/dockerenv.sh \
    && printf "Patching dc-chain config.mk file to set GCC versions...$NL" \
    \
    && printf "Patching SH4 toolchain versions$NL" \
    && sed -i "/^sh_binutils_ver/c\sh_binutils_ver=$DOCKER_SH4_BINUTILS_VER" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^sh_gcc_ver/c\sh_gcc_ver=$DOCKER_SH4_GCC_VER" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^newlib_ver/c\newlib_ver=$DOCKER_SH4_NEWLIB_VER" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^gdb_ver/c\gdb_ver=$DOCKER_SH4_GDB_VER" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^insight_ver/c\insight_ver=$DOCKER_SH4_INSIGHT_VER" $DC_CHAIN_CONFIG_MK \
    \
    && printf "Patching SH4 toolchain extensions$NL" \
    && sed -i "/^sh_binutils_tarball_type/c\sh_binutils_tarball_type=$DOCKER_SH4_BINUTILS_EXT" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^sh_gcc_tarball_type/c\sh_gcc_tarball_type=$DOCKER_SH4_GCC_EXT" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^newlib_tarball_type/c\newlib_tarball_type=$DOCKER_SH4_NEWLIB_EXT" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^gdb_tarball_type/c\gdb_tarball_type=$DOCKER_SH4_GDB_EXT" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^insight_tarball_type/c\insight_tarball_type=$DOCKER_SH4_INSIGHT_EXT" $DC_CHAIN_CONFIG_MK \
    \
    && printf "Patching ARM toolchain versions$NL" \
    && sed -i "/^arm_binutils_ver/c\arm_binutils_ver=$DOCKER_ARM_BINUTILS_VER" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^arm_gcc_ver/c\arm_gcc_ver=$DOCKER_ARM_GCC_VER" $DC_CHAIN_CONFIG_MK \
    \
    && printf "Patching SH4 toolchain extensions$NL" \
    && sed -i "/^arm_binutils_tarball_type/c\arm_binutils_tarball_type=$DOCKER_ARM_BINUTILS_EXT" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^arm_gcc_tarball_type/c\arm_gcc_tarball_type=$DOCKER_ARM_GCC_EXT" $DC_CHAIN_CONFIG_MK \
    && printf "Done.\n"
RUN \
    # Download and unpack compiler toolchain
    . /tmp/dockerenv.sh \
    && cd $DC_CHAIN \
    && printf "Downloading compiler toolchain...$NL" \
    && $DC_CHAIN/download.sh &> $REDIRECT \
    && printf "Done.\nUnpacking compiler toolchain...$NL" \
    && $DC_CHAIN/unpack.sh &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Patch compiler toolchain
    . /tmp/dockerenv.sh \
    && printf "Patching compiler toolchain...$NL" \
    && make -C $DC_CHAIN patch &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Update arch detection when building GCC 4 for linux/arm64 platform
    . /tmp/dockerenv.sh \
    && printf "Updating arch detection if necessary...$NL" \
    && if [ "$KOS_GCC_VER" -eq 4 ] && [ "$TARGETPLATFORM" == "linux/arm64" ]; then \
            printf "Updating config.guess to support $TARGETPLATFORM when building GCC $KOS_GCC_VER...$NL" \
            && wget -O $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess 'https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD' &> $REDIRECT \
            && wget -O $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub 'https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD' &> $REDIRECT \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/mpc-0.8.1/ \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/libjava/libltdl/ \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/libjava/classpath/ \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/mpfr-2.4.2/ \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/gmp-4.3.2/ \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/newlib-$DOCKER_SH4_NEWLIB_VER/ \
            && cp $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.guess $DC_CHAIN/gcc-$DOCKER_SH4_GCC_VER/config.sub $DC_CHAIN/binutils-$DOCKER_SH4_BINUTILS_VER/; \
        else \
            printf "No need to update config.guess since we're building GCC $KOS_GCC_VER on $TARGETPLATFORM...$NL"; \
        fi \
    && printf "Done.\n"
RUN \
    # Build SH4 compiler toolchain
    . /tmp/dockerenv.sh \
    && printf "Building SH4 compiler toolchain...$NL" \
    && make -C $DC_CHAIN build-sh4 &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Build ARM compiler toolchain
    . /tmp/dockerenv.sh \
    && printf "Building ARM compiler toolchain...$NL" \
    && make -C $DC_CHAIN build-arm &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Build debugging tools
    . /tmp/dockerenv.sh \
    && printf "Building GDB...$NL" \
    && make -C $DC_CHAIN gdb &> $REDIRECT \
    && printf "Done.\n"
RUN \
    # Prepare files to copy in next stage
    . /tmp/dockerenv.sh \
    && printf "Preparing files to copy to next stage...$NL" \
    && mkdir /tmp/copy \
    && mv $DCTOOLCHAIN/sh-elf /tmp/copy \
    && mv $DCTOOLCHAIN/arm-eabi /tmp/copy \
    && mv /tmp/dockerenv.sh /tmp/copy \
    && printf "Done.\n"

#
# Second stage: Alpine image with only the Dreamcast cross-compiler toolchain (no KOS) and minimal files to reduce image size
#

FROM alpine:latest
LABEL maintainer "Tim Riemann <triemann@octoate.de>"

# Set build arguments
ARG VERBOSE

# Set environment variables
ENV DCTOOLCHAIN="/opt/toolchains/dc"

# Copy only the compiled toolchains from the build image to save space
COPY --from=builder /tmp/copy $DCTOOLCHAIN

# Install bash shell and tree
RUN . $DCTOOLCHAIN/dockerenv.sh \
    && rm -f $DCTOOLCHAIN/dockerenv.sh \
    && printf "Installing bash and tree...$NL" \
    && apk --update add --no-cache bash tree &> $REDIRECT \
	&& rm -rf /var/cache/apk/* \
    && printf ". /etc/profile\n" > /root/.bashrc \
    && printf "Done.\n"

# Default to /src directory
WORKDIR /src

# Run any command passed in a bash shell
ENTRYPOINT ["/bin/bash", "-c"]

# If no command is passed, run an interactive bash shell
CMD ["/bin/bash"]
