# A image for building paddle binaries
# Use cuda devel base image for both cpu and gpu environment
# When you modify it, please be aware of cudnn-runtime version
FROM nvidia/cuda:11.2.1-cudnn8-devel-ubuntu18.04
MAINTAINER PaddlePaddle Authors <paddle-dev@baidu.com>

# ENV variables
ARG WITH_GPU
ARG WITH_AVX

ENV WITH_GPU=${WITH_GPU:-ON}
ENV WITH_AVX=${WITH_AVX:-ON}
ENV DEBIAN_FRONTEND=noninteractive
ENV LD_LIBRARY_PATH=/usr/local/cuda-11.2/targets/x86_64-linux/lib:$LD_LIBRARY_PATH 

RUN chmod 777 /tmp

RUN apt-key del 7fa2af80
RUN rm /etc/apt/sources.list.d/cuda.list 
RUN apt-key adv --fetch-keys https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub

RUN apt-get update --allow-unauthenticated && \
  apt-get install -y software-properties-common && add-apt-repository ppa:deadsnakes/ppa && \
  apt-get update && \
  apt-get install -y curl wget vim git unzip unrar tar xz-utils libssl-dev bzip2 gzip \ 
    coreutils ntp language-pack-zh-hans python-qt4 libsm6 libxext6 libxrender-dev libgl1-mesa-glx \
    bison graphviz libjpeg-dev zlib1g-dev automake locales swig net-tools libtool module-init-tools

