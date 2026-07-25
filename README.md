# Simulated Real-Time Sensor Acquisition & Alert System (FreeRTOS)

A professional **FreeRTOS-based multitasking simulation** demonstrating real-time task scheduling, inter-task communication, synchronization, and software timers using the **official FreeRTOS POSIX simulator port**.

Unlike traditional embedded projects that require a development board, this project executes entirely on a Linux PC while using the **same FreeRTOS kernel APIs** that run on ARM Cortex-M microcontrollers. The POSIX port maps FreeRTOS tasks to POSIX threads, allowing development, debugging, and experimentation without hardware.

---

# Project Overview

This project simulates an industrial **temperature monitoring and alert system** consisting of multiple concurrent FreeRTOS tasks communicating through queues, mutexes, semaphores, and software timers.

The application demonstrates how a real embedded firmware application is structured, including:

- Periodic sensor acquisition
- Producer–Consumer communication
- Shared resource protection
- Event-driven interrupt-like processing
- Priority-based task scheduling
- Periodic health monitoring

Although the temperature sensor and cooling actuator are simulated, the FreeRTOS scheduler, synchronization primitives, task priorities, and timer mechanisms are identical to those used in production embedded systems.

---

# Features

- ✅ FreeRTOS Preemptive Scheduler
- ✅ Four Concurrent Tasks
- ✅ Three Priority Levels
- ✅ Producer–Consumer Architecture
- ✅ Queue-Based Inter-Task Communication
- ✅ Mutex-Protected Shared Resources
- ✅ Binary Semaphore Event Notification
- ✅ Periodic Software Timer
- ✅ FreeRTOS Hook Functions
- ✅ Modular Firmware Architecture
- ✅ POSIX Simulation (No Hardware Required)

---

# RTOS Concepts Demonstrated

| RTOS Concept | Implementation |
|--------------|----------------|
| Task Management | Sensor, Processing, Alert and Logging Tasks |
| Queue | Sensor Task → Processing Task |
| Mutex | Protect shared sensor data |
| Binary Semaphore | Processing Task → Alert Task |
| Software Timer | Periodic Health Monitoring |
| Scheduler | Fixed-priority preemptive scheduling |
| Timer Service Task | Executes Health Timer callback |
| Hook Functions | Memory allocation failure, stack overflow and assertions |

---

# System Architecture

```text
                         +----------------------+
                         |    Sensor Task       |
                         |   (Priority = 2)     |
                         +----------+-----------+
                                    |
                                    | Queue
                                    v
                         +----------------------+
                         | Processing Task      |
                         |  (Priority = 2)      |
                         +----------+-----------+
                                    |
                 +------------------+------------------+
                 |                                     |
                 | Mutex                               | Binary Semaphore
                 v                                     v
      +------------------------+           +------------------------+
      | Shared Sensor Reading  |           |      Alert Task        |
      | Latest Temperature     |           |   (Priority = 3)       |
      +------------+-----------+           +------------------------+
                   ^
                   |
                   | Mutex
                   |
      +------------+-----------+
      |    Logging Task        |
      |   (Priority = 1)       |
      +------------------------+

             Software Timer (Every 3 Seconds)
                         |
                         v
                 Timer Service Task
                         |
                         v
              Health Timer Callback
```

---

# Task Description

## 1. Sensor Task (Priority 2)

**Execution Type**

Periodic Task

**Period**

500 ms

**Responsibilities**

- Simulates temperature sensor readings
- Generates random temperature values
- Inserts readings into the FreeRTOS queue
- Acts as the Producer in the Producer–Consumer architecture

---

## 2. Processing Task (Priority 2)

**Execution Type**

Queue Consumer

**Responsibilities**

- Receives sensor readings from the queue
- Updates the shared system state
- Protects shared data using a mutex
- Detects threshold violations
- Signals the Alert Task using a binary semaphore

---

## 3. Alert Task (Priority 3)

**Execution Type**

Event Driven

**Responsibilities**

- Blocks indefinitely waiting on a binary semaphore
- Executes immediately when a threshold violation occurs
- Simulates activation of a cooling actuator
- Demonstrates priority-based preemption

---

## 4. Logging Task (Priority 1)

**Execution Type**

Periodic Background Task

**Period**

1500 ms

**Responsibilities**

- Reads the latest processed sensor value
- Uses mutex synchronization
- Prints system status
- Demonstrates deterministic periodic execution using `vTaskDelayUntil()`

---

# Software Timer

A FreeRTOS auto-reload software timer executes every **3 seconds**.

The timer callback executes in the **FreeRTOS Timer Service Task** and periodically reports:

- Remaining heap memory
- Total alert count
- Overall system health

