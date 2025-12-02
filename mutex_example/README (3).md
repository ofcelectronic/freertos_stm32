# 📘 FreeRTOS Mutex Example --- STM32 + UART

## ✨ Introduction

This project demonstrates how to use a **FreeRTOS mutex** to safely
access a shared resource (UART1) from multiple tasks.\
Two tasks with different priorities attempt to transmit over UART, and a
mutex ensures exclusive access.

The project also shows **priority inheritance**, an important mechanism
that prevents priority inversion when a high-priority task is waiting
for a mutex held by a lower-priority task.

------------------------------------------------------------------------

## 🔐 What is a Mutex?

A **mutex (Mutual Exclusion)** is a synchronization object used to
prevent multiple tasks from accessing a shared resource at the same
time.

### Key Properties

-   Only **one task at a time** can hold the mutex\
-   A task **owns** the mutex until it releases it\
-   Other tasks attempting to take it will be **blocked**\
-   Supports **priority inheritance**

### Why Use a Mutex?

Because UART (or any shared peripheral) cannot safely be accessed from
multiple tasks at the same time.

Without a mutex: - UART messages get corrupted - Tasks interrupt each
other while transmitting - System becomes unpredictable

------------------------------------------------------------------------

## 🚦 Priority Inheritance

When: - A low-priority task **holds the mutex** - A high-priority task
**tries to take it** - High-priority task becomes **blocked**

➡ FreeRTOS temporarily **raises the priority** of the low-priority task.

This prevents: - Low task being preempted by medium tasks\
- High task waiting too long\
- Classic **priority inversion**

------------------------------------------------------------------------

## 🧰 Project Overview

### Tasks Used

  Task Name   Priority     Description
  ----------- ------------ -----------------------------------
  HPT_Task    3 (High)     Prints messages through UART
  MPT_Task    2 (Medium)   Also prints messages through UART
  LPT_Task    *Not used*   Could run at a lower priority

### Shared Peripheral

-   **UART1**

### Synchronization

-   **Mutex:** `SimpleMutex`

------------------------------------------------------------------------

# 🧩 Code Flow Explanation

## 🟦 1. Creating the Mutex

``` c
SimpleMutex = xSemaphoreCreateMutex();

if(SimpleMutex != NULL)
{
    HAL_UART_Transmit(&huart1, (uint8_t*)"Mutex Created\n\n", 15, 1000);
}
```

The mutex is created and ready for all tasks to use.

------------------------------------------------------------------------

## 🟦 2. Creating the Tasks

``` c
xTaskCreate(HPT_Task, "HPT", 128, NULL, 3, &HPT_Handler);
xTaskCreate(MPT_Task, "MPT", 128, NULL, 2, &MPT_Handler);
```

-   HPT has higher priority than MPT\
-   HPT always preempts MPT unless MPT is in a **critical section
    protected by mutex**

------------------------------------------------------------------------

## 🟦 3. Critical Section (UART Protected Function)

``` c
void Sende_Uart(char *str)
{
    xSemaphoreTake(SimpleMutex, portMAX_DELAY);   // Lock mutex
    HAL_UART_Transmit(&huart1, (uint8_t *)str, strlen(str), HAL_MAX_DELAY);
    HAL_Delay(2000);                              // Simulated long work
    xSemaphoreGive(SimpleMutex);                  // Release mutex
}
```

### What happens here?

-   Task tries to take the mutex
-   If available → enters section\
-   If not → waits (BLOCKED state)
-   After finishing its UART operation, the task releases the mutex

------------------------------------------------------------------------

# ⚡ Priority Inheritance in Action (Important Behavior)

### Scenario:

1.  **MPT** runs first and takes the mutex\
2.  **HPT** becomes ready\
3.  HPT tries to take mutex → **blocked**\
4.  FreeRTOS temporarily boosts **MPT to priority 3**\
5.  MPT finishes its critical section, releases mutex\
6.  Scheduler immediately switches to **HPT**\
7.  HPT prints its message\
8.  MPT continues afterwards

------------------------------------------------------------------------

## 🟩 Why Does "leaving MPT" Appear *After* HPT Message?

Because the moment MPT releases the mutex:

-   HPT is unblocked\
-   HPT has priority 3\
-   Scheduler immediately switches to HPT\
-   MPT gets CPU time **only after** HPT finishes

This is **expected and correct behavior**.

------------------------------------------------------------------------

# 📊 Example Output Order

Typical UART output order:

    Entered MPT and about to take Mutex
    IN MPT ======================

    [Mutex released → HPT takes over instantly]

    Entered HPT and about to take Mutex
    IN HPT ======================
    leaving HPT

    [After HPT finishes → MPT continues]
    leaving MPT

This output confirms: - Mutex protection works\
- Priority inheritance works\
- High-priority task takes over immediately after mutex release

------------------------------------------------------------------------

# 🏁 Summary

### ✔ What you learned

-   How a FreeRTOS mutex works\
-   How tasks safely share UART\
-   How priority inheritance prevents priority inversion\
-   Why HPT preempts MPT immediately after mutex release

### ✔ Key Takeaways

-   Always protect shared peripherals with mutexes\
-   Expect priority changes when mutexes are used\
-   Task execution order may change due to priority inheritance
