---
title: "MORS"
excerpt: "Low-volume production compact quadruped robot for education and robotics research<br/><img src='/images/mors/mors_preview.jpg'>"
collection: portfolio
---

MORS is a lightweight 12-DoF quadruped robot developed as a compact, reliable platform for robotics research and low-volume production.

The project followed the earlier Boxer quadruped. Boxer proved the basic actuator and control architecture, but remained a heavy research prototype. MORS was intended to be a much more repeatable machine: compact enough to transport easily, mechanically designed as a complete product, and reliable enough to serve as a stable platform for advanced locomotion research.

My work covered the embedded electronics, actuator firmware, robot communication hardware, and much of the electronics integration. Mechanical design and high-level locomotion control were outside my scope.

A [paper](https://www.researchgate.net/publication/389319462_MORS_BLDC-Based_Small_Quadruped_Robot) about MORS is available.  
The robot also has its own [Instagram](https://www.instagram.com/mors_quadruped/).

{% include youtube.html id="X0VH79P7XvY" %}


## From Boxer to MORS

A major change from Boxer was the use of commercially available 4310 BLDC motors with integrated 10:1 planetary gearboxes. Their size and performance defined much of the robot geometry and removed the need for the expensive custom gearboxes used in Boxer.

Electronics packaging and joint-position sensing were developed together with the mechanical layout so that board size, wiring, mounting, and sensor geometry were considered before parts were manufactured.

MORS also became the first quadruped built around the **VBCores** modular electronics architecture. Instead of repeating the MCU, power supply, and CAN interface on every board, the robot reused the same STM32G4 System-on-Module across actuator, power, sensing, and communication hardware.

Several subsystems that matured during MORS development—including the PowerBoard, Ethernet–FDCAN bridge, and IMU hardware—later became reusable parts of the wider [VBCores](https://vbcores.com/) ecosystem.


## My responsibility

I was responsible for nearly the complete embedded electronics stack of MORS.

My work included:

- second-generation BLDC actuator-controller electronics;
- STM32G4 FOC firmware evolved from the Boxer actuator stack;
- absolute joint-position sensing;
- PowerBoard power-management and safety hardware;
- Ethernet–FDCAN communication between the onboard Linux computer and actuator networks;
- BHI360-based IMU hardware and USB interface;
- UAVCAN and LCM communication infrastructure;
- flashing, calibration, wiring, electronics integration, and system bring-up for the production batch.


## Electronics architecture

MORS used a distributed electronics architecture built around VBCores modules.

The robot was powered by a 6S Li-ion battery through the PowerBoard. Four separate CAN-FD networks connected the four legs, while a fifth CAN-FD bus connected the PowerBoard. An Ethernet–FDCAN bridge linked these real-time networks to the onboard Linux computer.

The onboard computer also received IMU and RC-control interfaces over USB.

<a href="/images/mors/MORS_elec_scheme_3.png" target="_blank" style="border: none; outline: none; background: none; box-shadow: none;">
  <img src="/images/mors/MORS_elec_scheme_3.png" alt="MORS electronic architecture">
</a>

*Electronic architecture of MORS. Blocks highlighted in green were designed and implemented by me.*


## Actuator control and joint sensing

Each actuator used a 4310 BLDC motor with a 10:1 planetary reduction and a VBCore-based motor controller located inside the robot body.

The actuator firmware evolved from the Boxer control stack and added:

- tuned current-loop parameters for the new motors;
- UAVCAN communication instead of the custom Boxer CAN-FD protocol;
- support for motor-side and external joint encoders;
- a common encoder-calibration procedure;
- EEPROM storage for individual motor calibration data and configuration.

The controller operated from the 6S battery system with three-phase Hall-effect current sensing. PWM generation and the FOC current loop ran synchronously at 40 kHz.

Each joint used an internal AS5047 motor encoder together with a second absolute joint encoder used during startup to establish the mechanical joint offset.

Magnetic AS5047 encoders were used for hip abduction/adduction and knee flexion/extension. Hip flexion/extension required a custom inductive encoder based on the AS5715.


## Real-time communication

The communication architecture grew out of a practical problem: the PCIe CAN-FD interface used in Boxer was expensive and difficult to source, while an evaluated open USB-CAN implementation showed occasional latency spikes of tens or even hundreds of milliseconds.

An Ethernet prototype on STM32F4 showed sub-millisecond response, leading to a dedicated bridge based on STM32H723 and STM32G474.

The final board provided six independent CAN-FD buses; MORS used five. Four buses served the four legs and preserved a linear topology on each leg, while the fifth connected the PowerBoard.

The bridge exposed two communication paths:

- **Real-time actuator traffic:** actuator setpoints and feedback were aggregated by the bridge and exchanged with Linux through LCM.
- **Background configuration and diagnostics:** ordinary UAVCAN traffic was forwarded to Linux through virtual CAN interfaces.

From the actuator side, communication remained UAVCAN-based; LCM existed only on the Ethernet/Linux side of the bridge.

This kept the fast 12-actuator command/feedback path lightweight while preserving ordinary UAVCAN access for configuration and diagnostics.


## Power and safety

MORS used the VBCores PowerBoard for power distribution and hardware safety functions.

One high-current input was connected to the onboard battery, while the second was routed to an external connector for tethered bench operation.

A hardware E-stop disconnected the actuator bus independently of application software while keeping the onboard computer powered for diagnostics.

The PowerBoard was intended for the wider VBCores ecosystem, but several practical features—including configurable 12 V GPIO for buttons, indicators, and buzzers—were refined during MORS integration.


## IMU

Earlier robot generations used the Bosch BNO055. During MORS development I evaluated replacements and selected the BHI360 for its higher update rate and supported Bosch SDK.

I ported Bosch's BHY2 software to STM32 and developed a BHI360 + BMM350 module together with a VBCore-based USB adapter running TinyUSB.

The BMM350 magnetometer proved unreliable inside MORS because of interference from the twelve electric motors, so the robot primarily used accelerometer/gyroscope data and the BHI360 game-rotation-vector output rather than magnetic heading.


## From one robot to ten

Unlike Boxer, MORS was designed for repeatable low-volume production rather than as a one-off research prototype.

The complete mechanical structure was designed in CAD before production, custom aluminum parts were manufactured externally, motors and gearboxes were purchased as repeatable assemblies, and PCBAs were outsourced.

Final integration was still largely manual. For the ten-robot batch I assembled and installed the electronics, completed part of the wiring, flashed and configured embedded devices, calibrated motor and joint encoders, installed Linux on the onboard computers, and brought each electronics system to a working state before handoff for high-level control integration.

![First Ethernet-FDCAN PCB](/images/mors/mors_first_eth.jpg)
*First Ethernet–FDCAN PCB during hardware bring-up.*

![Actuator controllers](/images/mors/mors_controller_cassette.jpg)
*First power-on test of a group of body-mounted actuator controllers.*

![First prototype](/images/mors/mors_first_assembly.jpg)
*Electronics integration on the first prototype.*

![Electronics carrier](/images/mors/mors_proper_carrier.jpg)
*Electronics installed on the structural carrier of the second prototype.*

![Pre-production MORS](/images/mors/mors_second_assembly.jpg)
*Work on the pre-production robot.*

![Motor characterization](/images/mors/mors_kv_meas.jpg)
*Measuring motor KV during actuator development.*

![Production electronics](/images/mors/mors_body_electronics.jpg)
*Body electronics during production assembly.*

![Production batch](/images/mors/mors_serial_prod.jpg)
*Eight robots from the ten-unit production batch during assembly.*


## What the production batch changed

Building ten robots exposed problems that were much less visible on a single prototype.

Body-mounted actuator controllers required long multi-conductor cables to the motor and joint encoders. A batch of purchased 7-pin GH1.25 cables also suffered from poor crimps, turning wiring into a significant source of assembly and debugging work.

The custom inductive hip encoder exposed another integration problem. Motor operation coupled enough interference through the metallic structure to corrupt its readings. Because the joint position was only required during initialization, I introduced a short delay before energizing the motors so the encoder could be sampled while the system was electrically quiet.

The workaround was sufficient for the production batch, but the larger lesson was architectural:

1. actuators should be self-contained and identical wherever possible;
2. actuator calibration should be completed before installation into the robot;
3. long, fine-pitch encoder harnesses should be eliminated.


## Next-generation integrated actuator

Based on these lessons, I designed a third-generation actuator controller intended for future MORS revisions and other compact robotic joints.

The new design moved the electronics into the actuator itself and combined:

- an STSPIN32G4 motor-control MCU/gate-driver;
- integrated MOSFET power stage;
- motor-side encoder interface;
- NCV77320-based inductive output-shaft sensing;
- flex-PCB encoder coils;
- daisy-chain power and CAN-FD connections.

Only power and CAN-FD needed to leave the actuator, eliminating the long motor and encoder harnesses used in the production MORS.

The integrated actuator was completed as a hardware development but was not deployed in the original ten-robot production batch.

<a href="/images/mors/mors_drive_collage.jpg" target="_blank" style="border: none; outline: none; background: none; box-shadow: none;">
  <img src="/images/mors/mors_drive_collage.jpg" alt="Integrated actuator electronics">
</a>

*Integrated actuator electronics developed for a future MORS revision.*


## Technical highlights

- 12 DoF
- 7.4 kg robot mass
- 5.5 kg tested payload
- 1.2 m/s maximum speed
- approximately 60 minutes of no-payload outdoor operation in a maximum-speed test
- 4310 BLDC motors with 10:1 planetary reduction
- 6.1 Nm maximum actuator output torque
- STM32G4 FOC actuator controllers
- internal motor encoder + absolute joint-position sensing
- 6S Li-ion power system
- hardware emergency-stop actuator-bus disconnect
- five CAN-FD networks used on the robot
- Ethernet–FDCAN / LCM / UAVCAN communication architecture
- BHI360 IMU
- ten-unit low-volume production batch
