FROM ubuntu:18.04 as build_base

# Set environment variables.
## Set non-interactive to prevent asking for user inputs blocking image creation.
ENV DEBIAN_FRONTEND=noninteractive
## Set timezone as it is required by some packages.
ENV TZ=Asia/Shanghai

# 换源
# RUN sed -i 's/archive.ubuntu.com/mirrors.cqu.edu.cn/g' /etc/apt/sources.list \
#     && sed -i 's/security.ubuntu.com/mirrors.cqu.edu.cn/g' /etc/apt/sources.list
RUN sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list \
    && sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list

# 设置timezone,还有认证的更新
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    ca-certificates \
    tzdata \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && rm -rf /var/lib/apt/lists/*
# && dpkg-reconfigure --frontend noninteractive tzdata \

FROM build_base as build_env

#安装编译所需工具,
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    wget \
    git \
    build-essential

FROM build_env as gcc_build
#由于apt安装gcc7.3.0报错，所以直接下载源码编译安装
#安装GCC的依赖:https://gcc.gnu.org/install/prerequisites.html
# 源码里面有这个脚本：./contrib/download_prerequisites
#下载gcc7.3.0源码
# https://gcc.gnu.org/git/?p=gcc.git;a=commit;h=6184085fc664265dc78fd1ad4204c0cbef202628

# 设置代理
# ARG http_proxy=http://172.17.0.1:7890
# ARG https_proxy=http://172.17.0.1:7890
# ARG http_proxy
# ARG https_proxy

# RUN git clone --depth 1 -b releases/gcc-7.3.0 https://gcc.gnu.org/git/gcc.git \
# RUN git clone --depth 1 -b releases/gcc-7.3.0 https://mirrors.tuna.tsinghua.edu.cn/git/gcc.git
# RUN cd gcc \
RUN wget https://ftp.gnu.org/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.xz \
    && tar -xvf gcc-7.3.0.tar.xz \
    && cd gcc-7.3.0 \
    && ./contrib/download_prerequisites \
    && ./configure --prefix=/gcc_install/ --enable-languages=c,c++ --disable-multilib \
    && make \
    && make install \
    && cd ..

FROM build_env as cmake_download
#下载cmake3.22.0官方二进制文件解压
RUN wget https://cmake.org/files/v3.22/cmake-3.22.0-linux-x86_64.tar.gz \
    && mkdir /cmake_install/ \
    && tar -zxvf cmake-3.22.0-linux-x86_64.tar.gz -C /cmake_install/ --strip-components 1 

FROM build_base as cpp_env
#复制gcc的安装目录
COPY --from=gcc_build /gcc_install/ /

#安装make 4.1
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    make=4.1-9.1ubuntu1 \
    && rm -rf /var/lib/apt/lists/*

#复制cmake安装
COPY --from=cmake_download /cmake_install/ /

FROM build_env as python_build
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    zlib1g-dev \
    libncurses5-dev \
    libgdbm-dev \
    libnss3-dev \
    libssl-dev \
    libsqlite3-dev \
    libreadline-dev \
    libffi-dev \
    libbz2-dev

# 下载源码，编译安装
# RUN wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz \
RUN wget https://mirrors.huaweicloud.com/python/3.7.3/Python-3.7.3.tgz \
    && tar -zxvf Python-3.7.3.tgz \
    && cd Python-3.7.3 \
    && ./configure --prefix=/python_build/ \
    && make \
    && make install \
    && cd ..

FROM cpp_env as python_env
#复制python的安装目录
COPY --from=python_build /python_build/ /

# 安装numpy
ENV PYTHONHOME=/
RUN /bin/python3 -m pip install --no-cache-dir numpy==1.19.4 -i https://mirrors.huaweicloud.com/repository/pypi/simple

FROM build_env as java_build
# 下载jdk
RUN wget https://mirrors.huaweicloud.com/java/jdk/8u202-b08/jdk-8u202-linux-x64.tar.gz \
    && mkdir /jdk_install/ \
    && tar -zxvf jdk-8u202-linux-x64.tar.gz -C /jdk_install/ --strip-components 1 \
    && find /jdk_install/ -maxdepth 1 -type f -delete

FROM python_env as java_env
#复制jdk的安装目录
COPY --from=java_build /jdk_install/ /

FROM java_env as judge_env
ARG username=judge
RUN useradd -ms /bin/bash ${username}
USER ${username}
WORKDIR /home/${username}/