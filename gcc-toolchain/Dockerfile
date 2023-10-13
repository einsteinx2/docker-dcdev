# --------------------------------------------------------
# KOS GCC Toolchain Image (einsteinx2/dcdev-gcc-toolchain)
# --------------------------------------------------------
#
# This image is meant to be a minimally sized image to be used as a base image to create the final fully functional 
# KOS toolchain images as KOS changes much more often than the GCC cross-compiler toolchain and takes much less time to compile.
# 
# Build Commands:
# ---------------
# GCC 13 Toolchain: `docker build -t einsteinx2/dcdev-gcc-toolchain:stable .`
# GCC  4 Toolchain: `docker build --build-arg KOS_GCC_VER=legacy -t einsteinx2/dcdev-gcc-toolchain:legacy .`


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
# First stage: Debian-based builder image to compile the GCC toolchain
#

FROM einsteinx2/dcdev-gcc-base:latest as builder

# Choose GCC major version. Supported values are "stable" (currently SH4 13.2.0/ARM 8.5.0) and "legacy" (always both 4.7.4)
# Can be set using `--build-arg KOS_GCC_VER=legacy` when building, default is stable)
ARG KOS_GCC_VER=stable

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
RUN if [ "$KOS_GCC_VER" != "stable" ] && [ "$KOS_GCC_VER" != "legacy" ]; then \
        printf "Invalid KOS_GCC_VER specified, supported values are \"stable\" (currently SH4 13.2.0/ARM 8.5.0) and \"legacy\" (always both 4.7.4)."; \
        exit 1; \
    fi \
    && printf "Building GCC version $KOS_GCC_VER\n"

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
    # Copy the correct config.mk file and patch it to set job threads and remove Obj-C support 
    . /tmp/dockerenv.sh \
    && printf "Copying sample dc-chain config.mk file...$NL" \
    && cp $DC_CHAIN/config/config.mk.$KOS_GCC_VER.sample $DC_CHAIN_CONFIG_MK \
    && printf "Patching dc-chain config.mk file to set threads and languages...$NL" \
    && sed -i "/^makejobs/c\makejobs=-j$DOCKER_THREADS" $DC_CHAIN_CONFIG_MK \
    && sed -i "/^pass2_languages/c\pass2_languages=c,c++" $DC_CHAIN_CONFIG_MK \
    # TODO: Remember why I don't erase the build directories... I think it was because I needed access to them later for some reason but I don't remember why. Maybe becuase I build SH4 and ARM in separate steps?
    && sed -i "/^erase/c\erase=0" $DC_CHAIN_CONFIG_MK \ 
    && printf "Done.\n"
# RUN \
#     # Update arch detection when building GCC 4 for linux/arm64 platform
#     . /tmp/dockerenv.sh \
#     && printf "Updating arch detection if necessary...$NL" \
#     && if [ "$KOS_GCC_VER" = "legacy" ] && [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
#             printf "Updating config.guess to support $TARGETPLATFORM when building GCC $KOS_GCC_VER...$NL" \
#             # Download GCC and it's dependencies
#             && make -C $DC_CHAIN fetch-sh4 \
#             # Update the config.guess files so it knows what to do with an arm64 linux host
#             && wget -O $DC_CHAIN/gcc-4.7.4/config.guess 'https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD' &> $REDIRECT \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/gcc-4.7.4/mpc/ \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/gcc-4.7.4/libjava/libltdl/ \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/gcc-4.7.4/libjava/classpath/ \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/gcc-4.7.4/mpfr/ \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/gcc-4.7.4/gmp/ \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/newlib-2.0.0/ \
#             && cp $DC_CHAIN/gcc-4.7.4/config.guess $DC_CHAIN/binutils-2.34/; \
#         else \
#             printf "No need to update config.guess since we're building GCC $KOS_GCC_VER on $TARGETPLATFORM...$NL"; \
#         fi \
#     && printf "Done.\n"
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
# Second stage: Debian image with only the Dreamcast cross-compiler toolchain (no KOS) and minimal files to reduce image size
#

FROM debian:12-slim
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"
LABEL org.opencontainers.image.source=https://github.com/einsteinx2/docker-dcdev
LABEL org.opencontainers.image.licenses=MIT
LABEL org.opencontainers.image.description="The base image to create the final fully functional KOS toolchain images as KOS changes much more often than the GCC cross-compiler toolchain and takes much less time to compile."

# Set build arguments
ARG VERBOSE

# Set environment variables
ENV DCTOOLCHAIN="/opt/toolchains/dc"

# Copy only the compiled toolchains from the build image to save space
COPY --from=builder /tmp/copy $DCTOOLCHAIN

# Default to /src directory
WORKDIR /src

# Run any command passed in a bash shell
ENTRYPOINT ["/bin/bash", "-c"]

# If no command is passed, run an interactive bash shell
CMD ["/bin/bash"]
