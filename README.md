
# The Use of Robotic Frameworks in Tasks of Swarm Robotics

This repository accompanies the paper *"The Use of Robotic Frameworks in Tasks of Swarm Robotics"* by Ing. Jakub Suďa and doc. Dr. Ing. Ján Vaščák (Technical University of Košice). It contains supplementary material, figures, and source code for a low-cost indoor UAV platform and centralised ground-control framework developed for aerial swarm coordination.

## Overview

Robotics has traditionally focused on individual autonomous platforms, while swarm robotics investigates groups of simpler agents coordinating to accomplish tasks collectively, inspired by self-organising systems in nature such as ant colonies and bird flocks. UAV swarms are a particularly active area of this research, offering three-dimensional mobility and distributed sensing for applications such as environmental monitoring, infrastructure inspection, search and rescue, and surveillance — but their deployment raises substantial challenges in communication, coordination, sensing, and energy efficiency, especially indoors where GNSS positioning is unavailable.

This project addresses that gap with a low-cost, custom-built ESP32 UAV platform and a Python-based Ground Control Station (GCS) that centralises swarm-level coordination, so individual UAVs only need enough onboard computation for local flight stabilisation and communication. A matching Webots simulation model was developed alongside the physical platform to evaluate swarm-scale behaviours that the hardware could not yet support reliably.

## Proposed UAV Platform

The platform is built on a low-cost ESP32 drone base, chosen over more established indoor research platforms (such as the Crazyflie) to keep per-unit cost low enough for swarm-scale experimentation. It was substantially modified to support indoor flight: beyond the stock MPU-6050 IMU (gyroscope + accelerometer), it adds a **PMW3901 optical-flow sensor** for horizontal velocity estimation and position-holding, and a **VL53L1X Time-of-Flight sensor** for altitude measurement, compensating for the absence of reliable GNSS indoors.

![UAV platform](figures/UAV_platform.png)
![Drone housing](figures/drone_housing.png)

## Firmware

The flight controller is written in C/C++ using **FreeRTOS** via ESP-IDF (v5.5) for the ESP32-S2, which assigns execution priorities so time-critical tasks such as IMU acquisition and motor control take precedence over lower-priority tasks like network communication. Attitude stabilisation uses a cascaded PID controller: a 500 Hz inner loop stabilises roll, pitch, and yaw from filtered IMU data, while an outer loop manages altitude and horizontal drift using optical-flow and ToF data.

The firmware is organised into modular components: a Sensor Data Collector aggregates peripheral readings and feeds them to a Drone State Manager, which tracks attitude, position estimate, and battery status; a Movement Control module uses this state to drive the PID loops and generate motor PWM signals; and a Telemetry & Communication module transmits UAV state as binary UDP payloads while a Commander module parses incoming movement commands from the GCS.

![UAV firmware data flow](figures/UAV_firmware_data_flow.png)

## Ground Control Framework

The GCS is a Python framework run on a dedicated control computer that provides three main functions: network scanning and UAV discovery via a parallelised ICMP ping sweep across the Wi-Fi subnet; real-time telemetry and state management through an asynchronous UDP dispatcher that decodes binary telemetry, maintains a unified global state, visualises it live, and logs it for post-flight analysis; and swarm coordination through a displacement-based spatial model that translates each UAV's local sensor frame into a shared global coordinate frame to compute the movement commands needed for group behaviours such as splitting, merging, and coordinated motion. A path-deconfliction procedure also checks each UAV's intended trajectory against others at similar altitude before committing to a move, to avoid mid-air conflicts.

![Framework architecture](figures/framework_architecture.png)
![Telemetry module](figures/telemetry_module.png)

## Simulation Model

A Webots model was developed to replicate the physical drone's key properties (weight, propeller size, and other physical characteristics) via a `.PROTO` file. This allowed swarm behaviours to be tested repeatedly and safely at scales the physical hardware could not yet support, isolating the coordination software from the hardware limitations described below.

![Webots drone model](figures/webots_drone.png)
![Webots simulation showcase](figures/webots_simulation_showcase.png)

## Key Results

- **Sensor accuracy**: the ToF sensor stayed accurate to within 0–3 cm (mean absolute error) at 20 cm and 50 cm target heights on both light and dark surfaces, degrading to 7 cm at 150 cm on dark surfaces. The optical-flow sensor was far more surface-dependent, with 37.47 cm error on plain white surfaces versus 1.53 cm on heavily patterned ones.
- **Physical flight trials**: the cascaded PID controller kept roll and pitch deviations under 2° across five 30-second flights, but reliable autonomous position hold was not achieved, due to gyroscope drift from the lack of a magnetometer, sensor-payload weight reducing available thrust margin, and vibration-induced optical-flow errors.
- **Network discovery and telemetry**: UAV discovery reliability reached 100% for one or two UAVs, dropping to 93.34% for three or four due to ICMP timeout congestion; UDP telemetry intervals stayed stable between 7.76–9.82 ms with zero dropped packets.
- **Simulation scalability**: swarms of up to 16 UAVs ran with no anomalies. At 32 and 64 UAVs, RAM usage rose sharply (up to ~4 GB) and command dispatch lag appeared, traced to a shared lock serialising outgoing UDP commands rather than to the underlying coordination algorithms, whose CPU cost stayed flat (6.1–7.6%) across all tested swarm sizes.

## Contact

- jakub.suda@tuke.sk
- jan.vascak@tuke.sk
