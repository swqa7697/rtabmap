# Image: introlab3it/rtabmap:jammy-deps

FROM ubuntu:22.04

ARG TARGETPLATFORM
ENV TARGETPLATFORM=${TARGETPLATFORM:-linux/amd64}
RUN echo "I am building for $TARGETPLATFORM"

# Install ROS2
RUN apt update && \
    apt install software-properties-common -y && \
    add-apt-repository universe && \
    apt update && \
    apt install curl -y && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt-get clean && rm -rf /var/lib/apt/lists/

ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies
RUN apt-get update && \
    apt upgrade -y && \
    apt-get install -y git libtbb-dev ros-humble-ros-base ros-dev-tools && \
    apt-get clean && rm -rf /var/lib/apt/lists/

RUN apt-get update && \
    apt upgrade -y && \
    apt-get install -y libfreenect-dev ros-humble-rtabmap-ros && \
    apt-get remove -y ros-humble-rtabmap* ros-humble-grid-map-core && \
    apt-get clean && rm -rf /var/lib/apt/lists/

WORKDIR /root/

# PDAL
RUN apt-get update && apt-get install -y libpdal-dev && \
    apt-get clean && rm -rf /var/lib/apt/lists/

# RealSense2
RUN apt-get update && apt-get install -y ros-humble-librealsense2 && \
    apt-get clean && rm -rf /var/lib/apt/lists/

# libfreenect2 
RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then echo "Installing libfreenect2..." && \
    apt-get update && apt-get install -y mesa-utils xserver-xorg-video-all libusb-1.0-0-dev libturbojpeg0-dev libglfw3-dev && \
    apt-get clean && rm -rf /var/lib/apt/lists/ && \
    git clone https://github.com/OpenKinect/libfreenect2 && \
    cd libfreenect2 && \
    mkdir build && \
    cd build && \
    cmake .. && \
    make -j2 && \
    make install && \
    cd && \
    rm -r libfreenect2; fi

# zed open capture
RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then echo "Installing zed-open-capture..." && \
    apt-get update && apt install -y libusb-1.0-0-dev libhidapi-libusb0 libhidapi-dev wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/ && \
    git clone https://github.com/stereolabs/zed-open-capture.git && \
    cd zed-open-capture && \
    mkdir build && \
    cd build && \
    cmake .. && \
    make -j2 && \
    make install && \
    cd && \
    rm -r zed-open-capture; fi
    
RUN git clone --branch 4.5.4 https://github.com/opencv/opencv.git && \
    git clone --branch 4.5.4 https://github.com/opencv/opencv_contrib.git && \
    cd opencv && \
    mkdir build && \
    cd build && \
    cmake -DWITH_TBB=ON -DWITH_OPENMP=ON -DBUILD_opencv_python3=OFF -DBUILD_opencv_python_bindings_generator=OFF -DBUILD_opencv_python_tests=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_TESTS=OFF -DOPENCV_ENABLE_NONFREE=ON -DOPENCV_EXTRA_MODULES_PATH=/root/opencv_contrib/modules .. && \
    make -j2 && \
    make install && \
    cd ../.. && \
    rm -rf opencv opencv_contrib

#commit Aug 6 2020
RUN apt-get update && apt install -y wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/
RUN git clone https://github.com/laurentkneip/opengv.git && \
    cd opengv && \
    git checkout 91f4b19c73450833a40e463ad3648aae80b3a7f3 && \
    wget https://gist.githubusercontent.com/matlabbe/a412cf7c4627253874f81a00745a7fbb/raw/accc3acf465d1ffd0304a46b17741f62d4d354ef/opengv_disable_march_native.patch && \
    git apply opengv_disable_march_native.patch && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release .. && \
    make -j2 && \
    make install && \
    cd && \
    rm -r opengv

RUN rm /bin/sh && ln -s /bin/bash /bin/sh

RUN echo -e '#!/bin/bash\nset -e\n\n# setup ros2 environment\nsource "/opt/ros/humble/setup.bash" --\nexec "$@"' > /ros_entrypoint.sh
RUN chmod +x /ros_entrypoint.sh
ENTRYPOINT [ "/ros_entrypoint.sh" ]

# ros2 seems not sourcing by default its multi-arch folders
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/ros/humble/lib/x86_64-linux-gnu:/opt/ros/humble/lib/aarch64-linux-gnu

# for jetson (https://github.com/introlab/rtabmap/issues/776)
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/aarch64-linux-gnu/tegra

