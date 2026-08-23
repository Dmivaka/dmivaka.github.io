---
title: "VBCores"
excerpt: "Reusable embedded hardware platform for low-volume robotics development<br/><img src='/images/vbcores/VBCore_preview.jpg'>"
collection: portfolio
---

[VBCores](https://vbcores.com/) is a modular embedded hardware platform I developed to avoid redesigning the same difficult parts of robot electronics for every new device.

The central idea was to move the MCU, power conversion, CAN-FD interface, and debugging circuitry onto a compact factory-made System-on-Module. Application-specific boards could then remain comparatively simple, inexpensive, repairable, and easy to assemble in low volumes.

The platform became the foundation of much of my robotics electronics work over the following several years. Hundreds of VBCore modules were manufactured and used across quadrupeds, balancing robots, large mobile platforms, educational robots, and university laboratory equipment.


## Why VBCores

The idea emerged after a visit to the European Rover Challenge, where a capable rover would require many different embedded devices: motor controllers, manipulator electronics, power systems, sensors, and scientific payload interfaces.

At the same time, developing the actuator electronics for Boxer made the manufacturing problem very clear. Repeatedly hand-assembling the MCU, clock, decoupling, power, and communication circuitry on every custom board was time-consuming, while the application-specific parts of many robot boards were comparatively easy to assemble and rework.

Low-volume production amplified the problem. A custom PCBA typically took at least a month to manufacture and deliver, so a design mistake that could not be repaired locally could cost another production cycle. Setup and handling costs also represented a large fraction of the price of small PCB batches.

The solution was to manufacture the complicated, repeated electronics in larger quantities and keep each application board as simple as possible.

This architecture removed the need to reimplement the MCU, low-voltage power, CAN-FD, clock, and debug sections for every new device while keeping the robot-specific electronics easy to prototype and repair.


## VBCore STM32G4 SoM

The main VBCore module is a compact 4-layer System-on-Module based on STM32G474RE.

Its main functions are:

- STM32G474RE MCU;
- robot-bus input up to 50 V;
- onboard 5 V and 3.3 V power rails available to the application board;
- integrated CAN-FD transceiver;
- standardized SWD and UART debug interface;
- castellated solder-down form factor.

The 50 V input range was chosen so that the same module could work directly from common robot battery buses, including 10S Li-ion systems.

STM32G474 was selected after experience with the STM32G4 family in Boxer. FDCAN, motor-control peripherals, CORDIC, and the broader STM32 ecosystem made it a strong general-purpose controller for robotics.

Early VBCore revisions used STM32F446 because it had stronger Arduino ecosystem support at the time. Later versions moved to STM32G474 so that educational compatibility would not constrain the capabilities of the long-lived robotics platform.


## Standardized hardware interface

The SoM uses 2.54 mm-pitch castellated holes and is soldered directly onto the application board.

This format kept assembly simple, added only about 1 mm to the PCB stack, and left the SoM-to-application-board signals accessible for probing during bring-up and debugging.

Application boards also followed consistent peripheral conventions for motor PWM, encoder interfaces, sensors, ADC feedback, communication, and debugging. Experience with one VBCores device therefore transferred naturally to another.

The same keyed SWD/UART debug connection was used across the platform, simplifying programming and standalone SoM testing.

Later module revisions added vias adjacent to the castellated pads and connected them through all four PCB layers. This made repeated rework more tolerant of damage to the edge plating.

One original use case was student-developed electronics for the European Rover Challenge, so the module also supported the standard STM32Duino workflow. The same hardware could therefore be used either through Arduino IDE or through a conventional STM32Cube/SWD development toolchain.


## Hardware built on the platform

The same SoM became the embedded core of several substantially different application boards.

### Motor control

A common hardware format was developed for [BLDC](https://vbcores.com/products/bldc-driver), [brushed DC](https://vbcores.com/products/dc-driver), and [stepper](https://vbcores.com/products/stepper-driver) motor controllers.

The boards shared the VBCore SoM, connector layout, encoder interfaces, and configuration EEPROM while using different power stages for the motor type.

I designed the electronics and developed the low-level bring-up and motor-control prototypes used to validate the boards.

The BLDC controller used in MORS was a related robot-specific design built around the same VBCore concept, but with mechanical and electrical interfaces tailored to the quadruped.

### PowerBoard

PowerBoard provided high-current robot power management with dual inputs, actuator-bus emergency shutdown, separate onboard-computer power, monitoring, and configurable robot I/O.

I designed the hardware and implemented the core power-control firmware.

[PowerBoard documentation](https://vbcores.com/products/powerboard)

### Ethernet–FDCAN bridge

A six-bus STM32H7/G4 Ethernet–FDCAN bridge developed for MORS later became the high-performance communication module in the VBCores hardware family.

I designed both the hardware and firmware, including Ethernet, CAN-FD, LwIP, and the real-time data paths used by MORS.

[Ethernet–FDCAN documentation](https://vbcores.com/products/ethernet-can)

### BHI360 IMU

I developed a BHI360/BMM350 inertial module together with a VBCore-based USB adapter running Bosch BHY2 firmware and TinyUSB.

The hardware provided a reusable high-rate inertial interface and was used successfully on MORS and GyroBro, although magnetic-heading performance remained dependent on the mechanical and electromagnetic environment of the robot.

[IMU documentation](https://vbcores.com/products/imu-bhi360)


## Where the platform was used

VBCores was used across several different robotic systems.

### MORS

The [MORS](/portfolio/1_mors/) quadruped used VBCore-based actuator controllers together with PowerBoard, Ethernet–FDCAN, and IMU hardware.

This was the most complete use of the platform: the same embedded foundation covered motor control, power, communication, and sensing.

{% include youtube.html id="X0VH79P7XvY" %}

### GyroBro

GyroBro is a compact balancing robot using BLDC hoverboard motors and a 10S battery. It used VBCores motor-control and IMU hardware as a research platform for balancing-control experiments.

{% include youtube.html id="0S3ydYjXFfI" %}

### Issledovatel

Issledovatel is a large all-terrain mobile robot using VBCores electronics for embedded motor and power control.

{% include youtube.html id="MS0JZKo-uWI" %}

<br>
VBCores hardware was also adopted in later educational six-wheel robots and in university laboratory equipment.


## Trade-offs

A common platform inevitably introduces constraints.

On PowerBoard, nearly every SoM pin was eventually used; even a two-bit configuration switch had to be encoded through a resistor network into the remaining ADC input.

VBCores was also unnecessary for some simpler systems. On small educational rovers, the computing power, 50 V architecture, and modular hardware could be excessive compared with a dedicated low-cost controller.


## Outcome

VBCores achieved its main engineering goal.

It became the embedded foundation for several years of robotics development and made one-off and low-volume systems practical without redesigning the same MCU, power, CAN, and debug infrastructure each time.

Hundreds of VBCore SoMs were manufactured and used across internal products, educational platforms, and laboratory equipment.

The platform was used primarily as an internal engineering foundation; external adoption as a standalone electronics product remained limited.

Its engineering value, however, was clear: new robot electronics no longer started with another copy of the same embedded foundation.


## Technical highlights

- STM32G474RE 4-layer castellated System-on-Module
- up to 50 V robot-bus input
- onboard 5 V and 3.3 V rails
- integrated CAN-FD transceiver
- standardized SWD + UART debug interface
- STM32Cube and STM32Duino development workflows
- common peripheral conventions across application boards
- BLDC, brushed DC, and stepper application hardware
- PowerBoard, Ethernet–FDCAN, and IMU modules
- hundreds of SoMs manufactured and reused across multiple robot classes
