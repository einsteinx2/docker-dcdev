# --------------------------------------------------------
# KOS GCC Toolchain Image (einsteinx2/dcdev-gcc-toolchain)
# --------------------------------------------------------
#
# This image is meant to be a minimally sized image to be used as a base image to create the final fully functional 
# KOS toolchain images as KOS changes much more often than the GCC cross-compiler toolchain and takes much less time to compile.
# 
# Build Commands:
# ---------------
# GCC 9 Toolchain: `docker build -t einsteinx2/dcdev-gcc-toolchain:gcc-9 .`
# GCC 4 Toolchain: `docker build --build-arg KOS_GCC_VER=4 -t einsteinx2/dcdev-gcc-toolchain:gcc-4 .`


#
# Global Arguments
#

# Defaults to verbose output off as it should build faster without all of the console printing.
# To debug build problems, add the flag `--build-arg VERBOSE=true` when building the image to 
# have it print all output from each download and compilation step.
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

FROM einsteinx2/dcdev-gcc-base:latest as builder
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"

# Choose GCC major version. Supported values are 9 (SH4 9.3/ARM 8.4), 4 (both 4.7.4)
# Can be set using `--build-arg KOS_GCC_VER=4` when building, default is 9)
ARG KOS_GCC_VER=9

# Set build arguments
ARG VERBOSE
ARG THREADS
ARG KOS_REPO
ARG KOS_BRANCH

# Set environment variables
ENV DCTOOLCHAIN="/opt/toolchains/dc"
ENV KOS="$DCTOOLCHAIN/kos"
ENV DC_CHAIN="$KOS/utils/dc-chain"

# Use the bash shell instead of sh for the build process 
SHELL ["/bin/bash", "-c"]

# Validate KOS_GCC_VER argument and set build variables
RUN if [ "$KOS_GCC_VER" -eq 9 ]; then \
        printf "export DOCKER_SH4_GCC_VER=9.3.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_ARM_GCC_VER=8.4.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_BINUTILS_VER=2.34\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_NEWLIB_VER=3.3.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_GMP_VER=6.1.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_MPFR_VER=3.1.4\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_MPC_VER=1.0.3\n" >> /tmp/dockerenv.sh; \
    elif [ "$KOS_GCC_VER" -eq 4 ]; then \
        printf "export DOCKER_SH4_GCC_VER=4.7.4\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_ARM_GCC_VER=4.7.4\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_BINUTILS_VER=2.31.1\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_NEWLIB_VER=2.0.0\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_GMP_VER=4.3.2\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_MPFR_VER=2.4.2\n" >> /tmp/dockerenv.sh; \
        printf "export DOCKER_MPC_VER=0.8.1\n" >> /tmp/dockerenv.sh; \
    else \
        printf "Invalid KOS_GCC_VER specified, supported values are 9 (SH4 9.3/ARM 8.4) and 4 (both 4.7.4)." >&2; \
        exit 1; \
    fi \
    && . /tmp/dockerenv.sh \
    && printf "Building SH4 GCC $DOCKER_SH4_GCC_VER and ARM GCC $DOCKER_ARM_GCC_VER\n" >&2

# Validate THREADS argument and set build variables
RUN if [ "$THREADS" -eq 0 ] 2>/dev/null; then \
        # If THREADS is 0, default to max possible threads
        printf "export DOCKER_THREADS=$(getconf _NPROCESSORS_ONLN)\n" >> /tmp/dockerenv.sh; \
    elif ! [[ -z "${THREADS//[0-9]}" ]]; then \
        # If THREADS is a negative number or any non-integer, print an error and exit
        printf "Invalid THREADS specified, please enter a positive integer value\n"; >&2; \
        exit 1; \
    else \
        # If THREADS is a positive integer, use that value
        printf "export DOCKER_THREADS=$THREADS\n" >> /tmp/dockerenv.sh; \
    fi \
    && . /tmp/dockerenv.sh \
    && printf "Building using $DOCKER_THREADS threads\n" >&2

# Build the KOS patched GCC cross compiler toolchain for SH4 (Main CPU) and ARM7DI (AICA Control CPU)
RUN \
    # Clone repositories
    . /tmp/dockerenv.sh \
    && printf "Cloning latest mainline KOS...$NL" >&2 \
    && mkdir -p $DCTOOLCHAIN \
    && git clone --single-branch --branch $KOS_BRANCH $KOS_REPO $KOS &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Patch the dc-chain Makefile to set job threads and remove Obj-C support 
    . /tmp/dockerenv.sh \
    && printf "Patching dc-chain Makefile...$NL" >&2 \
    && sed -i "/^makejobs/c\makejobs=-j$DOCKER_THREADS" $DC_CHAIN/Makefile \
    && sed -i "/^pass2_languages/c\pass2_languages=c,c++" $DC_CHAIN/Makefile \
    && printf "Done.\n" >&2
