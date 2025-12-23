# Subscriber-in-ROS2

# ROS 2 Temperature Subscriber Project

## Project Overview
This project demonstrates the creation of a ROS 2 subscriber node that monitors temperature data and provides real-time feedback on whether the temperature is too high, too low, or within acceptable range.

---

## 1. Introduction

### What is a Subscriber Node in ROS 2?
A subscriber node in ROS 2 is a component that listens to messages published on a specific topic. It registers a callback function that is automatically invoked whenever new data is received on the subscribed topic. This publish-subscribe pattern enables decoupled communication between different nodes in a robotic system.

### Use Case: Temperature Monitoring System
**Topic Name:** `temperature`  
**Message Type:** `std_msgs/msg/Float32`

This project implements a temperature monitoring system where:
- A publisher node simulates temperature sensor readings
- A subscriber node monitors these readings in real-time
- The subscriber provides warnings when temperature exceeds safe thresholds (22°C - 28°C)

**Applications:**
- Server room monitoring
- Industrial process control
- Smart home climate management
- Laboratory equipment monitoring

---

## 2. Materials & Tools Used

| Component | Details |
|-----------|---------|
| **Operating System** | Ubuntu 22.04 LTS |
| **ROS Distribution** | ROS 2 Humble Hawksbill |
| **Programming Language** | Python 3.10 |
| **Build System** | CMake (ament_cmake) |
| **Development Tools** | VS Code / Nano text editor |
| **Testing Tools** | `ros2 topic`, `ros2 node`, `ros2 run` |

---

## 3. Methodology

### 3.1 Workspace and Package Creation

### 3.2 Package Structure

```
task1_ws/
├── src/
│   └── temp_pkg/
│       ├── CMakeLists.txt
│       ├── package.xml
│       └── scripts/
│           ├── temp_pub.py      # Temperature publisher
│           └── temp_sub.py      # Temperature subscriber
├── build/
├── install/
└── log/
```
### 3.4 Callback Function Logic

The `subscribe_temperature` callback implements threshold-based monitoring:

1. **Receives** the Float32 message containing temperature data
2. **Extracts** the temperature value from `msg.data`
3. **Evaluates** against thresholds:
   - `temp > 28°C` → HIGH temperature warning
   - `temp < 22°C` → LOW temperature warning
   - `22°C ≤ temp ≤ 28°C` → Temperature OK
4. **Logs** appropriate message using ROS 2 logger

### 3.5 CMakeLists.txt Configuration

### 3.6 Build and Run Commands

```bash
# Build the package
cd ~/task1_ws
colcon build 

# Source the workspace
source install/setup.bash

# Run subscriber (Terminal 1)
ros2 run temp_pkg temp_sub.py

# Run publisher (Terminal 2)
ros2 run temp_pkg temp_pub.py

# Run rqt graph (Terminal 3)
rqt_graph

```

---

## 4. Problem-Solving Approach

### Critical Error Encountered: Missing Callback Parameter

#### Problem Description
```
TypeError: Node.create_subscription() missing 1 required positional argument: 'qos_profile'
```

#### Root Cause
The `create_subscription()` method was called incorrectly:

**❌ Incorrect Code:**
```python
self.subscriber = self.create_subscription(Float32, 'temperature', 10)
```

The callback function parameter was missing, causing Python to interpret `10` (QoS profile) as the callback, which resulted in the error.

#### Solution
**✅ Correct Code:**
```python
self.subscriber = self.create_subscription(
    Float32,                      # Message type
    'temperature',                # Topic name
    self.subscribe_temperature,   # Callback function (REQUIRED)
    10)                           # QoS profile
```

#### Key Learning
The `create_subscription()` method signature requires **exactly 4 parameters**:
1. Message type
2. Topic name
3. Callback function
4. QoS profile
---

## 6. Testing & Results

### 6.1 Test Setup

**Terminal 1: Subscriber**
```bash
$ ros2 run temp_pkg temp_sub.py
[INFO] [temperature_subscriber]: Temperature Subscriber Node Started
```

**Terminal 2: Publisher**
```bash
$ ros2 run temp_pkg temp_pub.py
[INFO] [temperature_publisher]: Publishing: 25.34°C
[INFO] [temperature_publisher]: Publishing: 29.12°C
[INFO] [temperature_publisher]: Publishing: 20.45°C
```

### 6.2 Test Results

#### Scenario 1: Normal Temperature (22-28°C)
```
[INFO] [temperature_subscriber]: Temperature OK: 25.34°C
[INFO] [temperature_subscriber]: Temperature OK: 26.78°C
[INFO] [temperature_subscriber]: Temperature OK: 23.12°C
```