WORKDIR /usr/bin 
# install tensorrt
#RUN wget -q https://paddle-qa.bj.bcebos.com/nvidia/trt/TensorRT-8.2.4.2.tgz --no-check-certificate \
#    && tar -xvf TensorRT-8.2.4.2.tgz -C /usr/local \
COPY ./TensorRT-8.2.4.2.tgz /usr/bin
RUN tar -xvf TensorRT-8.2.4.2.tgz -C /usr/local \
    && cp -rf /usr/local/TensorRT-8.2.4.2/include/* /usr/include/ && cp -rf /usr/local/TensorRT-8.2.4.2/lib/* /usr/lib/ \
    && rm -f TensorRT-8.2.4.2.tgz
RUN git clone -b 0.15.0 https://github.com/NixOS/patchelf patchelf_tmp \
    && cd patchelf_tmp \
    && ./bootstrap.sh \
    && ./configure \
    && make \
    && make install \
    && cd .. \
    && rm -rf patchelf_tmp 

# cudnn8.1.1 -> cudnn8.2.1
RUN rm -rf /usr/include/cudnn* && rm -rf /usr/lib/x86_64-linux-gnu/libcudnn* 
#RUN wget -q https://paddle-ci.gz.bcebos.com/cudnn/cudnn-11.3-linux-x64-v8.2.1.32.tgz --no-check-certificate && \
#    tar -xvf cudnn-11.3-linux-x64-v8.2.1.32.tgz && \
COPY ./cudnn-11.3-linux-x64-v8.2.1.32.tgz /usr/bin
RUN tar -xvf cudnn-11.3-linux-x64-v8.2.1.32.tgz && \
    cd cuda && \
    cp -r include /usr && \
    cp lib64/* /usr/lib/x86_64-linux-gnu/ && cd ../ && \
    rm -f cudnn-11.3-linux-x64-v8.2.1.32.tgz && \
    rm -rf cuda

# install cmake
WORKDIR /home
RUN wget -q https://cmake.org/files/v3.16/cmake-3.16.0-Linux-x86_64.tar.gz && tar -zxvf cmake-3.16.0-Linux-x86_64.tar.gz && rm cmake-3.16.0-Linux-x86_64.tar.gz
ENV PATH=/home/cmake-3.16.0-Linux-x86_64/bin:$PATH


RUN apt-get update && \
  apt-get install -y python3.7 python3.7-dev \
  python3.8 python3.8-dev python3.8-distutils \
  python3.9 python3.9-dev python3.9-distutils && \
  rm /usr/bin/python && ln -s /usr/bin/python3.7 /usr/bin/python && \
  rm /usr/bin/python3 && ln -s /usr/bin/python3.7 /usr/bin/python3


WORKDIR /home
RUN wget https://files.pythonhosted.org/packages/a7/e0/30642b9c2df516506d40b563b0cbd080c49c6b3f11a70b4c7a670f13a78b/setuptools-50.3.2.zip && apt-get -y install unzip && unzip setuptools-50.3.2.zip
WORKDIR /home/setuptools-50.3.2
RUN python3.9 setup.py build && python3.9 setup.py install && \
  python3.8 setup.py build && python3.8 setup.py install && \
  python3.7 setup.py build && python3.7 setup.py install

WORKDIR /home
RUN wget https://files.pythonhosted.org/packages/28/af/2c76c8aa46ccdf7578b83d97a11a2d1858794d4be4a1610ade0d30182e8b/pip-20.0.1.tar.gz && tar -zxvf pip-20.0.1.tar.gz
WORKDIR pip-20.0.1

RUN python3.9 setup.py install && \
  python3.8 setup.py install && \
  python3.7 setup.py install

WORKDIR /home
RUN rm setuptools-50.3.2.zip pip-20.0.1.tar.gz && \
    rm -r setuptools-50.3.2 pip-20.0.1
RUN rm /usr/local/bin/pip && ln -s /usr/local/bin/pip3.7 /usr/local/bin/pip && \
  rm /usr/local/bin/pip3 && ln -s /usr/local/bin/pip3.7 /usr/local/bin/pip3


# remove them when apt-get support 2.27 and higher version
RUN wget -q https://ftp.gnu.org/gnu/binutils/binutils-2.33.1.tar.gz && \ 
    tar -xzf binutils-2.33.1.tar.gz && \ 
    cd binutils-2.33.1 && \
    ./configure && make -j && make install && cd .. && rm -rf binutils-2.33.1 binutils-2.33.1.tar.gz


# Install Go and glide
#RUN wget --no-check-certificate -qO- https://paddle-ci.gz.bcebos.com/go1.17.2.linux-amd64.tar.gz | \
#    tar -xz -C /usr/local && \
#    mkdir /root/gopath && \
#    mkdir /root/gopath/bin && \
#    mkdir /root/gopath/src
#ENV GOROOT=/usr/local/go GOPATH=/root/gopath
## should not be in the same line with GOROOT definition, otherwise docker build could not find GOROOT.
#ENV PATH=${PATH}:${GOROOT}/bin:${GOPATH}/bin
# install glide
RUN curl -s -q https://glide.sh/get | sh

# git credential to skip password typing
RUN git config --global credential.helper store

# Fix locales to en_US.UTF-8
RUN localedef -i en_US -f UTF-8 en_US.UTF-8

RUN pip3.7 --no-cache-dir install ipython==5.3.0 && \
    pip3.7 --no-cache-dir install ipykernel==4.6.0 wheel && \
    pip3.8 --no-cache-dir install ipython==5.3.0 && \
    pip3.8 --no-cache-dir install ipykernel==4.6.0 wheel && \
    pip3.9 --no-cache-dir install ipython==5.3.0 && \
    pip3.9 --no-cache-dir install ipykernel==4.6.0 wheel

#For docstring checker
RUN pip3.7 --no-cache-dir install pytest astroid isort && \
    pip3.8 --no-cache-dir install pytest astroid isort && \
    pip3.9 --no-cache-dir install pytest astroid isort

#For pre-commit
RUN pip3.7 --no-cache-dir install --upgrade pip && \
    pip3.8 --no-cache-dir install --upgrade pip && \
    pip3.9 --no-cache-dir install --upgrade pip

RUN pip3.7 --no-cache-dir install pre-commit==2.17.0 pylint==2.12.0 && \
    pip3.8 --no-cache-dir install pre-commit==2.17.0 pylint==2.12.0 && \
    pip3.9 --no-cache-dir install pre-commit==2.17.0 pylint==2.12.0 && \
    pip3.7 --no-cache-dir install cpplint==1.6.0 clang-format==13.0.0 && \
    pip3.8 --no-cache-dir install cpplint==1.6.0 clang-format==13.0.0 && \
    pip3.9 --no-cache-dir install cpplint==1.6.0 clang-format==13.0.0

COPY ./requirements.txt /root/
RUN pip3.7 --no-cache-dir install -r /root/requirements.txt && \
    pip3.8 --no-cache-dir install -r /root/requirements.txt && \
    pip3.9 --no-cache-dir install -r /root/requirements.txt

# Configure OpenSSH server. c.f. https://docs.docker.com/engine/examples/running_ssh_service
#RUN mkdir /var/run/sshd && echo 'root:root' | chpasswd && sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config && sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
#CMD source ~/.bashrc

# ccache 3.7.9
RUN wget https://paddle-ci.gz.bcebos.com/ccache-3.7.9.tar.gz && \
    tar xf ccache-3.7.9.tar.gz && mkdir /usr/local/ccache-3.7.9 && cd ccache-3.7.9 && \
    ./configure -prefix=/usr/local/ccache-3.7.9 && \
    make -j8 && make install && \
    ln -s /usr/local/ccache-3.7.9/bin/ccache /usr/local/bin/ccache && \
    cd ../ && rm -rf ccache-3.7.9 ccache-3.7.9.tar.gz

# clang+llvm 3.8.0
RUN wget https://paddle-ci.cdn.bcebos.com/clang+llvm-3.8.0-x86_64-linux-gnu-ubuntu-16.04.tar.xz && \ 
    tar xf clang+llvm-3.8.0-x86_64-linux-gnu-ubuntu-16.04.tar.xz && cd clang+llvm-3.8.0-x86_64-linux-gnu-ubuntu-16.04 && \
    cp -rn * /usr/local && cd .. && rm -rf clang+llvm-3.8.0-x86_64-linux-gnu-ubuntu-16.04 && rm -rf clang+llvm-3.8.0-x86_64-linux-gnu-ubuntu-16.04.tar.xz 

EXPOSE 22
