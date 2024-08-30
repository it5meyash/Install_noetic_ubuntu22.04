# Install_noetic_ubuntu22.04

Why do we want to do this ?
As mentioned here, ROS Noetic is primarily targeted at Ubuntu 20.04 and doesn’t officially support any newer distribution. Further, ROS Noetic is the last available ROS 1 version and is going to be discontinued in May 2025.

Thus, if you want to use ROS Noetic after end of life on Ubuntu 22.04 or any newer Ubuntu distribution, there is actually almost no way around building the required packages on your own.

How can we do it ?
First of all, please make sure you have the following packages installed on your native system to be able to run the commands without error interruptions:

docker
python-3
pip3
git
ROS Noetic base packages
The first thing we need to do is to figure out which packages are part of the original ros-noetic-ros-base package. The easiest way to get this information, is to make use of the rosinstall_generator as it is described here.

To get a list of required packages and version tags we simply run the following two commands within the terminal:

pip3 install -U rosinstall_generator 
rosinstall_generator ros_base --rosdistro noetic --deps --tar > noetic-base.rosinstall
The resulting file noetic-base.rosinstall should contain something like:

- tar:
    local-name: actionlib/actionlib
    uri: https://github.com/ros-gbp/actionlib-release/archive/release/noetic/actionlib/1.14.0-1.tar.gz
    version: actionlib-release-release-noetic-actionlib-1.14.0-1
- tar:
    local-name: bond_core/bond
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/noetic/bond/1.8.6-1.tar.gz
    version: bond_core-release-release-noetic-bond-1.8.6-1
- tar:
    local-name: bond_core/bond_core
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/noetic/bond_core/1.8.6-1.tar.gz
    version: bond_core-release-release-noetic-bond_core-1.8.6-1
- tar:
    local-name: bond_core/bondcpp
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/noetic/bondcpp/1.8.6-1.tar.gz
    version: bond_core-release-release-noetic-bondcpp-1.8.6-1
...
By inspecting noetic-base.rosinstall and researching for the original GitHub repositories, we get the following list of packages and version tags:

actionlib-1.14.0 | bond_core-1.8.6 | catkin-0.8.10 | class_loader-0.5.0 | cmake_modules-0.5.0 | common_msgs-1.13.1 | dynamic_reconfigure-1.7.3 | gencpp-0.7.0 | geneus-3.0.0 | genlisp-0.4.18 | genmsg-0.6.0 | gennodejs-2.0.1 | genpy-0.6.16 | message_generation-0.4.1 | message_runtime-0.4.13 | nodelet_core-1.10.2 | pluginlib-1.13.0 | ros-1.15.18 | ros_comm-1.16.0 | ros_comm_msgs-1.11.3 | ros_environment-1.3.2 | rosbag_migration_rule-1.0.1 | rosconsole-1.14.3 | rosconsole_bridge-0.5.4 | roscpp_core-0.7.2 | roslisp-1.9.25 | rospack-2.6.2 | std_msgs-0.5.13

Cloning the repositories
Since we are dealing with ROS packages and make use of catkin as a build tool, we need to follow the ROS package structure.

Therefore, we simply create a root directory called ros_noetic_base_2204 and a catkin workspace by executing

mkdir -p ros_noetic_base_2204/catkin_ws/src
ros_noetic_base_2204
└── catkin_ws
    └── src
Now it is time to clone all the necessary GitHub repositories. This can be achieved by executing the following commands within the directory ros_noetic_base_2204/catkin_ws/src

git clone https://github.com/ros/actionlib.git -b 1.14.0
git clone https://github.com/ros/bond_core.git -b 1.8.6
git clone https://github.com/ros/catkin.git -b 0.8.10
git clone https://github.com/ros/class_loader.git -b 0.5.0
git clone https://github.com/ros/cmake_modules.git -b 0.5.0
git clone https://github.com/ros/common_msgs.git -b 1.13.1
git clone https://github.com/ros/dynamic_reconfigure.git -b 1.7.3
git clone https://github.com/ros/gencpp.git -b 0.7.0
git clone https://github.com/jsk-ros-pkg/geneus.git -b 3.0.0
git clone https://github.com/ros/genlisp.git -b 0.4.18
git clone https://github.com/ros/genmsg.git -b 0.6.0
git clone https://github.com/RethinkRobotics-opensource/gennodejs.git -b 2.0.1
git clone https://github.com/ros/genpy.git -b 0.6.16
git clone https://github.com/ros/message_generation.git -b 0.4.1
git clone https://github.com/ros/message_runtime.git -b 0.4.13
git clone https://github.com/ros/nodelet_core.git -b 1.10.2
git clone https://github.com/ros/pluginlib.git -b 1.13.0
git clone https://github.com/ros/ros.git -b 1.15.8
git clone https://github.com/ros/ros_comm.git -b 1.16.0
git clone https://github.com/ros/ros_comm_msgs.git -b 1.11.3
git clone https://github.com/ros/ros_environment.git -b 1.3.2
git clone https://github.com/ros/rosbag_migration_rule.git -b 1.0.1
git clone https://github.com/ros/rosbag_migration_rule.git -b 1.0.1
git clone https://github.com/ros/rosconsole.git -b 1.14.3
git clone https://github.com/ros/rosconsole_bridge.git -b 0.5.4
git clone https://github.com/ros/roscpp_core.git -b 0.7.2
git clone https://github.com/ros/roslisp.git -b 1.9.25
git clone https://github.com/ros/rospack.git -b 2.6.2
git clone https://github.com/ros/std_msgs.git -b 0.5.13
Additionally to the repositories listed above, we also need to clone catkin_pkg as well as rospkg. Those libraries are non catkin packages and therefore need to stay outside the catkin_ws in order to avoid build errors later on. It is ok to just put it into the root directory

git clone https://github.com/ros-infrastructure/catkin_pkg.git -b 0.5.2
git clone https://github.com/ros-infrastructure/rospkg.git -b 1.5.0
ros_noetic_base_2204
├── catkin_pkg
├── catkin_ws
└── rospkg
Creating the Docker environment
To build the previously cloned packages, a docker environment is used. We start with a minimum Dockerfile based on Ubuntu 22.04 containing some build essentials and add in the course of the following steps further dependencies.

ros_noetic_base_2204
├── catkin_pkg
├── catkin_ws
├── Dockerfile
└── rospkg
Dockerfile:

FROM ubuntu:22.04

RUN apt-get update && \
        apt-get install -y \
        cmake \
        build-essential \
