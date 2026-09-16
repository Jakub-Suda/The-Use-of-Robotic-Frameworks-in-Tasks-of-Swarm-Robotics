# The Use of Robotic Frameworks in Tasks of Swarm Robotics

This repository accompanies the paper "The Use of Robotic Frameworks in Tasks of Swarm Robotics." It contains additional informations of this project by Ing. Jakub Suďa and doc. Dr. Ing. Ján Vaščák

### Framework Architecture

  link images from figures

### Proposed UAV platform
The proposed UAV platform uses esp32 drone as base. It was needed to expand that to be able to use in indoor/interior flight. The UAV platform alone defaultly contained MPU-6050 IMU unit, which contains basic gyroscope and accelerometer. For interior flight it was needed to add PMW3901 Optical flow sensor for position hold and VLX53LOX Time of Flight sensor for relatively accurate height detection
![image](figures/UAV_platform.png)

### Firmware of the drone
Firmware of drone is coded in FreeRTOS with adjustment to esp32s2, through using an extension of FreeRTOS called esp-idf (v5.5).
Firmware uses multiple software components from esp component registry. Firmware is possible to split into multiple simple software modules, which contains multiple more submodules.
 link images from figures

### Simulated UAV model
  The Webots simulation enviroment was used for the simulation. The UAV model is trying to copy most of physical properties(weight, propeller size,.....) of real drone to simulation. On figure it is possible to see the designed webots model throught .PROTO file
 link image from figures

### Contact:
jakub.suda@tuke.sk
jan.vascak@tuke.sk
