# ------------------------------------------------------------------------------
# Pull base image
FROM ubuntu:20.04

# ------------------------------------------------------------------------------
# Arguments
ARG WORKDIR=/root

# ------------------------------------------------------------------------------
# Install tools via apt
ENV DEBIAN_FRONTEND=noninteractive
RUN set -x \
    && apt -y update \
    && apt -y install \
    git \
    wget \
    python3 \
    python3-dev \
    python3-setuptools \
    python3-pip \
    build-essential \
    astyle \
    mercurial \
    ninja-build \
    libssl-dev \
    cargo \
    flex \
    bison \
    doxygen \
    aspell \
    ccache \
    gcovr \
    && apt clean && rm -rf /var/lib/apt/lists \
    && update-alternatives --install /usr/bin/python python /usr/bin/python3.8 1 \
    && : # last line

# Set up Mbed environment
WORKDIR /tmp/
COPY requirements.txt .
RUN set -x \
    && pip3 install -r requirements.txt  \
    && : # last line

# ------------------------------------------------------------------------------
# Install Python modules (which are not included in requirements.txt)
RUN set -x \
    && pip3 install -U \
    mbed-cli \
    && : # last line

# ------------------------------------------------------------------------------
# install scancode-toolkit, which is  available only in x86_64
RUN set -x \
    && [ "$(uname -m)" = "x86_64" ] && \
        pip install scancode-toolkit

# ------------------------------------------------------------------------------
# Install arm-none-eabi-gcc
WORKDIR /opt/mbed-os-toolchain

RUN set -x \
    && [ "$(uname -m)" = "aarch64" ] && \
        TARBALL="gcc-arm-none-eabi-9-2019-q4-major-aarch64-linux.tar.bz2" || \
        TARBALL="gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2" \
    && wget -q https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/${TARBALL} \
    && tar -xjf ${TARBALL} \
    && rm ${TARBALL} \
    && : # last line

# ------------------------------------------------------------------------------

# Configure environment variables
ENV MBED_GCC_ARM_PATH=/opt/mbed-os-toolchain/gcc-arm-none-eabi-9-2019-q4-major/bin/
ENV PATH="${PATH}:${MBED_GCC_ARM_PATH}"

# ------------------------------------------------------------------------------
# Display, check and save environment settings
# NOTE: using bash instead of Ubuntu default bash due to unsupport for pipefail
# Pipefail is crucial here, if the tools didn't install properly, docker build should not pass because of piping 
RUN /bin/bash -c \
   "set -x -o pipefail \
    && arm-none-eabi-gcc --version | grep arm-none-eabi-gcc | tee env_settings \
    && python --version 2>&1 | tee -a env_settings \
    && (echo -n 'mbed-cli ' && mbed --version) | tee -a env_settings \
    && (echo -n 'mbed-greentea ' && mbedgt --version | grep ^[0-9]) | tee -a env_settings \
    && (echo -n 'mbed-host-tests ' && mbedhtrun --version) | tee -a env_settings \
    && : # LAST LINE"

WORKDIR /root
