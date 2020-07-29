# docker build raspberry_pi -t vineyard_rpi --build-arg http_proxy=$http_proxy --build-arg https_proxy=$https_proxy
# docker run -v $(pwd):/mnt vineyard_rpi cp opencv.tar.gz /mnt
FROM debian:buster

RUN dpkg --add-architecture armhf && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    crossbuild-essential-armhf \
    git \
    wget \
    cmake \
    libusb-1.0-0-dev:armhf \
    libgtk-3-dev:armhf \
    libavcodec-dev:armhf \
    libavformat-dev:armhf \
    libswscale-dev:armhf \
    libgstreamer1.0-dev:armhf \
    libgstreamer-plugins-base1.0-dev:armhf \
    libpython3-dev:armhf \
    python3-pip \
    python-minimal \
    python-argparse \
    python3-numpy

# Clone repositories
RUN git clone https://github.com/opencv/opencv/ --depth 1
RUN git clone https://github.com/openvinotoolkit/openvino/ --depth 1 --recursive

# Build Inference Engine
RUN cd /openvino && \
    mkdir build && cd build && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_TOOLCHAIN_FILE="../cmake/arm.toolchain.cmake" \
        -DTHREADS_PTHREAD_ARG="-pthread" \
        -DENABLE_SSE42=OFF \
        -DTHREADING=SEQ \
        -DENABLE_OPENCV=OFF \
        -DENABLE_GNA=OFF .. && make --jobs=$(nproc --all)

# Build OpenCV
RUN cd /opencv && \
    mkdir /opencv_install && mkdir build && cd build && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX="/opencv_install" \
        -DOPENCV_CONFIG_INSTALL_PATH="cmake" \
        -DCMAKE_TOOLCHAIN_FILE="../platforms/linux/arm-gnueabi.toolchain.cmake" \
        -DWITH_IPP=OFF \
        -DBUILD_TESTS=OFF \
        -DBUILD_PERF_TESTS=OFF \
        -DOPENCV_ENABLE_PKG_CONFIG=ON \
        # Python flags
        -DPKG_CONFIG_EXECUTABLE="/usr/bin/arm-linux-gnueabihf-pkg-config" \
        -DPYTHON3_INCLUDE_PATH="/usr/include/python3.7" \
        -DPYTHON3_NUMPY_INCLUDE_DIRS="/usr/local/lib/python3.7/dist-packages/numpy/core/include" \
        -DPYTHON3_CVPY_SUFFIX=".cpython-37m-arm-linux-gnueabihf.so" \
        -DENABLE_NEON=ON \
        -DCPU_BASELINE="NEON" \
        # Inference Engine related flags
        -DWITH_INF_ENGINE=ON \
        -DInferenceEngine_DIR=/openvino/build \
        -Dngraph_DIR=/openvino/build/ngraph \
        -DINF_ENGINE_RELEASE=2021010000 \
        -DBUILD_LIST=dnn,python3 \
        .. && make --jobs=$(nproc --all) && make install

RUN mv /openvino/bin/armv7l/Release/lib/*.so /opencv_install/lib/ && \
    mv /openvino/bin/armv7l/Release/lib/*.mvcmd /opencv_install/lib/ && \
    mv /openvino/bin/armv7l/Release/lib/plugins.xml /opencv_install/lib/ && \
    tar -cvzf opencv_debian10_armhf.tar.gz /opencv_install
