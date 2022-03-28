# Installation documentation for KNN+Caffe Object recognition

The following documentation assumes that the target system is a Ubuntu 20.04 with ROS Noetic.

```
# Step 1)
# Install RoboSherlock based on these instructions: http://robosherlock.org/installation.html
# This will give you the basic RoboSherlock install.
----------------
# Step 2
# Set up Object detection with Caffe (NNs)

# Clone rs_resources to your workspace
cd YOUR_CATKIN_WORKSPACE/src
git clone https://github.com/RoboSherlock/rs_resources
git clone https://github.com/RoboSherlock/rs_classifiers
git clone https://github.com/ros-perception/image_transport_plugins.git -b noetic-devel # if you haven't done that already in your RS basic installation

# Caffe install
sudo apt install cmake git unzip libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev libhdf5-dev protobuf-compiler libatlas-base-dev  libgflags-dev libboost-all-dev liblmdb-dev libgoogle-glog-dev libopenblas-dev
cd
git clone https://github.com/RoboSherlock/caffe -b ssd
cd caffe
mkdir build
cd build
cmake .. -DCPU_ONLY=ON -DBUILD_python=OFF
make all -j$(nproc)
make install
make runtest
cd ~/caffe
protoc src/caffe/proto/caffe.proto --cpp_out=.
mkdir include/caffe/proto
mv src/caffe/proto/caffe.pb.h include/caffe/proto
python3 scripts/download_model_binary.py models/bvlc_reference_caffenet/


cd ~/caffe/models/bvlc_reference_caffenet/
cp bvlc_reference_caffenet.caffemodel YOUR_CATKIN_WORKSPACE/src/rs_resources/caffe/models/bvlc_reference_caffenet/

cd YOUR_CATKIN_WORKSPACE
catkin clean # we have to rebuild to get caffe properly detected in the following steps
catkin build

# Step 3
# Test Object detection in a RS pipeline
# First, 'rosbag play' the bag file from the RS tutorial.
# Then start an example pipeline from RoboSherlock:
rosrun robosherlock runAAE _ae:=demo_addons
```


