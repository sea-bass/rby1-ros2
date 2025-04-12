# rby1_ros2
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%2022.04-E95420?logo=ubuntu)
![ROS 2](https://img.shields.io/badge/ROS%202-Humble-blue?logo=ros)

> [!WARNING]
> This software is currently under active development, and its stability is not guaranteed.  
> Various bugs may occur during use, so please proceed with caution.

## Installation
### Install system dependencies
```shell
sudo apt install -y build-essential cmake git

mkdir -p ~/sdk && cd ~/sdk

#install conan
pip install conan

#clone the repository
git clone --recurse-submodules git@github.com:RainbowRobotics/rby1-sdk.git

#install or build dependencies
cd rby1-sdk
conan install . -s build_type=Release -b missing -of build

cmake --preset conan-release
cmake --build --preset conan-release
```

### Set up environment
```shell
export CMAKE_PREFIX_PATH=/opt/ros/humble:$CMAKE_PREFIX_PATH
export RBY1_SDK_PATH=~/sdk/rby1-sdk
source /opt/ros/humble/setup.bash
source ~/rby1_ros2_ws/install/setup.bash
```

### Build from source
#### Create ROS 2 workspace
```shell
mkdir -p ~/rby1_ros2_ws/src
```

#### Clone repo and build `rby1_ros2` packages:

One time set up to clone the repo and install ROS 2 dependencies.

```bash
cd ~/rby1_ros2_ws
git clone https://github.com/RainbowRobotics/rby1-ros2.git src
rosdep install --from-paths src -y --ignore-src
```

Build and source the workspace.

```bash
colcon build --symlink-install
source install/setup.bash
```

## How to Use
### Manipulation
#### 1. Bring Up
This command launches the robot bringup process:
- Establishes a connection to the robot via gRPC.
- Turns **All Servos ON**.
- Enables **Control Manager** (Control Enable).

> [!TIP]
> We recommend verifying this step in simulation before running on the real robot.

```shell
ros2 launch rby1_bringup bringup.launch robot_ip:=<Rpc_ip_address:port_number>
```

#### 2. MoveIt Config
```shell
ros2 launch rby1_moveit_config rby1_moveit.launch
```

#### Available Controllers

The following controllers are configured and available through `ros2_control`.  
Each controller controls a specific set of joints:

| Controller Name             | Controlled Joints                                                |
|-----------------------------|------------------------------------------------------------------|
| `rby1_right_arm_controller` | `right_arm_0` to `right_arm_6`                                  |
| `rby1_left_arm_controller`  | `left_arm_0` to `left_arm_6`                                    |
| `rby1_torso_controller`     | `torso_0` to `torso_5`                                           |
| `rby1_dualarm_controller`   | `left_arm_0` to `left_arm_6` + `right_arm_0` to `right_arm_6`   |
| `rby1_head_controller`      | `head_0`, `head_1`                                               |
| `rby1_wholebody_controller` | Torso + Both Arms + Head (※ Does **not** include wheels)        |

> [!CAUTION]
> `rby1_wholebody_controller` controls the **torso, both arms, and head only**.  
> It does **not** include wheel or mobility joints, even though the name may suggest so.


### Mobile Controller
Originally, the mobile control feature was intended to allow real-time parameter adjustments via the UI — for example, changing linear and angular velocity on the fly.  
However, this functionality is not yet implemented.

Currently, to control the mobile platform, you must manually modify the velocity parameters in the `mobile_publisher` code, then restart the node to apply the changes.

> [!WARNING]
> This feature is still under development and may contain bugs.  
> As it may cause collisions with surrounding objects or people, please use it with caution.

#### Publish mobile control msg
```shell
ros2 run rby1_mobile_control mobile_publisher
```

#### Subscribe mobile control msg
```shell
ros2 run rby1_subscriber_pkg 07_mobile_control
```

