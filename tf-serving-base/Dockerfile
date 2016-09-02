FROM ubuntu:14.04

RUN apt-get update && apt-get install -y \
        build-essential \
        curl \
        git \
        libfreetype6-dev \
        libpng12-dev \
        libzmq3-dev \
        pkg-config \
        software-properties-common \
        swig \
        zip \
        zlib1g-dev \
        libcurl3-dev \
        python-dev \
        python-numpy \
        python-pip \
        wget \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# We need to add a custom PPA to pick up JDK8, since trusty doesn't
# have an openjdk8 backport.  openjdk-r is maintained by a reliable contributor:
# Matthias Klose (https://launchpad.net/~doko).  It will do until
# we either update the base image beyond 14.04 or openjdk-8 is
# finally backported to trusty; see e.g.
#   https://bugs.launchpad.net/trusty-backports/+bug/1368094
RUN add-apt-repository -y ppa:openjdk-r/ppa && \
    apt-get update && \
    apt-get install -y openjdk-8-jdk openjdk-8-jre-headless && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# Set up Bazel.
# ========================================================================================
# Running bazel inside a `docker build` command causes trouble, cf:
#   https://github.com/bazelbuild/bazel/issues/134
# The easiest solution is to set up a bazelrc file forcing --batch.
RUN echo "startup --batch" >>/root/.bazelrc

# Similarly, we need to workaround sandboxing issues:
#   https://github.com/bazelbuild/bazel/issues/418
RUN echo "build --spawn_strategy=standalone --genrule_strategy=standalone" \
    >>/root/.bazelrc
ENV BAZELRC /root/.bazelrc

# Install the most recent bazel release.
ENV BAZEL_VERSION 0.2.0
WORKDIR /
RUN mkdir /bazel && \
    cd /bazel && \
    wget https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    chmod +x bazel-*.sh && \
    ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh


# Python (2.7), to be upgraded to 3.5
# ========================================================================================
RUN curl -fSsL -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

# Set up grpc
RUN pip install enum34 futures six && \
    pip install --pre protobuf>=3.0.0a3 && \
    pip install --pre grpcio

# Checkout tensorflow
# ========================================================================================
RUN git clone --recurse-submodules https://github.com/tensorflow/serving

# Configure Tensorflow by working-around the prompt for python libs
# ----------------------------------------------------------------------------------------
COPY python_config.sh /serving/tensorflow/util/python/python_config.sh
RUN chmod +x /serving/tensorflow/util/python/python_config.sh
RUN cd serving/tensorflow && ./configure

# Serving
# ========================================================================================
RUN pip install mock
RUN cd serving && bazel build -c opt tensorflow_serving/...

# Example
# ----------------------------------------------------------------------------------------
RUN rm -rf /tmp/mnist_model && \
    cd serving && \
    bazel build tensorflow_serving/example:mnist_export && \
    bazel-bin/tensorflow_serving/example/mnist_export /tmp/mnist_model

#
CMD ["/bin/bash"]