FROM ubuntu:20.04


# Install git and other dependencies
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get -y update && apt-get -y upgrade
RUN apt-get -y install wget
RUN apt-get -y install git
RUN apt-get -y install g++
RUN apt-get -y install cmake
RUN apt-get -y install gfortran libblas-dev liblapack-dev # Dependency of suitesparse
RUN apt-get -y install freeglut3 freeglut3-dev libglew-dev libglu1-mesa libglu1-mesa-dev libgl1-mesa-glx libgl1-mesa-dev # Dependency of pangolin

# Make directory for deps and lib
RUN mkdir repos


# ====== <Eigen3.2> ======
WORKDIR /repos
RUN git clone --depth 1 --branch 3.2.10 https://gitlab.com/libeigen/eigen.git eigen3

# Build eigen
WORKDIR eigen3
RUN mkdir build
WORKDIR build
RUN cmake .. -DBUILD_SHARED_LIBS=OFF  -DCMAKE_BUILD_TYPE=Release  "-DCMAKE_INSTALL_PREFIX:PATH=/deps/lsdslam/eigen3/"
RUN make -j2 && make install


# ====== <OpenCV 3.4.3> ======
WORKDIR /repos
RUN git clone "https://github.com/opencv/opencv.git" opencv
RUN git clone "https://github.com/opencv/opencv_contrib.git" opencv_contrib
WORKDIR /repos/opencv
RUN git checkout 3.4.3
WORKDIR /repos/opencv_contrib
RUN git checkout 3.4.3

# Build opencv
WORKDIR /repos/opencv
RUN mkdir build_dir
WORKDIR build_dir
RUN cmake -D CMAKE_BUILD_TYPE=RELEASE  -D CMAKE_INSTALL_PREFIX=/deps/lsdslam/opencv    -DCMAKE_CXX_FLAGS="-Wno-error=address"             \
	-DWITH_GSTREAMER=OFF -DWITH_FFMPEG=OFF -DBUILD_PERF_TESTS=OFF  -D WITH_OPENCL=OFF -D BUILD_WITH_DEBUG_INFO=OFF  -D WITH_1394=OFF                \
	-D BUILD_TESTS=OFF  -D WITH_TBB=OFF  -D WITH_V4L=OFF  -D WITH_OPENGL=OFF -D BUILD_opencv_gpu=OFF    \
	  -D BUILD_opencv_java=OFF -D WITH_CUDA=OFF -DWITH_GTK=ON   -D BUILD_opencv_ml=ON  -D BUILD_opencv_videostab=OFF             \
	   -D BUILD_opencv_ts=OFF    -D BUILD_opencv_photo=ON  -D BUILD_opencv_video=ON -D BUILD_opencv_stitching=OFF -DOPENCV_EXTRA_MODULES_PATH=/repos/opencv_contrib/modules -DENABLE_PRECOMPILED_HEADERS=OFF .. > /repos/opencv/build_dir/opencv_cmake.log
RUN cat /repos/opencv/build_dir/opencv_cmake.log
RUN make -j2 && make install


# ====== <Pangolin> ======
# Required dependencies: opengl, libglew-dev, eigen3
WORKDIR /repos
RUN git clone "https://github.com/stevenlovegrove/Pangolin.git" pangolin
WORKDIR pangolin
RUN git checkout c2a6ef524401945b493f14f8b5b8aa76cc7d71a9

# Build pangolin
RUN mkdir build
WORKDIR build
RUN cmake .. -DBUILD_PANGOLIN_VIDEO=OFF  "-DCMAKE_INSTALL_PREFIX:PATH=/deps/lsdslam/pangolin"  -DEIGEN_INCLUDE_DIR=/deps/lsdslam/eigen3/include/eigen3
RUN make -j2 && make install


# ====== <Suitesparse v4.3.1> ======
# Required dependencies: gfortran libblas-dev liblapack-dev
WORKDIR /repos
RUN git clone "https://github.com/jluttine/suitesparse.git"
WORKDIR suitesparse
RUN git checkout v4.3.1

