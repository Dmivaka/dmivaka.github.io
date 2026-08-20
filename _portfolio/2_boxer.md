---
title: "Boxer"
excerpt: "Research quadruped built around a custom BLDC actuator and embedded-control stack<br/><img src='/images/boxer/boxer_preview.png'>"
collection: portfolio
---

Boxer is a 12-DoF quadruped robot developed as an internal research project and technology demonstrator.

The project was inspired by MIT Mini Cheetah and its published actuator design. The initial goal was simply to determine whether a custom underactuated quadruped could be developed with the available components and manufacturing capabilities; there was no commercial requirement and no certainty that the project would work.

For me, Boxer marked the transition from embedded electronics for mobile robots to a complete robotic actuator stack: motor-control hardware, FOC firmware, sensing, CAN-FD communication, Linux integration, and robot-level power electronics.

A [paper](https://www.researchgate.net/publication/362930025_CPG-Based_Gait_Generator_for_a_Quadruped_Robot_with_Sidewalk_and_Turning_Operations) based on later work with Boxer is available.

{% include youtube.html id="bFAk3QXRYn4" %}


## Starting from Mini Cheetah

Mini Cheetah provided a known working reference for the first Boxer actuator.

Boxer adopted the same general actuator class: 8108 BLDC motors and approximately 6:1 reduction. I first reproduced Ben Katz's published motor controller and FOC implementation using STM32F446, DRV8302, and AS5047P, then gradually adapted the electronics and firmware to the components, manufacturing methods, onboard computer, and communication hardware available for Boxer.

The mechanical architecture diverged substantially from Mini Cheetah. Locally manufactured drivetrain components and a simpler gearbox arrangement resulted in a bulkier and heavier actuator, while allowing the project to be built with available manufacturing processes.


## My responsibility

I was responsible for Boxer's embedded electronics and firmware from 2019 to 2022.

My work included:

- BLDC actuator-controller schematic and PCB design;
- porting and evolving the FOC firmware from the published Mini Cheetah implementation;
- motor-position sensing and calibration;
- robot-level power-management electronics;
- four-bus CAN-FD communication;
- Linux configuration and the initial CAN-to-LCM interface;
- hardware bring-up, actuator calibration, and electronics maintenance;
- an early open-loop walking demonstration for end-to-end system validation.

Mechanical design and later locomotion-control development were outside my responsibility.


## From reference design to Boxer actuator electronics

The published Mini Cheetah controller gave me a working baseline, but the Boxer actuator electronics gradually became a design I understood and could maintain independently.

The mature Boxer controller moved to STM32G4, mainly for native FDCAN support and the integrated CORDIC accelerator. With CORDIC acceleration, the complete 40 kHz control loop used roughly 60% of the available processing time.

Current sensing also changed substantially. I redesigned the controller around three-phase inline current measurement. An intermediate Hall-effect implementation proved sensitive to the stray magnetic field from the rotor-position magnet, so the final controller moved to shunts with INA240 current-sense amplifiers.

The motor-position sensor also changed from AS5047P to Allegro A1339, whose effective update rate was better suited to the 40 kHz control loop.

![First in-house BLDC controller](/images/boxer/boxer_first_bldc.jpg)
*An early in-house developed BLDC controller.*


## Motor-control firmware

Ben Katz's reference firmware was built around Mbed OS. I reimplemented the actuator firmware using STM32Cube HAL/LL and an interrupt-driven control architecture.

Later revisions added:

- STM32G4 CORDIC-based trigonometric calculations;
- FDCAN communication;
- ADC oversampling, which noticeably reduced actuator acoustic noise;

The current loop was limited to 30 A peak phase current, a limit the actuators regularly reached during walking.

![FOC development setup](/images/boxer/boxer_firmware_setup.jpg)
*Early setup used while porting and validating the FOC firmware.*


## Encoder calibration

Encoder non-linearity was characterized over a full mechanical revolution in both directions. The resulting periodic correction was reduced to a 128-point lookup table, with the electrical phase offset calibrated separately for each actuator.

![Encoder calibration](/images/boxer/boxer_calibration.png)
*Example encoder-calibration data used to generate the per-actuator non-linearity compensation table.*


## Robot communication and Linux integration

Boxer used an Intel NUC as its onboard computer.

The NUC connected to the actuator networks through a M.2 PCIe CAN-FD interface. Four independent CAN-FD buses were used, one per leg, with the robot power controller also connected to the first bus.

All buses used a linear topology and operated at 1 Mbit/s nominal and 8 Mbit/s data rate.

I installed Ubuntu, built a real-time patched kernel, configured the CAN-FD interfaces, and implemented the initial Linux-side CAN-to-LCM bridge.

The resulting data path was:

**Actuator STM32 → CAN-FD → PCIe CAN interface → Linux → LCM**


## Robot power electronics

I also developed the robot-level power board, providing battery voltage/current monitoring, undervoltage shutdown, inrush-current limiting, power distribution to the four legs, and a commercial Murata DC/DC supply for the onboard computer.

Several of these requirements were revisited more systematically in the later VBCores PowerBoard.


## Getting the robot to walk

To validate the complete actuator, communication, and kinematic stack, I implemented a simple open-loop gait using half-cycloidal foot trajectories. The first version was prototyped in Python and later ported to C on Linux.

It was intentionally a system-validation demonstration rather than a locomotion controller; later closed-loop locomotion software was developed separately.


## Joint initialization

One of Boxer's persistent usability problems was joint initialization.

The motor-side encoder provided the rotor position, but the system could not reliably determine the absolute mechanical joint configuration after the gearbox at startup.

I prototyped an optical output-shaft encoder using three IR LED/photodiode pairs and a patterned rotating PCB. The concept worked as a standalone device, but drivetrain deflection under load changed the optical spacing enough to make the sensor unreliable when installed in the robot.

The prototype was abandoned, and Boxer continued to require a particular startup pose.

This failure directly influenced MORS: deterministic absolute joint-position sensing became a design requirement from the beginning rather than an add-on after the mechanical system was already built.


## Lessons carried into MORS

Boxer proved that a custom underactuated quadruped could be built and controlled, but it also exposed the difference between a research prototype that can walk and a robot that is practical to use repeatedly.

The main lessons were:

- absolute joint position should be available at startup;
- wiring and serviceability need to be designed together with the mechanics;
- actuator electronics should be reproducible and easy to replace;
- the complete robot architecture should be designed for repeatable operation, not merely for a successful prototype.

These lessons became starting requirements for the next quadruped generation, [MORS](/portfolio/1_mors/).


## Technical highlights

- 12 DoF
- 14 kg robot mass
- eX8108 BLDC motors with custom 6:1 planetary reduction
- 16 Nm peak actuator output torque
- STM32G4 actuator controllers
- 40 kHz FOC current loop
- 30 A peak phase-current limit
- three-phase inline current sensing with INA240 amplifiers
- A1339 motor-position sensing
- four independent CAN-FD buses at 1 / 8 Mbit/s
- X86 onboard computer
- M.2 PCIe CAN-FD interface
- 6S2P Li-ion battery
- 0.39 m/s measured maximum speed


## Selected development stages

![Completed actuator electronics](/images/boxer/boxer_electronics.jpg)
*Completed actuator assembly with an early revision of the robot power-distribution board.*

![PCB batch](/images/boxer/boxer_pcb_batch.jpg)
*Final Boxer controller PCBs assembly.*

![Robot assembly](/images/boxer/boxer_assembled.jpg)
*All four legs attached to the body for the first time.*

![Maintenance](/images/boxer/boxer_maintenance.jpg)
*Hardware maintenance during the research phase.*

![Completed Boxer](/images/boxer/boxer_staged.jpg)
*Boxer in its completed research-platform configuration.*
