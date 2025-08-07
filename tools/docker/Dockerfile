FROM snipersim/benchmarks:latest AS dockerhub

# Install compatible Sniper version from GitHub
ARG SNIPER_GIT_REPO=https://github.com/snipersim/snipersim
ARG SNIPER_GIT_TREEISH=b58797c3993148174c7de23a18f71fc22e92340f
ARG PINPLAY_DOWNLOAD=https://snipersim.org/packages/pinplay-dcfg-3.11-pin-3.11-97998-g7ecce2dac-gcc-linux.tar.bz2
# Tell Sniper to use Pin/Pinplay (instead of SDE) and allow the use of our own version
ARG USE_PINPLAY=1

RUN yum update -y && \
    yum install -y nano && \
    yum clean all

# Install dependencies for Python build
RUN yum groupinstall -y "Development Tools" && \
    yum install -y gcc openssl-devel bzip2-devel libffi-devel wget make && \
    yum clean all

# Build and install Python 3.10 from source
RUN cd /tmp && \
    wget https://www.python.org/ftp/python/3.10.13/Python-3.10.13.tgz && \
    tar xzf Python-3.10.13.tgz && \
    cd Python-3.10.13 && \
    ./configure --enable-optimizations && \
    make altinstall && \
    ln -s /usr/local/bin/python3.10 /usr/bin/python3 && \
    ln -s /usr/local/bin/pip3.10 /usr/bin/pip3 && \
    cd / && rm -rf /tmp/Python-3.10.13*

# Copy ThermSniper parts into container home
COPY ThermSniper /root/ThermSniper
RUN rm -rf /root/ThermSniper/external/snipersim && \
    rm -rf /root/ThermSniper/benchmarks && \
    rm -rf /root/ThermSniper/external/benchmarks && \
    rm -f /root/ThermSniper/external/HotSpot/hotspot

# Clone snipersim under external/
RUN git clone ${SNIPER_GIT_REPO} /root/ThermSniper/external/snipersim && \
    cd /root/ThermSniper/external/snipersim && \
    git checkout ${SNIPER_GIT_TREEISH} 

# Replace makefile for tests...
COPY ThermSniper/external/snipersim/test/shared/Makefile.shared /root/ThermSniper/external/snipersim/test/shared/Makefile.shared

# Replace run-sniper in cloned snipersim with one from benchmarks dir
COPY ThermSniper/benchmarks/run-sniper /root/ThermSniper/external/snipersim/run-sniper

# Replace mcpat.py in snipersim/tools with one from benchmarks dir
COPY ThermSniper/benchmarks/mcpat.py /root/ThermSniper/external/snipersim/tools/mcpat.py

# Replace intervalThermSniper.py in ThermSniper/scripts with one from benchmarks dir
COPY ThermSniper/benchmarks/intervalThermSniper.py /root/ThermSniper/scripts/intervalThermSniper.py

# Utility such that sniper root can be found for benchmark execution in container w.r.t. ThermSniper dir structure
COPY ThermSniper/benchmarks/env_setup_bm.py /root/benchmarks/tools/scheduler/../python/env_setup_bm.py

# Replace runThermSniper in cloned snipersim with one from benchmarks dir
COPY ThermSniper/benchmarks/runThermSniper.py  /root/ThermSniper/runThermSniper.py

# Let's Encrypt certificates aren't supported in Centos 6.x
ARG WGET_OPTIONS=--no-check-certificate

# Build HotSpot binary
RUN cd /root/ThermSniper/external/HotSpot && \
    make

RUN cd /root/ThermSniper/external/snipersim && \
    mkdir -p pin_kit && \
    wget --quiet $WGET_OPTIONS $PINPLAY_DOWNLOAD && \
    tar -x -f $(basename $PINPLAY_DOWNLOAD) --auto-compress --strip-components=1 -C pin_kit && \
    rm $(basename $PINPLAY_DOWNLOAD) && \
    chmod +x run-sniper &&\
    chmod +x tools/mcpat.py &&\
    sed -i 's/sys\.executable/"python2"/g' ../../runThermSniper.py &&\
    make

