# ==================================================================
# module list
# ------------------------------------------------------------------
# python        3.7    (conda)
# pytorch       1.8.0  (conda)
# apex          from github
# ==================================================================

FROM nvidia/cuda:11.1-cudnn8-devel-ubuntu18.04

ENV LANG C.UTF-8
ENV OFED_VERSION=5.2-1.0.4.0

RUN apt-mark unhold libnccl & apt remove libnccl2

RUN /bin/sh -c apt-get update && apt-get install -y --no-install-recommends libnccl2=2.8.4-1+cuda11.1

RUN APT_INSTALL="apt-get install -y --no-install-recommends" && \
    GIT_CLONE="git clone --depth 10" && \
    rm -rf /var/lib/apt/lists/* \
           /etc/apt/sources.list.d/cuda.list \
           /etc/apt/sources.list.d/nvidia-ml.list && \

    apt-get update && \
    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        software-properties-common \
        && \
    add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main" && \
    apt-get update && \

# ==================================================================
# tools
# ------------------------------------------------------------------

    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        build-essential \
        apt-utils \
        ca-certificates \
        wget \
        git \
        vim \
        libssl-dev \
        curl \
        unzip \
        unrar \
        cmake \
        net-tools \
        sudo \
        autotools-dev \
        rsync \
        jq \
        openssh-server \
        openssh-client


# ==================================================================
# IB
# ------------------------------------------------------------------
# Install Mellanox OFED user-mode drivers and its prereqs
Run cd $(mktemp -d) && \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install -y --no-install-recommends \
    # For MLNX OFED
        dnsutils \
        pciutils \
        ethtool \
        lsof \
        python-libxml2 \
        quilt \
        libltdl-dev \
        dpatch \
        autotools-dev \
        graphviz \
        autoconf \
        chrpath \
        swig \
        automake \
        tk8.5 \
        tcl8.5 \
        libgfortran3 \
        tcl \
        libnl-3-200 \
        libnl-route-3-200 \
        libnl-route-3-dev \
        libnl-utils \
        gfortran \
        tk \
        bison \
        flex \
        libnuma1 \
        checkinstall && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/* && \
    # libnl1 is not available in ubuntu16 so build from source
    wget -q -O - http://www.infradead.org/~tgr/libnl/files/libnl-1.1.4.tar.gz | tar xzf - && \
    cd libnl-1.1.4 && \
    ./configure && \
    make && \
    checkinstall -D --showinstall=no --install=yes -y -pkgname=libnl1 -A amd64 && \
    cd .. && \
    rm -rf libnl-1.1.4 && \
    wget -q -O - http://www.mellanox.com/downloads/ofed/MLNX_OFED-$OFED_VERSION/MLNX_OFED_LINUX-$OFED_VERSION-ubuntu18.04-x86_64.tgz | tar xzf - && \
    cd MLNX_OFED_LINUX-$OFED_VERSION-ubuntu18.04-x86_64 && \
    ./mlnxofedinstall --add-kernel-support && \
    cd .. && \
    rm -rf MLNX_OFED_LINUX-*

# Update environment variables
ENV PATH=/usr/mpi/gcc/openmpi-4.1.0rc5/bin:$PATH \
    LD_LIBRARY_PATH=/usr/mpi/gcc/openmpi-4.1.0rc5/lib:$LD_LIBRARY_PATH

# ==================================================================
# blobfuse
# ------------------------------------------------------------------

RUN cd $(mktemp -d) && \
wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb && \
dpkg -i packages-microsoft-prod.deb && \
apt-get update && \
apt-get install -y blobfuse

# ==================================================================
# python
# ------------------------------------------------------------------

# Set timezone
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV PATH /usr/local/nvidia/bin:/usr/local/nvidia/lib64:$PATH
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib64:/usr/local/cuda/lib64:$LD_LIBRARY_PATH
ENV PYTHON_VERSION=3.7
RUN wget -O ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh  && \
     chmod +x ~/miniconda.sh && \
     ~/miniconda.sh -b -p /opt/conda && \
     rm ~/miniconda.sh
ENV PATH /opt/conda/bin:$PATH
RUN conda install -y python=3.7 numpy scipy h5py scikit-learn matplotlib dask pandas pyyaml mkl
# install pytorch
RUN conda install pytorch=1.8.0 torchvision torchaudio cudatoolkit=11.1 -c pytorch -c conda-forge
# force update numpy by pip due to cython error
RUN pip install numpy --upgrade
# install more
RUN pip install tensorboard tensorboardX six tqdm lmdb pyarrow py-lz4framed methodtools pathlib regex cython cffi sacrebleu editdistance


# ==================================================================
# apex
# ------------------------------------------------------------------
RUN cd $(mktemp -d) && \
git clone -q https://github.com/NVIDIA/apex.git && \
cd apex && \
git reset --hard 154c6336aa7aedd40d3b3583fb5e1328f9cdf387 && \
pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" --global-option="--deprecated_fused_adam" --global-option="--fast_multihead_attn" --global-option="--xentropy" ./ 


# ==================================================================
# config & cleanup
# ------------------------------------------------------------------

RUN conda install -c r -y conda pip=20.1.1

RUN ldconfig && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* ~/*