RUN \
    # Patch dc-chain Makefile and scripts to set GCC versions
    . /tmp/dockerenv.sh \
    && printf "Patch dc-chain Makefile and scripts to set GCC versions...$NL" >&2 \
    # Patch Makefile
    && sed -i "/^sh_gcc_ver/c\sh_gcc_ver=$DOCKER_SH4_GCC_VER" $DC_CHAIN/Makefile \
    && sed -i "/^arm_gcc_ver/c\arm_gcc_ver=$DOCKER_ARM_GCC_VER" $DC_CHAIN/Makefile \
    && sed -i "/^sh_binutils_ver/c\sh_binutils_ver=$DOCKER_BINUTILS_VER" $DC_CHAIN/Makefile \
    && sed -i "/^arm_binutils_ver/c\arm_binutils_ver=$DOCKER_BINUTILS_VER" $DC_CHAIN/Makefile \
    && sed -i "/^newlib_ver/c\newlib_ver=$DOCKER_NEWLIB_VER" $DC_CHAIN/Makefile \
    # Patch scripts
    && scripts=( download.sh unpack.sh cleanup.sh ) \
    && for script in "${scripts[@]}"; do \
         sed -i "/^export SH_GCC_VER/c\export SH_GCC_VER=$DOCKER_SH4_GCC_VER" $DC_CHAIN/$script; \
         sed -i "/^export ARM_GCC_VER/c\export ARM_GCC_VER=$DOCKER_ARM_GCC_VER" $DC_CHAIN/$script; \
         sed -i "/^export BINUTILS_VER/c\export BINUTILS_VER=$DOCKER_BINUTILS_VER" $DC_CHAIN/$script; \
         sed -i "/^export NEWLIB_VER/c\export NEWLIB_VER=$DOCKER_NEWLIB_VER" $DC_CHAIN/$script; \
         sed -i "/^export GMP_VER/c\export GMP_VER=$DOCKER_GMP_VER" $DC_CHAIN/$script; \
         sed -i "/^export MPFR_VER/c\export MPFR_VER=$DOCKER_MPFR_VER" $DC_CHAIN/$script; \
         sed -i "/^export MPC_VER/c\export MPC_VER=$DOCKER_MPC_VER" $DC_CHAIN/$script; \
       done
RUN \
    # Download and unpack compiler toolchain
    . /tmp/dockerenv.sh \
    && cd $DC_CHAIN \
    && printf "Downloading compiler toolchain...$NL" >&2 \
    && $DC_CHAIN/download.sh &> $REDIRECT \
    && printf "Done.\nUnpacking compiler toolchain...$NL" >&2 \
    && $DC_CHAIN/unpack.sh &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Patch compiler toolchain
    . /tmp/dockerenv.sh \
    && printf "Patching compiler toolchain...$NL" >&2 \
    && make -C $DC_CHAIN patch &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Build SH4 compiler toolchain
    . /tmp/dockerenv.sh \
    && printf "Building SH4 compiler toolchain...$NL" >&2 \
    && make -C $DC_CHAIN build-sh4 &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Build ARM compiler toolchain
    . /tmp/dockerenv.sh \
    && printf "Building ARM compiler toolchain...$NL" >&2 \
    && make -C $DC_CHAIN build-arm &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Build debugging tools
    . /tmp/dockerenv.sh \
    && printf "Building GDB...$NL" >&2 \
    && make -C $DC_CHAIN gdb &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Prepare files to copy in next stage
    . /tmp/dockerenv.sh \
    && printf "Preparing files to copy to next stage...$NL" >&2 \
    && mkdir /tmp/copy \
    && mv $DCTOOLCHAIN/sh-elf /tmp/copy \
    && mv $DCTOOLCHAIN/arm-eabi /tmp/copy \
    && mv /tmp/dockerenv.sh /tmp/copy \
    && printf "Done.\n" >&2

#
# Second stage: Alpine image with only the Dreamcast cross-compiler toolchain (no KOS) and minimal files to reduce image size
#

FROM alpine:latest
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"

# Set build arguments
ARG VERBOSE

# Set environment variables
ENV DCTOOLCHAIN="/opt/toolchains/dc"

# Copy only the compiled toolchains from the build image to save space
COPY --from=builder /tmp/copy $DCTOOLCHAIN

# Install bash shell and tree
RUN . $DCTOOLCHAIN/dockerenv.sh \
    && rm -f $DCTOOLCHAIN/dockerenv.sh \
    && printf "Installing bash and tree...$NL" >&2 \
    && apk --update add --no-cache bash tree &> $REDIRECT \
	&& rm -rf /var/cache/apk/* \
    && printf ". /etc/profile\n" > /root/.bashrc \
    && printf "Done.\n" >&2

# Default to /src directory
WORKDIR /src

# Run any command passed in a bash shell
ENTRYPOINT ["/bin/bash", "-c"]

# If no command is passed, run an interactive bash shell
CMD ["/bin/bash"]
