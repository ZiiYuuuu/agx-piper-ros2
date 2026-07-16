# AGX Piper ROS2 Usage Guide
## Requirements
- Ubuntu 24.04
- ROS2 Jazzy
- AGX Arm ROS SDK
- CAN interface configured (can0)
- Intel RealSense D435 (optional)

## 1. Build the Workspace

Compile the workspace:
'''
cd ~/agx_arm_ws
colcon build
'''
Source the workspace:
'''
source ~/agx_arm_ws/install/setup.bash
'''
**Every new terminal should source the workspace before running any ROS commands.**

2. Activate the CAN Interface

Navigate to the script directory:

cd ~/agx_arm_ws/src/agx_arm_ros/scripts

Activate CAN:

bash can_activate.sh
3. Start the Arm Controller

Run the controller node:

ros2 run agx_arm_ctrl agx_arm_ctrl_single \
  --ros-args \
  -p can_port:=can0 \
  -p arm_type:=piper \
  -p effector_type:=none

Expected output:

All joints enable status is True
Agx_arm feedback is ready, control is now enabled

You can also launch the controller using:

ros2 launch agx_arm_ctrl start_single_agx_arm.launch.py \
    can_port:=can0 \
    arm_type:=piper \
    effector_type:=none \
    tcp_offset:='[0.0,0.0,0.0,0.0,0.0,0.0]'
4. Verify Arm Status

Check whether the controller is connected:

ros2 topic echo /feedback/arm_status

A normal status should look similar to:

ctrl_mode: 1
teach_status: 0
motion_status: 0
err_status: 0

Meaning:

Field	Description
ctrl_mode = 1	CAN command control
teach_status = 0	Not in teach mode
motion_status = 0	Normal
err_status = 0	No error
5. Move the Robot (Joint Command)

Publish a JointState command:

ros2 topic pub --once /control/move_j sensor_msgs/msg/JointState "
name:
- joint1
- joint2
- joint3
- joint4
- joint5
- joint6

position:
- 1.3
- 0.8
- -1.0
- 0.0
- 0.0
- 1.0

velocity: []
effort: []
"

Joint angles are specified in radians.

6. RViz Control

Launch RViz with interactive control:

ros2 launch agx_arm_ctrl start_single_agx_arm_rviz.launch.py \
    can_port:=can0 \
    arm_type:=piper \
    follow:=true \
    control:=true
7. MoveIt

Launch MoveIt:

ros2 launch agx_arm_ctrl start_single_agx_arm_moveit.launch.py \
    can_port:=can0 \
    arm_type:=piper \
    effector_type:=agx_gripper \
    gripper_default_effort:=2.2

This starts

MoveIt Planning
RViz
Motion Planning
Gripper Controller
8. RealSense Camera

Launch the camera:

ros2 launch realsense2_camera rs_launch.py

Launch with aligned depth image:

source /opt/ros/jazzy/setup.bash

ros2 launch realsense2_camera rs_launch.py \
    align_depth.enable:=true
9. Visualize Images

Using RViz:

rviz2

Or

ros2 run rqt_image_view rqt_image_view
10. Frequently Used Services

Enable the arm:

ros2 service call /enable_agx_arm std_srvs/srv/SetBool "{data: true}"

Move to Home:

ros2 service call /move_home std_srvs/srv/Empty

Emergency Stop:

ros2 service call /emergency_stop std_srvs/srv/Empty
11. Check Available Nodes
ros2 node list

Inspect node information:

ros2 node info /agx_arm_ctrl_single_node
12. Check Available Topics
ros2 topic list

Useful topics:

Topic	Description
/control/move_j	Joint control
/feedback/joint_states	Joint feedback
/feedback/tcp_pose	TCP pose
/feedback/arm_status	Controller status
13. Typical Startup Workflow
Build Workspace
        │
        ▼
Source Workspace
        │
        ▼
Activate CAN
        │
        ▼
Launch agx_arm_ctrl
        │
        ▼
Verify ctrl_mode == 1
        │
        ▼
Launch RViz / MoveIt
        │
        ▼
Control the Robot
Notes
The robot must report:
ctrl_mode: 1

before accepting motion commands.

Verify that
err_status: 0

before executing trajectories.

If the controller does not respond, first check the CAN interface and ensure the controller node has started successfully.
📁 Recommended Repository Structure

建议你的 GitHub 最终目录结构整理成这样：

agx_arm_project/
│
├── README.md                ← 使用说明（就是上面的内容）
├── images/
│   ├── rviz.png
│   ├── moveit.png
│   ├── camera.png
│   ├── system_overview.png
│   └── hardware_setup.jpg
│
├── launch/
├── scripts/
├── config/
└── src/

然后在 README 中插入图片，例如：

## System Overview

<p align="center">
  <img src="images/system_overview.png" width="700">
</p>

## RViz

<p align="center">
  <img src="images/rviz.png" width="700">
</p>
