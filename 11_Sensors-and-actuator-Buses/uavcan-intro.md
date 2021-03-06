# UAVCAN Introduction

官网英文原文地址：http://dev.px4.io/uavcan-intro.html

![uavcan](../pictures/logos/uavcan-logo-transparent.png)

[UAVCAN](http://uavcan.org) is an onboard network which allows the autopilot to connect to avionics. It supports hardware like:

- Motor controllers
  - [Pixhawk ESC](https://pixhawk.org/modules/pixhawk_esc)
  - [SV2740 ESC](https://github.com/thiemar/vectorcontrol)
- Airspeed sensors
  - [Thiemar airspeed sensor](https://github.com/thiemar/airspeed)
- GNSS receivers for GPS and GLONASS
  - [Zubax GNSS](http://zubax.com/product/zubax-gnss)

In contrast to hobby-grade devices it uses rugged, differential signalling and supports firmware upgrades over the bus. All motor controllers provide status feedback and implement field-oriented-control (FOC).

## Initial Setup

The following instructions provide a step-by-step guide to connect and setup a quadcopter with ESCs and GPS connected via UAVCAN. The hardware of choice is a Pixhawk 2.1, Zubax Orel 20 ESCs and a Zubax GNSS GPS module.

### Wiring

The first step is to connect all UAVCAN enabled devices with the flight controller. The following diagram displays how to wire all components. The used Zubax devices all support a redundant CAN interface in which the second bus is optional but increases the robustness of the connection. 

![wire](../pictures/logos/UAVCAN_wiring.png)

It is important to mention that some devices require an external power supply \(e.g. Zubax Orel 20\) and others can be powered by the CAN connection \(e.g Zubax GNSS\) itself. Please refer to the documentation of your hardware before continuing with the setup.


### Firmware Setup

Next, follow the instructions in [UAVCAN Configuration](../11_Sensors-and-actuator-Buses/uavcan-node-enumeration.md) to activate the UAVCAN functionalities in the firmware. Disconnect your power supply and reconnect it. After the power cycle all UAVCAN devices should be detected which is confirmed by a beeping motor on the Orel 20 ESCs. You can now continue with the general setup and calibration. 

Depending on the used hardware, it can be reasonable to perform an update of the firmware on the UAVCAN devices. This can be done via the UAVCAN itself and the PX4 firmware. For more details please refer to the instructions in [UAVCAN Firmware](../11_Sensors-and-actuator-Buses/uavcan-node-firmware.md).

## Upgrading Node Firmware

The PX4 middleware will automatically upgrade firmware on UAVCAN nodes if the matching firmware is supplied. The process and requirements are described on the [UAVCAN Firmware](../11_Sensors-and-actuator-Buses/uavcan-node-firmware.md) page.

## Enumerating and Configuring Motor Controllers

The ID and rotational direction of each motor controller can be assigned after installation in a simple setup routine: [UAVCAN Node Enumeration](../11_Sensors-and-actuator-Buses/uavcan-node-enumeration.md). The routine can be started by the user through QGroundControl.

## Useful links

* [Homepage](http://uavcan.org)
* [Specification](http://uavcan.org/Specification)
* [Implementations and tutorials](http://uavcan.org/Implementations)


