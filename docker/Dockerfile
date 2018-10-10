FROM nvidia/cuda:8.0-cudnn6-devel-ubuntu16.04

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        cmake \
        curl \
        git \
        wget \
        libatlas-base-dev \
        libboost-all-dev \
        libcgal-dev \
        libeigen3-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler \
        python-dev \
        python-numpy \
        python-setuptools \
        python-scipy \
        rsync \
        software-properties-common \
        zip && \
    rm -rf /var/lib/apt/lists/*

RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
        rm get-pip.py

ARG WORKSPACE=/workspace

ARG P3D_DIR=$WORKSPACE/p3d
ARG P3D_COMMIT=origin/master
WORKDIR $P3D_DIR
RUN git clone https://github.com/ZhaofanQiu/pseudo-3d-residual-networks.git . && \
    git reset --hard $P3D_COMMIT

ARG NCCL_DIR=$WORKSPACE/nccl
ARG NCCL_COMMIT=286916a1a37ca1fe8cd43e280f5c42ec29569fc5
WORKDIR $NCCL_DIR
RUN git clone https://github.com/NVIDIA/nccl.git . && git reset --hard $NCCL_COMMIT && \
    make -j install && cd .. && rm -rf $NCCL_DIR

ARG CAFFE_ROOT=/opt/caffe
ARG CAFFE_COMMIT=99bd99795dcdf0b1d3086a8d67ab1782a8a08383
WORKDIR $CAFFE_ROOT
RUN git clone https://github.com/BVLC/caffe.git . && git reset --hard $CAFFE_COMMIT && \
    rsync -a $P3D_DIR/caffe_add_layers/include/ ./include/caffe/layers/ && \
    rsync -a $P3D_DIR/caffe_add_layers/src/ ./src/caffe/layers/ && \
    cat $P3D_DIR/caffe_add_layers/caffe.proto >> ./src/caffe/proto/caffe.proto && \
    ex -sc '424i|  optional Pooling3DParameter pooling3d_param = 149;' -cx ./src/caffe/proto/caffe.proto && \
    ex -sc '425i|  optional BNParameter bn_param = 150;' -cx ./src/caffe/proto/caffe.proto && \
    ex -sc '83i|set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++11")' -cx CMakeLists.txt && \
    cd python && for req in $(cat requirements.txt) pydot; do pip install $req; done && cd .. && \
    mkdir build && cd build && \
    cmake -DUSE_CUDNN=1 -DUSE_NCCL=1 .. && \
    make -j"$(nproc)" && \
    mkdir $CAFFE_ROOT/include/caffe/proto && \
    cp $CAFFE_ROOT/build/include/caffe/proto/caffe.pb.h $CAFFE_ROOT/include/caffe/proto/caffe.pb.h

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

WORKDIR /workspace
