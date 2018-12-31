FROM ubuntu:16.04
MAINTAINER Parker Lusk <parkerclusk@gmail.com>

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    vim wget git mercurial \
    build-essential \
    cmake \
    libboost-all-dev \
    libeigen3-dev \
    libwxgtk3.0-dev \
    libopenscenegraph-dev \
    libsuitesparse-dev \
    libgoogle-glog-dev \
    libatlas-base-dev
    # libtbb-dev
  #apt-get clean && rm -rf /var/lib/apt/lists/*

# Use the home directory as our working space
WORKDIR /root

#
# Install Eigen 3.3.3
#

ENV EIGEN3_VERSION="3.3.3"
RUN hg clone https://bitbucket.org/eigen/eigen --insecure
RUN cd eigen && hg update ${EIGEN3_VERSION}
RUN cd eigen && mkdir build && cd build \
    && cmake .. && make && make install

#
# Install Ceres Solver
#

ENV CERES_VERSION="1.13.0"
RUN wget http://ceres-solver.org/ceres-solver-${CERES_VERSION}.tar.gz -O- | tar xz
RUN cd ceres-solver-${CERES_VERSION} && mkdir build && cd build \
    && cmake -DBUILD_EXAMPLES=OFF -DBUILD_TESTING=OFF .. && make -j${nproc} && make install

# Ceres 1.14.0 can be used after making the changes in the commits associated
# with this issue: https://github.com/openMVG/openMVG/issues/1323

#
# Install Regard3D's modified version of OpenMVG
#

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    p7zip-full \
    libflann-dev

# Regard3D v0.9.5 uses OpenMVG v1.3. The modified source of OpenMVG v1.3
# is found on SourceForge with the downloads associated with Regard3D v0.9.4
ENV OPENMVG_R3D="openMVG-1.3_r3d"
RUN wget https://sourceforge.net/projects/regard3d/files/Regard3D/0.9.4/${OPENMVG_R3D}.7z --no-check-certificate \
    && 7z x ${OPENMVG_R3D}.7z

# This should really be removed from CMakeLists.txt so that LAPACK-enabled
# Eigen doesn't include its complex.h, which defines the macro 'I' and casues
# a compiler error for variables named 'I'.
# For example, see: https://github.com/libigl/libigl/issues/651
RUN sed -i '/IF(MSVC)/ d' ${OPENMVG_R3D}/src/CMakeLists.txt

RUN cd ${OPENMVG_R3D} && mkdir build && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release -DOpenMVG_BUILD_EXAMPLES=OFF -DOpenMVG_BUILD_DOC=OFF -DOpenMVG_BUILD_SOFTWARES=OFF ../src/ \
    && make -j${nproc} && make install

# Install extra headers that were excluded because OpenMVG_BUILD_SOFTWARES=OFF
# see: https://github.com/rhiestan/Regard3D/issues/3#issuecomment-360791551
RUN mkdir /usr/local/include/openMVG/software && cd /usr/local/include/openMVG/software \
    && mkdir SfM  && mkdir SfMViewer \
    && cp /root/${OPENMVG_R3D}/src/software/SfM/InterfaceMVS.h SfM/ \
    && cp /root/${OPENMVG_R3D}/src/software/SfM/SfMIOHelper.hpp SfM/ \
    && cp /root/${OPENMVG_R3D}/src/software/SfM/SfMPlyHelper.hpp SfM/ \
    && cp /root/${OPENMVG_R3D}/src/software/SfMViewer/document.h SfMViewer/

#
# Install OpenCV 3.4
#

RUN apt-get install -y software-properties-common
RUN add-apt-repository -y ppa:timsc/opencv-3.4
RUN apt-get update -y && apt-get install -y libopencv-dev libopencv-shape*

#
# Install Regard3D
#

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    libassimp-dev \
    libopenblas-dev \
    liblapacke-dev
RUN update-ca-certificates

ENV R3D_VERSION="0.9.5"
RUN git clone https://github.com/rhiestan/Regard3D \
    && cd Regard3D && git checkout v${R3D_VERSION} \
    && mkdir build

# Patch document.h to account for namespaces
# see: https://github.com/rhiestan/Regard3D/issues/3#issuecomment-368671962
RUN sed -i 's/ PinholeCamera/ cameras::PinholeCamera/g' /usr/local/include/openMVG/software/SfMViewer/document.h
RUN sed -i 's/!openMVG::load/!openMVG::cameras::load/g' /usr/local/include/openMVG/software/SfMViewer/document.h

# Patch R3D for linux build
COPY *.patch ./
RUN cd Regard3D && git apply ../*.patch

# Create an empty opencv dir to appease CMake ... why?
# empty otherwise flann chooses wrong (opencv's) header
RUN mkdir /usr/include/opencv

RUN cd Regard3D/build \
    && cmake -DCMAKE_BUILD_TYPE=Release \
             -DR3D_USE_BLAS=ON \
             ../src \
    && make -j${nproc}

#
# Add OpenMVG Sensor Database
#

RUN git clone https://github.com/openMVG/CameraSensorSizeDatabase \
    && cd Regard3D/build \
    && ln -s ../../CameraSensorSizeDatabase/sensor_database.csv

# libcanberra-gtk-module