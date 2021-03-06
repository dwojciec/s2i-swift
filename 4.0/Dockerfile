# This container can provide an environment for building and running Swift applications.
# latest info on swift releases for linux are here: https://swift.org/download/#releases

# Base Image Info
# https://hub.docker.com/_/ubuntu/

FROM ubuntu:16.04


##################################################################################################
#### THIS IS BASE UBUNTU IMAGE STUFF, SHOULD EVENTUALLY BE MOVED INTO ONE
##################################################################################################
LABEL \
      # Location of the STI scripts inside the image.
      io.openshift.s2i.scripts-url=image:///usr/libexec/s2i \
      # DEPRECATED: This label will be kept here for backward compatibility.
      io.s2i.scripts-url=image:///usr/libexec/s2i

ENV \
    # DEPRECATED: Use above LABEL instead, because this will be removed in future versions.
    STI_SCRIPTS_URL=image:///usr/libexec/s2i \
    # Path to be used in other layers to place s2i scripts into
    STI_SCRIPTS_PATH=/usr/libexec/s2i \
    # The $HOME is not set by default, but some applications needs this variable
    HOME=/opt/app-root/src \
    PATH=/opt/app-root/src/bin:/opt/app-root/bin:$PATH

RUN \
    mkdir -p ${HOME} && \
    useradd -u 1001 -r -g 0 -d ${HOME} -s /sbin/nologin -c "Default Application User" default && \
    chown -R 1001:0 /opt/app-root

WORKDIR ${HOME}

##################################################################################################


EXPOSE 8080

ENV SWIFT_MAJOR_VERSION=4 \
    SWIFT_MINOR_VERSION=0 \
    SWIFT_PATCH_VERSION=2

ENV SWIFT_VERSION=${SWIFT_MAJOR_VERSION}.${SWIFT_MINOR_VERSION} \
    PATH=$PATH:/swift/usr/bin

LABEL io.k8s.description="Platform for building and running Swift applications" \
      io.k8s.display-name="Swift ${SWIFT_VERSION}" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,swift,swift${SWIFT_MAJOR_VERSION}${SWIFT_MINOR_VERSION}" \
      io.openshift.s2i.scripts-url="image:///${STI_SCRIPTS_PATH}" \
      name="swift-${SWIFT_MAJOR_VERSION}${SWIFT_MINOR_VERSION}-ubuntu14"

# Install toolchain needed for builds
RUN apt-get -q update && \
    DEBIAN_FRONTEND=noninteractive apt-get -q install -y \
    git \
    build-essential \
    clang-3.6 \
    libicu-dev \
    wget \
    curl \
    python2.7-dev \
    libxml2 \
    libcurl4-openssl-dev \
    libssl-dev \
    libbsd-dev && \
    update-alternatives --install /usr/bin/clang clang /usr/bin/clang-3.6 100 && \
    update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-3.6 100 && \
    rm -r /var/lib/apt/lists/*

# Install actual SWIFT version 3.0.x
# Note: 3.1 will use a different S2I so only PATCH_VERSIONS differences will update *this* image
RUN DOWNLOAD_URL=https://swift.org/builds/swift-4.0.3-release/ubuntu1604/swift-4.0.3-RELEASE/swift-4.0.3-RELEASE-ubuntu16.04.tar.gz && \
    curl -fSsL $DOWNLOAD_URL -o swift-${SWIFT_VERSION}.tar.gz && \
    curl -fSsL $DOWNLOAD_URL.sig -o swift-${SWIFT_VERSION}.tar.gz.sig && \
    wget -q -O - https://swift.org/keys/all-keys.asc | gpg --import - && \
    gpg --keyserver hkp://pool.sks-keyservers.net --refresh-keys Swift && \
    gpg --verify swift-${SWIFT_VERSION}.tar.gz.sig && \
    mkdir -p /swift && \
    tar -zxf swift-${SWIFT_VERSION}.tar.gz --strip-components=1 -C /swift && \
    ln -s /swift/usr/bin/swift /usr/local/bin/swift && \
    ln -s /swift/usr/bin/swiftc /usr/local/bin/swiftc && \
    rm -rf swift-${SWIFT_VERSION}.tar.gz && \
    rm -rf swift-${SWIFT_VERSION}.tar.gz.sig

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin $STI_SCRIPTS_PATH

RUN chown -R 1001:0 /opt/app-root && chmod -R ug+rwx /opt/app-root
USER 1001

# Set the default CMD to print the usage of the language image.
CMD $STI_SCRIPTS_PATH/usage
