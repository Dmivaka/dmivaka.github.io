---
title: "TurtleBro"
excerpt: "ROS educational mobile robot produced in several hundred units<br/><img src='/images/tb/tb_preview.jpg'>"
collection: portfolio
---

TurtleBro is a ROS-based educational mobile robot developed for universities, robotics courses, and hands-on learning.

The platform evolved continuously between 2018 and 2025 through more than 10 main-controller revisions, and several hundred robots and related mobile platforms were manufactured during that period.

TurtleBot 3 served as an early reference platform, while TurtleBro was developed around more widely available drivetrain components, easier battery servicing, and a separate student-programmable subsystem. My responsibility was to implement and evolve the embedded electronics and firmware behind this architecture.

**2018–2025 · 10+ controller revisions · several hundred robots and related platforms · 4-layer / 400+ component later-generation controller**

[Official TurtleBro product page (Russian)](https://voltbro.ru/turtlebro) · [ROS course and robot documentation (Russian)](https://voltbro.gitbook.io/new_bazovyi-kurs-po-ros/lesson1/robot)

{% include youtube.html id="jQnDnTkWb7U" %}


## My responsibility

I was responsible for TurtleBro's embedded electronics and firmware from the first prototypes in 2018 through the final production revisions in 2025.

My work included:

- defining the embedded system architecture and designing more than 10 revisions of the main controller electronics;
- developing and maintaining the STM32F4 firmware using STM32Cube HAL/LL and FreeRTOS;
- implementing closed-loop motor control, wheel odometry, IMU and battery interfaces, status indication, and low-level system functions;
- integrating the STM32 controller with ROS using rosserial and later micro-ROS;
- integrating a separate Arduino-compatible student-programmable subsystem;
- designing the six-motor expansion controller used in larger mobile platforms;
- bringing up and debugging prototype and production electronics;
- supporting PCB manufacturing, configuration, flashing, incoming testing, rework, component substitutions, and engineering changes.


## Embedded architecture

The main controller combined the robot's low-level control electronics, power system, student-programmable subsystem, and internal communication infrastructure on a single PCB.

The **STM32F4** was the system controller. It handled two brushed DC motors with encoder feedback, PID velocity control and wheel odometry, communication with the BNO055 IMU, battery voltage/current measurement, external-power detection, undervoltage protection, status indication, and communication with the six-motor expansion board.

A separate **Arduino-compatible subsystem** was implemented directly on the controller PCB and exposed standard headers and student I/O. Students could modify Arduino code independently while the STM32 firmware responsible for motion, power management, and system functions remained unchanged. Early revisions were Mega-compatible; later generations moved to a Due-compatible implementation.

An onboard **USB hub** connected separate serial-over-USB interfaces for the STM32 controller, Arduino subsystem, and RPLIDAR A1 to the Raspberry Pi. The STM32 ROS interface initially used rosserial and was later migrated to micro-ROS.

<a href="/images/tb/tb_controller_v13_annotated_ru.png" target="_blank" style="border: none; outline: none; background: none; box-shadow: none;">
  <img src="/images/tb/tb_controller_v13_annotated_ru.png" alt="Annotated first-generation TurtleBro controller">
</a>

*First-generation 2-layer TurtleBro controller. This product illustration uses Russian labels and shows the STM32 system controller, motor and power electronics, integrated Arduino subsystem, USB hub, LiDAR interfaces, and expansion connector.*


## Battery and power system

The robot used a removable **4S battery cassette built from standard 18650 cells**. The cassette could be removed from the chassis without disassembling the robot, making battery replacement straightforward.

The main controller measured battery voltage and current and reported them to ROS. A four-level indicator and buzzer warned the user as the battery approached undervoltage shutdown. A dedicated 5 V / 3 A buck converter powered the Raspberry Pi, initially through USB-A and later through USB-C.

Early controller revisions could switch between battery and external DC power, allowing a depleted battery cassette to be replaced while the robot remained powered. Later revisions added onboard charging and cell balancing.


## Hardware evolution

The main controller went through more than **10 hardware revisions over seven years**. Early versions were 2-layer boards; later generations became 4-layer production PCBs with more than 400 components.

The electronics evolved together with the product: battery charging was added, the educational controller moved from Mega to Due, ROS integration moved from rosserial to micro-ROS, and boards were repeatedly revised in response to user feedback, manufacturing experience, new functionality, component shortages, and supplier changes.

This long product lifetime was as important to the project as the initial design. The same embedded architecture had to remain understandable, manufacturable, repairable, and extensible while both the electronics market and product requirements changed around it.


## Six-motor expansion

I designed a six-motor expansion board with higher-current drivers and a dedicated STM32 controller for larger mobile platforms.

The expansion controller received motor commands from the TurtleBro main controller over UART using Protocol Buffers, handled closed-loop motor control locally, and returned wheel odometry.

This allowed the TurtleBro embedded architecture to be reused in larger six-wheel platforms without moving ROS integration, power monitoring, and system-management functions away from the main controller.


## Production

I hand-assembled the prototype electronics and the first few dozen commercial controller boards. As production volumes increased, PCB assembly moved to contract manufacturers, with typical incoming batches of roughly 60–150 boards.

I prepared BOM and pick-and-place data, supported manufacturing questions and component substitutions, and handled the engineering side of incoming electronics.

The supplier performed basic power-up and test-firmware checks. After delivery I configured the USB-to-UART interfaces, flashed and tested the boards, performed rework where necessary, and prepared the electronics for final robot assembly and system-level testing.

Over the product lifetime, manufacturing support became an ongoing part of the engineering work rather than a one-time handoff: controller revisions had to account for component availability, assembly issues, and lessons learned from previous production batches.


## Technical highlights

- STM32F4 + STM32Cube HAL/LL + FreeRTOS
- 10+ main-controller revisions over 7 years
- 400+ components and 4-layer PCB in later generations
- brushed DC motor control with encoder feedback, PID velocity loops, and odometry
- BNO055 IMU integration
- 4S Li-ion battery monitoring, protection, and later integrated charging/balancing
- removable battery cassette based on standard 18650 cells
- rosserial and later micro-ROS
- onboard USB hub with separate serial-over-USB interfaces for STM32, Arduino, and LiDAR
- integrated Arduino Mega / Due-compatible student subsystem
- six-motor expansion architecture using STM32 + Protocol Buffers over UART
- several hundred manufactured robots and related mobile platforms


## Selected development stages

![First PCB for firmware development](/images/tb/tb_first_pcb.jpg)
*Early controller PCB used for firmware development with an external STM32 development board.*

![Early TurtleBro prototype](/images/tb/tb_TB3_comparison.jpg)
*Early TurtleBro prototype (left) alongside a TurtleBot 3 reference platform (right). The comparison shows the compact form factor targeted during early development.*

![Hand soldering of early production electronics](/images/tb/tb_hand_prod.jpg)
*Hand soldering controller boards for an early commercial batch.*

![Six-motor expansion controller](/images/tb/tb_shield_prod.jpg)
*First production batch of the six-motor expansion controller.*

![Six-wheel mobile platform](/images/tb/tb_rover_devel.jpg)
*The same embedded architecture reused in a larger six-wheel mobile platform.*

![Serial production TurtleBro robots](/images/tb/tb_serial_prod.jpg)
*Production robots with contract-manufactured electronics.*

![Later controller with charging](/images/tb/tb_2_gen.jpg)
*Prototype of a later controller generation with integrated battery charging.*

<a href="/images/tb/tb_28_top.jpg" target="_blank" style="border: none; outline: none; background: none; box-shadow: none;">
  <img src="/images/tb/tb_28_top.jpg" alt="Latest TurtleBro controller revision">
</a>

*One of the latest 4-layer controller revisions.*