#### Scenario 2: High Temperature (>28°C)
```
[WARN] [temperature_subscriber]: Temperature TOO HIGH: 29.12°C
[WARN] [temperature_subscriber]: Temperature TOO HIGH: 31.45°C
[WARN] [temperature_subscriber]: Temperature TOO HIGH: 28.89°C
```

#### Scenario 3: Low Temperature (<22°C)
```
[WARN] [temperature_subscriber]: Temperature TOO LOW: 20.45°C
[WARN] [temperature_subscriber]: Temperature TOO LOW: 18.23°C
[WARN] [temperature_subscriber]: Temperature TOO LOW: 21.67°C
```
### 6.3 Observations

1. **Real-time Monitoring:** Subscriber responds immediately to published messages
2. **Threshold Detection:** Accurately identifies temperature ranges
3. **Message Logging:** Clear, informative log messages with appropriate severity levels
4. **System Stability:** No message loss or delays observed during extended testing
5. **Resource Usage:** Minimal CPU and memory footprint

---

## 7. Conclusion

### Key Learnings

1. **ROS 2 Subscriber Architecture:**
   - Understanding the publish-subscribe pattern
   - Proper use of callback functions for asynchronous message handling
   - QoS (Quality of Service) profile configuration

2. **Python ROS 2 API:**
   - Correct signature for `create_subscription()` method
   - Importance of all four parameters (type, topic, callback, QoS)
   - Node lifecycle management (init, spin, destroy, shutdown)

3. **Debugging Skills:**
   - Systematic error diagnosis using ROS 2 CLI tools
   - Environment configuration importance (sourcing from correct directory)
   - Build system behavior and cache management

4. **Best Practices:**
   - Meaningful node and topic names
   - Clear, descriptive log messages
   - Proper shebang lines for Python scripts
   - File permissions and executable flags

### Possible Extensions

#### 1. Multiple Topic Subscription
Subscribe to multiple sensor topics simultaneously:
```python
self.temp_sub = self.create_subscription(Float32, 'temperature', self.temp_callback, 10)
self.humidity_sub = self.create_subscription(Float32, 'humidity', self.humidity_callback, 10)
self.pressure_sub = self.create_subscription(Float32, 'pressure', self.pressure_callback, 10)
```

#### 2. Alert System Integration
Send email/SMS alerts for critical temperatures:
```python
def subscribe_temperature(self, msg):
    temp = msg.data
    if temp > 30.0 or temp < 20.0:
        self.send_alert(f"Critical temperature: {temp}°C")
```

#### 3. Visualization Dashboard
Create real-time temperature monitoring dashboard using RViz2 or custom web interface

#### 4. Multi-Zone Monitoring
Subscribe to multiple temperature sensors in different locations:
```python
self.zone1_sub = self.create_subscription(Float32, 'zone1/temperature', self.zone1_callback, 10)
self.zone2_sub = self.create_subscription(Float32, 'zone2/temperature', self.zone2_callback, 10)
```
---

## How to Use This Repository

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/ros2-temperature-subscriber.git
cd ros2-temperature-subscriber
```

### 2. Setup Workspace
```bash
mkdir -p ~/task1_ws/src
cp -r src/temp_pkg ~/task1_ws/src/
cd ~/task1_ws
```

### 3. Build
```bash
colcon build --packages-select temp_pkg
source install/setup.bash
```

### 4. Run
```bash
# Terminal 1
ros2 run temp_pkg temp_sub.py

# Terminal 2
ros2 run temp_pkg temp_pub.py
```

---
<img width="1440" height="900" alt="Screenshot 2025-12-23 at 9 42 44 AM" src="https://github.com/user-attachments/assets/bfd5630e-fefb-4a2a-b24a-086d9bfda3b6" />

<img width="1440" height="900" alt="Screenshot 2025-12-23 at 9 42 53 AM" src="https://github.com/user-attachments/assets/8b5e6f62-dae5-43b5-8a29-63fbe7d2e90a" />

<img width="1440" height="900" alt="Screenshot 2025-12-23 at 9 41 57 AM" src="https://github.com/user-attachments/assets/654d9ad2-8781-4c2a-8806-89d32f68a66b" />

<img width="1440" height="900" alt="Screenshot 2025-12-23 at 9 42 16 AM" src="https://github.com/user-attachments/assets/c163b624-89af-450d-93d0-627f7512ff73" />



## Author
**Akhileshwar Pratap Singh**  
ROS 2 Development Project  
December 2025

## License
MIT License - See LICENSE file for details