# Build suitesparse
RUN make
RUN mkdir /deps/lsdslam/suitesparse/include -p
RUN mkdir /deps/lsdslam/suitesparse/lib -p
RUN make install INSTALL_LIB=/deps/lsdslam/suitesparse/lib INSTALL_INCLUDE=/deps/lsdslam/suitesparse/include


# ====== <g2o> ======
WORKDIR /repos
RUN git clone "https://github.com/RainerKuemmerle/g2o" g2o
WORKDIR g2o
RUN git checkout 1b118ac2ed2055c4016c3b7cbd710225ed1651af

# Build g2o
# Required dependencies: suitesparse, eigen3
RUN mkdir build
WORKDIR build
RUN cmake .. "-DCMAKE_INSTALL_PREFIX:PATH=/deps/lsdslam/g2o/" \
	-DCHOLMOD_LIBRARY=/deps/lsdslam/suitesparse/lib/libcholmod.a -DCHOLMOD_FOUND=TRUE -DCHOLMOD_INCLUDE_DIR=/deps/lsdslam/suitesparse/include/ -DCHOLMOD_LIBRARIES=/deps/lsdslam/suitesparse/lib \
	-DCSPARSE_INCLUDE_DIR=/deps/lsdslam/suitesparse/include/ -DCSPARSE_LIBRARY=/deps/lsdslam/suitesparse/lib/libcxsparse.a \
	-DEIGEN3_INCLUDE_DIR=/deps/lsdslam/eigen3/include/eigen3
RUN make -j2 && make install


# ====== <SLAMBench> ======
WORKDIR /repos
RUN git clone --branch fake_slambench https://github.com/nikolaradulov/slambench.git slambench

WORKDIR /deps/lsdslam
RUN mkdir wrapper
WORKDIR /repos/slambench/docker/wrapper
RUN mv libslambench-c-wrapper.a /deps/lsdslam/wrapper
RUN mv null.cpp /deps/lsdslam/wrapper

# ====== <LSD-SLAM> ======
WORKDIR /repos
RUN git clone --recursive --branch cpp https://github.com/yuhaozhang7/lsd_slam.git lsdslam

WORKDIR lsdslam
RUN mkdir build
WORKDIR build
RUN cmake .. "-DCMAKE_INSTALL_PREFIX:PATH=/deps/lsdslam/" \
	-DWRAPPER_PATH=/deps/lsdslam/wrapper/ \
	-DEIGEN3_INCLUDE_DIR=/deps/lsdslam/eigen3/include/eigen3/ \
	-DPangolin_INCLUDE_DIR=/deps/lsdslam/pangolin/include/ \
	-DG2O_INCLUDE_DIR=/deps/lsdslam/g2o/include/ \
	-DCSPARSE_INCLUDE_DIR=/deps/lsdslam/suitesparse/include/ \
	-DCHOLMOD_INCLUDE_DIR=/deps/lsdslam/suitesparse/lib/ \
	-DOpenCV_INCLUDE_DIRS=/deps/lsdslam/opencv/include/ \
	-DSLAMBENCH_INCLUDE_DIR=/repos/slambench/framework/shared/include/ \
	-DG2O_TYPES_SIM3=/deps/lsdslam/g2o/lib/libg2o_types_sim3.so \
	-DG2O_TYPES_SBA=/deps/lsdslam/g2o/lib/libg2o_types_sba.so \
	-DG2O_CORE_LIBRARY=/deps/lsdslam/g2o/lib/libg2o_core.so \
	-DG2O_SOLVER_CSPARSE=/deps/lsdslam/g2o/lib/libg2o_solver_csparse.so \
	-DG2O_SOLVER_CSPARSE_EXTENSION=/deps/lsdslam/g2o/lib/libg2o_csparse_extension.so \
	-DG2O_STUFF_LIBRARY=/deps/lsdslam/g2o/lib/libg2o_stuff.so \
	-DCSPARSE_LIBRARY=/deps/lsdslam/suitesparse/lib/libcxsparse.a
RUN make && make install


# Script for export lib path
WORKDIR /deps/lsdslam
RUN echo "export LD_LIBRARY_PATH=\"/deps/lsdslam/lib:$LD_LIBRARY_PATH\"" > libpath.txt

