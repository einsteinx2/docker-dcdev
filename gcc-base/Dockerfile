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


FROM alpine:latest
LABEL maintainer "Ben Baron <ben@einsteinx2.com>"

# Download all necessary dependencies needed to build the KOS GCC cross-compiler toolchain
RUN \
    # Install dependencies
    printf "Installing dependencies...\n" \
    && apk --update add --no-cache bash build-base cmake coreutils curl git libjpeg-turbo-dev libpng-dev linux-headers musl-dev patch python2 subversion texinfo tree wget \
	&& apk --update add --no-cache libelf-dev --repository=http://dl-cdn.alpinelinux.org/alpine/v3.9/main \
	&& rm -rf /var/cache/apk/* \
    && printf "Done.\n" \
\
    # Prepare .bashrc
    && printf ". /etc/profile\n" > ~/.bashrc \
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
