# AGX Piper ROS2 Usage Guide
## Requirements
- Ubuntu 24.04
- ROS2 Jazzy
- AGX Arm ROS SDK
- CAN interface configured (can0)
- Intel RealSense D435 (optional)

**Connect Hardware**
<img width="3024" height="4032" alt="a60db105ca9b49449b6acfebd11a4ec9" src="https://github.com/user-attachments/assets/356ba724-0ade-4374-a006-65791c853ac1" />

## 1. Build the Workspace

Compile the workspace:
```bash
cd ~/agx_arm_ws
colcon build
```
Source the workspace:
```
source ~/agx_arm_ws/install/setup.bash
```
<img width="727" height="339" alt="image" src="https://github.com/user-attachments/assets/20470656-7e04-4020-9262-c7cffcec0b3b" />

**Every new terminal should source the workspace before running any ROS commands.**

## 2. Activate the CAN Interface

Navigate to the script directory:
```
cd ~/agx_arm_ws/src/agx_arm_ros/scripts
```
Activate CAN:
```
bash can_activate.sh
```
<img width="811" height="227" alt="Screenshot from 2026-07-15 14-43-28" src="https://github.com/user-attachments/assets/94f3a16d-b384-4a64-9176-36428e835c4d" />

## 3. Start the Arm Controller

Run the controller node:
```
ros2 run agx_arm_ctrl agx_arm_ctrl_single \
  --ros-args \
  -p can_port:=can0 \
  -p arm_type:=piper \
  -p effector_type:=none
```
Expected output:
- All joints enable status is True
- Agx_arm feedback is ready, control is now enabled
<img width="811" height="532" alt="Screenshot from 2026-07-15 14-46-14" src="https://github.com/user-attachments/assets/b6d715dc-72c3-4c9c-8c79-0f4a70699101" />

You can also launch the controller using:
```
ros2 launch agx_arm_ctrl start_single_agx_arm.launch.py \
    can_port:=can0 \
    arm_type:=piper \
    effector_type:=none \
    tcp_offset:='[0.0,0.0,0.0,0.0,0.0,0.0]'
```
<img width="1909" height="432" alt="image" src="https://github.com/user-attachments/assets/836b9857-6c34-4fbf-aaea-c255cc3145ee" />

## 4. Verify Arm Status

Check whether the controller is connected:
```
ros2 topic echo /feedback/arm_status
```
A normal status should look similar to:
```
ctrl_mode: 1
teach_status: 0
motion_status: 0
err_status: 0
```

Field	Description
ctrl_mode = 1	CAN command control
teach_status = 0	Not in teach mode
motion_status = 0	Normal
err_status = 0	No error

## 5. Move the Robot (Joint Command)

Publish a JointState command:
```
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
```
Joint angles are specified in radians.

Expected Output Position
<img width="4032" height="3024" alt="ff73fa7b22cb61e04b6db5b48cda6ee7" src="https://github.com/user-attachments/assets/3f757b82-c8a1-492c-9adb-acf77e9d82d5" />

## 6. RViz Control

Launch RViz with interactive control:
```
ros2 launch agx_arm_ctrl start_single_agx_arm_rviz.launch.py \
    can_port:=can0 \
    arm_type:=piper \
    follow:=true \
    control:=true
```
Expected Output
<img width="1722" height="702" alt="image" src="https://github.com/user-attachments/assets/ae8d4d89-e72b-4b07-8c60-49af5bc365d1" />

Initial Position 
<img width="1470" height="890" alt="image" src="https://github.com/user-attachments/assets/1c2aab24-394d-4040-b1af-e400fd40f00c" />
<img width="3024" height="4032" alt="4c282509743cb0ef52abf87047e4b657" src="https://github.com/user-attachments/assets/bddc303d-138e-4f03-bbb2-6338bfa760c3" />

Control

## 7. MoveIt

Launch MoveIt:
```
ros2 launch agx_arm_ctrl start_single_agx_arm_moveit.launch.py \
    can_port:=can0 \
    arm_type:=piper \
    effector_type:=agx_gripper \
    gripper_default_effort:=2.2
```
This includes

- MoveIt Planning
- RViz
- Motion Planning
- Gripper Controller

Initial Position

Arm

Gripper

## 8. RealSense Camera

<img width="4032" height="3024" alt="20ee1b7f518a0b174a114a227c28b4d6" src="https://github.com/user-attachments/assets/df55f981-e6c5-406c-bf35-9255d6395ccb" />

Launch the camera:
```
ros2 launch realsense2_camera rs_launch.py
```
<img width="1878" height="766" alt="image" src="https://github.com/user-attachments/assets/bbe7dddf-efb7-47aa-bdbb-e70660b1b73c" />

## 9. Visualize Images

Using RViz:
```
rviz2
```
*Need to add camera manually*
Or
```
ros2 run rqt_image_view rqt_image_view
```
Sample Output: 
<img width="1211" height="764" alt="Screenshot from 2026-07-16 12-03-17" src="https://github.com/user-attachments/assets/fcd4b33e-9772-410b-a0b6-e925d8653d35" />
<img width="1211" height="764" alt="Screenshot from 2026-07-16 12-03-26" src="https://github.com/user-attachments/assets/a3e4bf6d-280b-426e-a5ad-754639b61319" />

## 10. Frequently Used Services

Enable the arm:
```
ros2 service call /enable_agx_arm std_srvs/srv/SetBool "{data: true}"
```
Move to Home:
```
ros2 service call /move_home std_srvs/srv/Empty
```
Emergency Stop:
```
ros2 service call /emergency_stop std_srvs/srv/Empty
```
## 11. Check Available Nodes
```
ros2 node list
```
Inspect node information:
```
ros2 node info /agx_arm_ctrl_single_node
```
## 12. Check Available Topics
```
ros2 topic list
```
Useful topics:
```
Topic	Description
/control/move_j	Joint control
/feedback/joint_states	Joint feedback
/feedback/tcp_pose	TCP pose
/feedback/arm_status	Controller status
```
## 13. Typical Startup Workflow
```
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
```
Notes
The robot must report:
ctrl_mode: 1

before accepting motion commands.

Verify that
err_status: 0

before executing trajectories.

If the controller does not respond, first check the CAN interface and ensure the controller node has started successfully.

<p align="center">
  <img src="images/rviz.png" width="700">
</p>
