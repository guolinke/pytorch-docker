# ==================================================================
# module list
# ------------------------------------------------------------------
# python        3.8    (conda)
# pytorch       1.8.1  (conda)
# apex          from github
# ==================================================================

FROM nvidia/cuda:11.3.0-cudnn8-devel-ubuntu20.04

ENV LANG C.UTF-8
ENV OFED_VERSION=5.3-1.0.0.1

RUN APT_INSTALL="apt-get install -y --no-install-recommends" && \
    rm -rf /var/lib/apt/lists/* \
           /etc/apt/sources.list.d/cuda.list \
           /etc/apt/sources.list.d/nvidia-ml.list && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        software-properties-common \
        && \
    apt-get update && \
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
        tmux \
        screen \
        htop \
        pdsh \
        openssh-client \
        lshw \
        dmidecode \
        util-linux \
        automake \
        autoconf \
        libtool \
        net-tools \
        pciutils \
        libpci-dev \
        libaio-dev \
        libcap2 \
        libtinfo5 \
        fakeroot \
        devscripts \
        debhelper 

RUN cd /tmp && \
    wget -q http://content.mellanox.com/ofed/MLNX_OFED-${OFED_VERSION}/MLNX_OFED_LINUX-${OFED_VERSION}-ubuntu20.04-x86_64.tgz && \
    tar xzf MLNX_OFED_LINUX-${OFED_VERSION}-ubuntu20.04-x86_64.tgz && \
    MLNX_OFED_LINUX-${OFED_VERSION}-ubuntu20.04-x86_64/mlnxofedinstall --user-space-only --without-fw-update --force --all && \
    rm -rf /tmp/MLNX_OFED_LINUX-${OFED_VERSION}*


RUN cd $(mktemp -d) && \
    git clone -b v2.9.9-1 --depth=1 https://github.com/nvidia/nccl.git && \
    cd nccl && \
    make -j src.build NVCC_GENCODE="-gencode=arch=compute_70,code=sm_70  -gencode=arch=compute_80,code=sm_80" && \
    make pkg.debian.build && \
    sudo dpkg -i build/pkg/deb/*

RUN cd /tmp && \
    mkdir -p /usr/local/nccl-rdma-sharp-plugins && \
    DEBIAN_FRONTEND=noninteractive apt install -y zlib1g-dev && \
    git clone --depth=1 https://github.com/Mellanox/nccl-rdma-sharp-plugins.git && \
    cd nccl-rdma-sharp-plugins && \
    ./autogen.sh && \
    ./configure --prefix=/usr/local/nccl-rdma-sharp-plugins --with-cuda=/usr/local/cuda && \
    make && \
    make install

RUN cd /tmp && \
    git clone --depth=1 https://github.com/Azure/azhpc-images && \
    cd azhpc-images && \
    bash common/network-tuning.sh

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
ENV PYTHON_VERSION=3.8
RUN wget -O ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh  && \
     chmod +x ~/miniconda.sh && \
     ~/miniconda.sh -b -p /opt/conda && \
     rm ~/miniconda.sh
ENV PATH /opt/conda/bin:$PATH
RUN conda install -y python=3.8 numpy scipy h5py scikit-learn matplotlib dask pandas pyyaml mkl
RUN pip install tensorboard tensorboardX pyarrow 

# install pytorch
RUN ldconfig

# set location for builds
ENV NCCL_INCLUDE_DIR="/usr/include" \
    NCCL_LIB_DIR="/usr/lib/x86_64-linux-gnu" \
    USE_SYSTEM_NCCL=1

# ==================================================================
# pytorch
# ------------------------------------------------------------------
ENV TORCH_CUDA_ARCH_LIST "7.0;7.5;8.0"

RUN conda install -y numpy pyyaml scipy ipython mkl mkl-include ninja cython typing && \
    conda install -y -c pytorch magma-cuda113 && \
    conda clean -ya

RUN cd $(mktemp -d) && \
    git clone -b v1.9.0 --depth=1 https://github.com/pytorch/pytorch --recursive && \
    cd pytorch && \
    CMAKE_PREFIX_PATH="$(dirname $(which conda))/../" && \
    pip install --no-cache-dir -v .

RUN wget https://raw.githubusercontent.com/microsoft/DeepSpeed/master/bin/ds_ssh && \
    cp ds_ssh /usr/local/bin && \
    chmod +x /usr/local/bin/ds_ssh


RUN conda install -c r -y conda pip=20.1.1

ENV LD_LIBRARY_PATH=/usr/local/nccl-rdma-sharp-plugins/lib:$LD_LIBRARY_PATH
ENV PATH=/usr/mpi/gcc/openmpi-4.1.0rc5/bin:$PATH
ENV LD_LIBRARY_PATH=/usr/mpi/gcc/openmpi-4.1.0rc5/lib:$LD_LIBRARY_PATH
ENV PYTHONUSERBASE=/tmp/site-packages 
ENV PATH=$PATH:$PYTHONUSERBASE/bin

RUN ldconfig && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* && \
    conda clean -ya

RUN mkdir -p $PYTHONUSERBASE && chmod -R a+rwx $PYTHONUSERBASE

