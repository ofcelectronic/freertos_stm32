# 🧠 FreeRTOS1 Project

This project demonstrates the basic usage of **FreeRTOS** on the **STM32F429ZI** microcontroller.  
The main focus is understanding how to create tasks (threads), manage their priorities, and configure the system timebase correctly when using FreeRTOS.

---

## ⚙️ Overview

During this project, we explored the following key concepts:

- How **FreeRTOS** uses the **SysTick** timer as its internal timebase.  
- Why **HAL timebase** must be redirected to another timer (e.g., TIM6 or TIM7) when FreeRTOS is enabled.  
- How to create FreeRTOS tasks both via the **IOC (CubeMX configuration)** and manually in code.  
- How to define, create, and link tasks using **thread IDs** and corresponding task functions.

---

## 🧩 Task Creation

Each FreeRTOS task (thread) follows this creation flow:

1. Define a **Thread ID** using `osThreadId_t`.
2. Write a **task function** that performs the desired operation.
3. Define the thread attributes (name, priority, stack size).
4. Create the thread using `osThreadNew()` and link it with the task function.

Example:
```c
osThreadId_t Task1Handle;
const osThreadAttr_t Task1_attributes = {
  .name = "Task1",
  .priority = (osPriority_t) osPriorityNormal,
  .stack_size = 128 * 4
};

Task1Handle = osThreadNew(StartTask1, NULL, &Task1_attributes);
```

---

## 🔄 Task Scheduling Experiment

In this project, **three tasks** were created — all with the **same priority** level.  
The behavior of the FreeRTOS scheduler was then observed using **UART debug messages**.

### Observations:

- When all tasks have the **same priority**, the scheduler runs them **in a round-robin manner**, giving the impression of random execution order.  
- When priorities are **modified**, tasks with **higher numeric priority values (higher priority)** are executed first.  
- Tasks with lower priority only run when higher-priority tasks enter the blocked or waiting state.

This experiment clearly demonstrates how **FreeRTOS handles task scheduling and priority preemption**.

---

## 🕒 Timebase Configuration

- **SysTick** is automatically used by FreeRTOS as the system tick timer.  
- Therefore, HAL timebase functions (like `HAL_Delay()` or `HAL_GetTick()`) **must not** use SysTick.
- To prevent conflicts, the HAL timebase source should be changed to another timer such as **TIM6** or **TIM7** in STM32CubeMX.

---

## 🧰 Development Environment

| Tool | Description |
|------|--------------|
| **MCU** | STM32F429ZI |
| **IDE** | STM32CubeIDE |
| **Framework** | FreeRTOS CMSIS-RTOS v2 |
| **Clock** | 168 MHz |
| **Debugger** | ST-Link v2 |

---

## 🧠 Key Learnings

- FreeRTOS manages system timing through **SysTick**, requiring HAL to use a different timebase.  
- Task creation can be done **manually or through CubeMX-generated code**.  
- When multiple tasks share the same priority, the scheduler alternates between them in a **time-sliced** fashion.  
- Priority levels directly influence which task gets CPU time first.

---

## ✍️ Author


**Project Name:** FreeRTOS1  
**MCU:** STM32F429ZI  
**Created by:** OFC  
📅 **Last Updated:** October 2025