This demonstrates periodic background processing without dedicating an application task.

---

# Synchronization Objects

| Primitive | Purpose |
|------------|---------|
| Queue | Transfer sensor readings between Sensor and Processing tasks |
| Mutex | Protect shared sensor data from concurrent access |
| Binary Semaphore | Notify Alert Task when threshold is exceeded |
| Software Timer | Execute periodic health monitoring |

---

# Priority Assignment

| Task | Priority | Execution Style |
|------|----------|----------------|
| Alert Task | 3 | Event Driven |
| Sensor Task | 2 | Periodic |
| Processing Task | 2 | Queue Consumer |
| Logging Task | 1 | Background Periodic |

The Alert Task has the highest priority, allowing it to preempt lower-priority tasks immediately after an alert event occurs.

---

# Project Structure

```text
Real-Time-Sensor-Monitoring-Simulator/
│
├── Config/
│   └── FreeRTOSConfig.h
│
├── include/
│   ├── config.h
│   ├── shared_data.h
│   ├── sensor_task.h
│   ├── processing_task.h
│   ├── alert_task.h
│   ├── logger_task.h
│   ├── health_timer.h
│   └── system_hooks.h
│
├── src/
│   ├── main.c
│   ├── shared_data.c
│   ├── sensor_task.c
│   ├── processing_task.c
│   ├── alert_task.c
│   ├── logger_task.c
│   ├── health_timer.c
│   └── system_hooks.c
│
├── build.sh
└── README.md
```

---

# Build Instructions

## Requirements

- GCC
- Git
- POSIX Threads (pthread)
- Linux or Windows Subsystem for Linux (WSL)

---

Clone the repository.

```bash
git clone <repository-url>
cd Real-Time-Sensor-Monitoring-Simulator
```

Build the project.

```bash
chmod +x build.sh
./build.sh
```

Run the simulation.

```bash
./sim
```

Terminate using:

```text
Ctrl + C
```

---

# Sample Output

```text
=== Simulated Real-Time Sensor Acquisition & Alert System ===
Running on FreeRTOS POSIX simulator (no hardware)

[Log    ] Sample #1 | Temp: 42.8C
[Log    ] Sample #4 | Temp: 74.0C

[Health ] System OK | Free heap: 849360 bytes | Total alerts so far: 0

[Log    ] Sample #7 | Temp: 67.9C

[ALERT!!] Temperature 75.1C exceeds threshold (75.0C)
          Simulated cooling actuator ENGAGED (alert #1)

[Log    ] Sample #10 | Temp: 48.5C

[Health ] System OK | Free heap: 849360 bytes | Total alerts so far: 1
```

Notice that the Alert Task executes immediately when the threshold is exceeded, interrupting the normal logging sequence. This demonstrates **priority-based preemptive scheduling** in FreeRTOS.

---

# What's Simulated vs Real Hardware

## Real

The following components are actual FreeRTOS functionality:

- FreeRTOS Kernel
- Scheduler
- Tasks
- Queue
- Mutex
- Binary Semaphore
- Software Timer
- Priority Scheduling
- Timer Service Task
- Hook Functions

---

## Simulated

The following hardware peripherals are simulated:

- Temperature Sensor (Random Number Generator)
- Cooling Actuator (Console Output)

---

## Porting to Real Hardware

Migrating this project to an STM32 or another ARM Cortex-M microcontroller requires only replacing the hardware-specific interfaces.

| Simulator | Embedded Hardware |
|------------|-------------------|
| Random Temperature | ADC / I²C / SPI Sensor |
| Console Output | UART |
| Console Alert | GPIO / Relay / Fan |
| POSIX Port | Cortex-M FreeRTOS Port |

The application architecture, scheduler, queues, mutexes, semaphores and software timers remain unchanged.

---

# Learning Outcomes

This project demonstrates practical experience with:

- FreeRTOS Task Scheduling
- Embedded Firmware Architecture
- Queue-Based Communication
- Mutex Synchronization
- Binary Semaphore Signaling
- Software Timers
- Hook Functions
- Producer–Consumer Design Pattern
- Event-Driven Firmware
- Real-Time Scheduling
- Modular Embedded Software Development

---

# Future Improvements

Potential extensions include:

- STM32 Cortex-M Port
- Real Temperature Sensor (LM35 / TMP102 / DHT22)
- UART Logging
- SPI/I²C Sensor Interface
- GPIO-Controlled Cooling Fan
- OLED/LCD Status Display
- SD Card Logging
- MQTT-Based IoT Monitoring
- CMSIS-RTOS Layer
- Unit Testing Framework

---

# References

- Official FreeRTOS Kernel
- FreeRTOS POSIX Simulator Port
- FreeRTOS API Documentation

---
