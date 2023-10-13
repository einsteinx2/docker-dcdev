# -------------------------------------------------
# GCC Base Image (einsteinx2/dcdev-gcc-base)
# -------------------------------------------------
#
# This image is meant to be a minimally sized image to be used as a base image to create the GCC cross-compiler 
# toolchain image that will be used to build the final fully functional KOS toolchain images to prevent the need to 
# download all of the dependencies each time when rebuilding images locally (or CI if image caching is available).
# When cached locally, this saves hundreds of megabytes of package downloads plus install time.
# 
# Build Command:
# -------------- 
# `docker build -t einsteinx2/dcdev-gcc-base:latest .`


FROM debian:12-slim
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"
LABEL org.opencontainers.image.source=https://github.com/einsteinx2/docker-dcdev
LABEL org.opencontainers.image.licenses=MIT
LABEL org.opencontainers.image.description="The base image to create the GCC cross-compiler toolchain image that will be used to build the final fully functional KOS toolchain images to prevent the need to download all of the dependencies each time when rebuilding images locally (or CI if image caching is available). When cached locally, this saves hundreds of megabytes of package downloads plus install time."

# Download all necessary dependencies needed to build the KOS GCC cross-compiler toolchain
RUN \
    # Install dependencies
    printf "Installing dependencies...\n" \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get update && apt-get upgrade -y \
    && apt-get install -y cmake gawk patch bzip2 tar make libgmp-dev libmpfr-dev libmpc-dev \
        gettext wget libelf-dev texinfo bison flex sed git build-essential diffutils curl \
        libjpeg-dev libpng-dev python3 pkg-config libisofs-dev meson ninja-build \
	&& rm -rf /var/lib/apt/lists/* \
    && printf "Done.\n" \
\
    # Create verbose output toggle script used in child images (file can be appended to by child images to store more environment variables)
    && printf 'if $VERBOSE; then\n  export REDIRECT="/dev/stdout";\n  export NL="\\n";\nelse\n  export REDIRECT="/dev/null";\n  export NL="";\nfi\n' > /tmp/dockerenv.sh \
    && chmod a+x /tmp/dockerenv.sh

# Default to /src directory
WORKDIR /src

# Run any command passed in a bash shell
ENTRYPOINT ["/bin/bash", "-c"]

# If no command is passed, run an interactive bash shell
CMD ["/bin/bash"]
