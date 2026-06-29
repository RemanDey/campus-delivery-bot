# Project Husky-Mandi: All-Terrain Campus Delivery AMR

An autonomous, high-torque, skid-steer Autonomous Mobile Robot (AMR) engineered specifically for the steep gradients, variable terrain, and satellite-occluded mountainous landscapes of the IIT Mandi campus in Himachal Pradesh, India. 

Developed by **Reman Dey**

ALL AI USAGE ARE written in the AI_USAGE.md file

---

## Document & Project Metadata
* **Author:** Reman Dey
* **Advisor/Mentorship:** Robotronics Club, IIT Mandi
* **Hardware Focus:** Cytron iMD16 Dual-Channel Brushed DC Motor Driver
* **Compute Platform:** NVIDIA Jetson Orin Nano
* **Software Environment:** ROS 2 Humble Hawksbill on Ubuntu 22.04 LTS (JetPack 6)

---

## Core Technical Specifications

### System Architecture
* **Drivetrain:** 4-Wheel Heavy-Duty Skid-Steer (Zero turning radius, maximum hill-holding capability).
* **Actuators:** $4 \times$ 24V Brushed Planetary Geared Motors ($T_{\text{stall}} \ge 50\text{ Nm}$, $n_{\text{free}} \approx 120\text{ RPM}$, internal armature resistance $R \ge 0.8\ \Omega$).
* **Motor Drive Authority:** $2 \times$ Cytron iMD16 Dual-Channel Drivers configured in parallel banks.

```
                  +-----------------------------+
                  |  NVIDIA Jetson Orin Nano    |
                  +--------------+--------------+
                                 |
                        [ 3.3V GPIO PWM ]
                                 |
                 +---------------+---------------+
                 |                               |
                 v                               v
       +-------------------+           +-------------------+
       | iMD16 Unit A      |           | iMD16 Unit B      |
       | (Left Bank)       |           | (Right Bank)      |
       +---------+---------+           +---------+---------+
                 |                               |
         +-------+-------+               +-------+-------+
         |               |               |               |
         v               v               v               v
    +---------+     +---------+     +---------+     +---------+
    | Motor   |     | Motor   |     | Motor   |     | Motor   |
    | Front-L |     | Rear-L  |     | Front-R |     | Rear-R  |
    +---------+     +---------+     +---------+     +---------+
```

### Motor Driver Boundary Evaluation (Cytron iMD16)
The iMD16 is deployed at its ideal engineering boundary. Below is a summary of the application scoping validated in the technical brief:

| Application / Drivetrain | Compatibility | Rationale |
| :--- | :--- | :--- |
| **Campus AMR Skid-Steer (24V Brushed)** | **Fully Suitable** | Matches voltage, topology, and peak current limits ($\le 30\text{A}$). |
| **Electric Auto-Rickshaw / E-Toto** | **Incompatible** | Voltage exceedance ($48\text{V} > 30\text{V}$) and motor topology mismatch (Requires 3-phase BLDC). |
| **Passenger Electric Bus Traction** | **Incompatible** | Extreme power deficit ($\Delta P \approx 149.5\text{ kW}$); requires multi-phase SiC/GaN inverters. |

---

## Technology Stack & Node Graph

The software pipeline leverages hardware acceleration for both neural network inference and pose estimation to preserve the Jetson compute budget.

### Node Topology (`/cmd_vel` & Perception Pipeline)
```bash
/oak_d_pro_node          # OAK-D Pro depth + RGB stream publisher
/yolov8_seg_node         # Obstacle segmentation inference (TensorRT)
/robot_localization      # EKF node: GNSS + IMU + wheel encoder fusion
/nav2_bringup            # Navigation2 stack (A* global, DWB local planner)
/imd16_driver_node       # PWM cmd_vel -> iMD16 duty cycle bridge
/ublox_driver            # u-blox ZED-F9P GNSS NMEA/UBX publisher
/bno085_imu_node         # 9-axis IMU data publisher
/encoder_odometry_node   # Wheel encoder dead-reckoning odometry
/estop_monitor_node      # 433 MHz E-Stop hardware watchdog node
```

### Perception & Segmentation
* **Luxonis OAK-D Pro:** Structured-light stereo depth mapping providing 30Hz point clouds to the Nav2 3D costmap.
* **YOLOv8-Segmentation:** Deployed directly onto the camera module's onboard Intel Myriad X VPU via OpenVINO. 
* **Dynamic Safety Inflation Zones:** Pedestrians trigger a $1.2\text{ m}$ safety cushion, while static campus obstacles default to $0.5\text{ m}$.

---

## Mathematical Modeling Framework

The vehicle's power matching and control loops are governed by the coupled lumped-parameter electromechanical equations detailed in the brief.

### 1. Electrical Armature Dynamics (KVL)
$$V(t) = R\,i(t) + L\,\frac{di(t)}{dt} + e_b(t)$$

Where $e_b(t) = K_e\,\omega(t)$ is the back-electromotive force opposing the driver's voltage loop.

### 2. Rotational Mechanics (Newton's Second Law)
$$T_m(t) = J\,\frac{d\omega(t)}{dt} + B\,\omega(t) + T_L(t)$$

Where the electromagnetic torque output is bounded by $T_m(t) = K_t\,i(t)$.

### 3. Steady-State Torque-Speed Boundary
$$\boxed{T_m(\omega) = \frac{K}{R}\,V - \frac{K^2}{R}\,\omega}$$

> **Engineering Mitigation Note:** Rapid deceleration generates strong regenerative energy pulses. Because standard SMPS units cannot handle bidirectional current spikes, this system implements a parallel **24V $\text{LiFePO}_4$ battery buffer** to safely absorb back-EMF spikes and prevent overvoltage fault trips during steep Himalayan descents.

---

## Getting Started

### Prerequisites
* NVIDIA Jetson Orin Nano flashed with **JetPack 6.x**
* ROS 2 Humble Hawksbill desktop installation
* Luxonis DepthAI library dependencies (`ros-humble-depthai-ros`)

### Installation & Workspace Setup
```bash
# Clone the repository into your ROS 2 workspace
cd ~/ros2_ws/src
git clone https://github.com/your-username/husky-mandi.git

# Install required system dependencies
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y

# Build the workspace using colcon
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

### Launching the Navigation Stack
To initialize the full localization suite (EKF sensor fusion, OAK-D perception, Nav2 controllers, and the low-level iMD16 PWM driver node), run:
```bash
ros2 launch husky_mandi_bringup fully_autonomous_launch.py
```

---

## Acknowledgments
Special thanks to the **Robotronics Club, IIT Mandi** for technical guidance, infrastructure, and hardware validation support during this evaluation phase.