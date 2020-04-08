#
# KOS GCC toolchain image (einsteinx2/dcdev-gcc-toolchain:gcc-9.3)
# ----------------------------------------------------------------
#
# This image is meant to be a minimally sized image to be used as a base image to create the final fully functional 
# KOS toolchain images as KOS changes much more often than the GCC cross-compiler toolchain and takes much less time to compile.
# 
# Docker image build instructions:
# Silence output for faster build: `docker build -t einsteinx2/dcdev-gcc-toolchain:gcc-9.3 .`
# Verbose output to help debugging: `docker build --build-arg VERBOSE=true -t einsteinx2/dcdev-gcc-toolchain:gcc-9.3 .`

#
# Global Arguments
#

# Default to verbose output off as it should build faster without all of the console printing
# NOTE: To debug build problems add the flag `--build-arg VERBOSE=true` when building the image
ARG VERBOSE=false

# For maximum speed, set this to the number of CPU threads you have
# i.e. if you have a 4 core / 8 thread CPU, choose 8
# (can be set using `--build-arg THREADS=8` when building)
ARG THREADS=4

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

# Build the KOS patched GCC cross compiler toolchain for SH4 (Main CPU) and ARM7DI (AICA Control CPU)
RUN \
    # Clone repositories
    . /tmp/verbose.sh \
    && printf "Cloning latest mainline KOS...$NL" >&2 \
    && mkdir -p $DCTOOLCHAIN \
    && git clone --single-branch --branch $KOS_BRANCH $KOS_REPO $KOS &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Download and unpack compiler toolchain
    . /tmp/verbose.sh \
    && cd $DC_CHAIN \
    && printf "Downloading compiler toolchain...$NL" >&2 \
    && $DC_CHAIN/download.sh &> $REDIRECT \
    && printf "Done.\nUnpacking compiler toolchain...$NL" >&2 \
    && $DC_CHAIN/unpack.sh &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Patch the dc-chain Makefile to set job threads and remove Obj-C support 
    . /tmp/verbose.sh \
    && printf "Patching dc-chain Makefile...$NL" >&2 \
    && sed -i "/^makejobs/c\makejobs=-j$THREADS" $DC_CHAIN/Makefile \
    && sed -i "/^pass2_languages/c\pass2_languages=c,c++" $DC_CHAIN/Makefile \
    && printf "Done.\n" >&2
RUN \
    # Patch compiler toolchain
    . /tmp/verbose.sh \
    && printf "Patching compiler toolchain...$NL" >&2 \
    && make -C $DC_CHAIN patch &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Build SH4 compiler toolchain
    . /tmp/verbose.sh \
    && printf "Building SH4 compiler toolchain...$NL" >&2 \
    && make -C $DC_CHAIN build-sh4 &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Build ARM compiler toolchain
    . /tmp/verbose.sh \
    && printf "Building ARM compiler toolchain...$NL" >&2 \
    && make -C $DC_CHAIN build-arm &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Build debugging tools
    . /tmp/verbose.sh \
    && printf "Building GDB...$NL" >&2 \
    && make -C $DC_CHAIN gdb &> $REDIRECT \
    && printf "Done.\n" >&2
RUN \
    # Prepare files to copy in next stage
    . /tmp/verbose.sh \
    && printf "Preparing files to copy to next stage...$NL" >&2 \
    && mkdir /tmp/copy \
    && mv $DCTOOLCHAIN/sh-elf /tmp/copy \
    && mv $DCTOOLCHAIN/arm-eabi /tmp/copy \
    && mv /tmp/verbose.sh /tmp/copy \
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
RUN . $DCTOOLCHAIN/verbose.sh \
    && rm -f $DCTOOLCHAIN/verbose.sh \
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
