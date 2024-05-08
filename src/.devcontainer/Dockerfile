FROM osrf/ros:noetic-desktop-full


LABEL author = "weison_pan"
LABEL email = "weisonweileen@gmail.com"


# Change source and make sure everything is up to date
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list &&\
    sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list &&\
    apt-get update && \
    apt-get -y upgrade && \
    rm -rf /var/lib/apt/lists/*
# 其实 upgrade 命令有必要吗，应该是有的，防止基础镜像的包不够新

# root environment，to build the pcl environment
USER root

# Eigen patch (https://eigen.tuxfamily.org/bz/show_bug.cgi?id=1462) to fix issue mentioned
# in https://github.com/PointCloudLibrary/pcl/issues/3729 is available in Eigen 3.3.7.
# Not needed from 20.04 since it is the default version from apt
ARG EIGEN_MINIMUM_VERSION=3.3.7

# See https://www.optonic.com/support/download/ensenso-sdk/archiv/ for available versions
ARG ENSENSOSDK_VERSION=3.2.489

# # See https://github.com/IntelRealSense/librealsense/tags for available tags of releases
# ARG REALSENSE_VERSION=2.50.0

# Check https://packages.ubuntu.com/search?suite=all&arch=any&searchon=names&keywords=libvtk%20qt-dev
# for available packages for chosen UBUNTU_VERSION
ARG VTK_VERSION=6

# Use the latest version of CMake by adding the Kitware repository if true,
# otherwise use the default version of CMake of the system.
ARG USE_LATEST_CMAKE=false

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get -V install -y \
      build-essential \
      clang \
      clang-tidy \
      libbenchmark-dev \
      libblas-dev \
      libboost-serialization-dev \
      libboost-filesystem-dev \
      libboost-iostreams-dev \
      libboost-system-dev \
      libflann-dev \
      libglew-dev \
      libgtest-dev \
      libomp-dev \
      libopenni-dev \
      libopenni2-dev \
      libpcap-dev \
      libproj-dev \
      libqhull-dev \
      libqt5opengl5-dev \
      libusb-1.0-0-dev \
      libvtk${VTK_VERSION}-dev \
      libvtk${VTK_VERSION}-qt-dev \
      lsb-release \
      qtbase5-dev \
      software-properties-common \
      wget \
      xvfb \
      vim \
      meshlab \
      python3-pip \
      git \
  && if [ "$USE_LATEST_CMAKE" = true ] ; then \
      cmake_ubuntu_version=$(lsb_release -cs) ; \
      if ! wget -q --method=HEAD "https://apt.kitware.com/ubuntu/dists/$cmake_ubuntu_version/Release"; then \
        ubuntu_version=$(lsb_release -sr) ; \
        if dpkg --compare-versions ${ubuntu_version} ge 22.04; then \
          cmake_ubuntu_version="jammy" ; \
        elif dpkg --compare-versions ${ubuntu_version} ge 20.04; then \
          cmake_ubuntu_version="focal" ; \
        else \
          cmake_ubuntu_version="bionic" ; \
        fi ; \
      fi ; \
      wget -qO - https://apt.kitware.com/kitware-archive.sh | bash -s -- --release $cmake_ubuntu_version ; \
      apt-get update ; \
    fi \
 && apt-get -V install -y cmake \
 && if [ "$(lsb_release -sr)" = "18.04" ]; then \
       apt-get -V install -y nvidia-cuda-toolkit g++-6 ; \
     else \
       apt-get -V install -y nvidia-cuda-toolkit-gcc ; \
     fi \
 && rm -rf /var/lib/apt/lists/* rm -rf /usr/lib/x86_64-linux-gnu/libnvidia* /usr/lib/x86_64-linux-gnu/libcuda*

# Use libeigen3-dev if it meets the minimal version.
# In most cases libeigen3-dev is already installed as a dependency of libvtk6-dev & libvtk7-dev, but not in every case (libvtk9 doesn't seem to have this dependency),
# so install it from apt if the version is sufficient.
RUN if dpkg --compare-versions $(apt-cache show --no-all-versions libeigen3-dev | grep -P -o 'Version:\s*\K.*') ge ${EIGEN_MINIMUM_VERSION}; then \
      apt-get -V install -y libeigen3-dev ; \
    else \
      wget -qO- https://gitlab.com/libeigen/eigen/-/archive/${EIGEN_MINIMUM_VERSION}/eigen-${EIGEN_MINIMUM_VERSION}.tar.gz | tar xz \
      && cd eigen-${EIGEN_MINIMUM_VERSION} \
      && mkdir build \
      && cd build \
      && cmake .. \
      && make -j$(nproc) install \
      && cd ../.. \
      && rm -rf eigen-${EIGEN_MINIMUM_VERSION}/ ; \
    fi

RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple \
  && pip install open3d

    #这两个sdk都不需要，要的时候再安装

# RUN wget -qO- https://github.com/IntelRealSense/librealsense/archive/v${REALSENSE_VERSION}.tar.gz | tar xz \
#  && cd librealsense-${REALSENSE_VERSION} \
#  && mkdir build \
#  && cd build \
#  && cmake .. -DBUILD_EXAMPLES=OFF -DBUILD_GRAPHICAL_EXAMPLES=OFF \
#  && make -j$(nproc) install \
#  && cd ../.. \
#  && rm -rf librealsense-${REALSENSE_VERSION}

# RUN wget -qO ensenso.deb https://download.ensenso.com/s/ensensosdk/download?files=ensenso-sdk-${ENSENSOSDK_VERSION}-x64.deb \
#  && dpkg -i ensenso.deb \
#  && rm ensenso.deb



# user environment
ARG USERNAME=USERNAME
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user, let the user password-free
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME 
    # && sudo usermod -aG dialout $USER         
    # let the user be able to deal with /dev/ttyACM0

## change bashrc and ADD path 
USER $USERNAME
RUN echo 'source /opt/ros/${ROS_DISTRO}/setup.bash'  >> ~/.bashrc && \
    #for ros2
    # echo 'source /home/ws/install/setup.bash' >> ~/.bashrc  && \  
    #for ros
    echo 'source /home/ws/devel/setup.bash' >> ~/.bashrc  && \ 
    echo 'export PATH=${PATH}:/home/${USERNAME}/.local/bin' >> ~/.bashrc

# RUN 
# 后面的不能省，不然终端会变成dash
ENV SHELL /bin/bash
# ********************************************************
# * Anything else you want to do like clean up goes here *
# ********************************************************


# [Optional] Set the default user. Omit if you want to keep the default as root.
CMD ["/bin/bash"]