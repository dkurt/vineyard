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
        -DBUILD_TESTS=ON \
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
        -DBUILD_LIST=ts,dnn,python3 \
        .. && make --jobs=$(nproc --all) && make install

# Prepare a package
RUN mkdir openvino_debian10_armhf && cd openvino_debian10_armhf && \
    # Create folders structure
    mkdir bin && \
    mkdir -p deployment_tools/inference_engine/lib/ && \
    mkdir -p deployment_tools/ngraph/lib && \
    mkdir -p python && \
    mkdir install_dependencies && \
    ln -s deployment_tools/inference_engine inference_engine && \
    # Copy the files
    cp -r /opencv_install opencv && \
    cp /openvino/scripts/setupvars/setupvars.sh bin && \
    cp /openvino/scripts/install_dependencies/install_NCS_udev_rules.sh install_dependencies && \
    cp /openvino/scripts/install_dependencies/install_openvino_dependencies.sh install_dependencies && \
    cp -r /opencv_install/lib/python3.7/dist-packages python/python3 && \
    cp -r /openvino/inference-engine/include deployment_tools/inference_engine && \
    rm /openvino/bin/armv7l/Release/lib/*.a && \
    cp -r /openvino/bin/armv7l/Release/lib deployment_tools/inference_engine/lib/armv7l && \
    mv deployment_tools/inference_engine/lib/armv7l/libngraph* deployment_tools/ngraph/lib && \
    mv deployment_tools/inference_engine/lib/armv7l/libinterpreter_backend.so deployment_tools/ngraph/lib

RUN tar -cvzf openvino_debian10_armhf.tar.gz openvino_debian10_armhf/*
RUN tar -cvzf opencv_debian10_armhf_tests.tar.gz /opencv/build/bin/opencv_test*
