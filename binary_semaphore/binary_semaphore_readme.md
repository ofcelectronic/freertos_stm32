# Binary Semaphore Example (FreeRTOS) — README

## Overview
This project demonstrates how to use a **Binary Semaphore** in FreeRTOS on an STM32 microcontroller. The application contains three tasks with different priorities:

- **HighTask** (High priority)
- **NormalTask** (Normal priority)
- **LowTask** (Low priority)

The purpose of this project is to show how tasks synchronize using a binary semaphore and how task scheduling behaves when using `osSemaphoreWait()` and `osSemaphoreRelease()`.

---

## What is a Binary Semaphore?
A **binary semaphore** is a synchronization object that has only **two states**:

- **0 (unavailable)** — a task must wait
- **1 (available)** — a task may continue

It is commonly used for:

- Task-to-task signaling
- Interrupt-to-task signaling
- Managing access to shared resources (mutex-like behavior)

**Simple explanation:**
> A binary semaphore is like a key that only one task can have at a time. If the key is available (1), a task takes it and continues. If the key is not available (0), the task waits until someone returns the key.

---

## Purpose and Behavior of This Code
This code demonstrates how a **binary semaphore** can prevent a low-priority task from being starved by higher-priority tasks. By holding the semaphore, the low-priority task temporarily blocks even high-priority tasks that also need the semaphore, ensuring it can finish its critical section without interruption.

At the same time, the example also illustrates that **priority inversion can occur**: a low-priority task holding a semaphore can indirectly delay a high-priority task that needs that same semaphore. However, tasks that do *not* require the semaphore continue running according to their normal priority scheduling rules.

## How It Works in This Project
Below is the main semaphore logic used in the HighTask:

```c
char *str1 = "Entered HighTask and waiting for Semaphore\n";
HAL_UART_Transmit(&huart1, (uint8_t *)str1, strlen(str1), 100);

osSemaphoreWait(myBinarySem01Handle, osWaitForever);

char *str3 = "Semaphore acquired by HIGH Task\n";
HAL_UART_Transmit(&huart1, (uint8_t *)str3, strlen(str3), 100);

char *str2 = "Leaving HighTask and releasing Semaphore\n\n";
HAL_UART_Transmit(&huart1, (uint8_t *)str2, strlen(str2), 100);

osSemaphoreRelease(myBinarySem01Handle);
osDelay(500);
```

### Step-by-step behavior
1. **HighTask logs that it has started** and is waiting for the semaphore.
2. `osSemaphoreWait()` is called:
   - If the semaphore is **0**, the task is BLOCKED until another task releases it.
   - If the semaphore is **1**, the task continues immediately.
3. Once the semaphore becomes available, HighTask logs that it has acquired the semaphore.
4. The task finishes its work, logs a message, and **releases the semaphore**.
5. HighTask sleeps for 500 ms, allowing other tasks to run.

---

## Task Scheduling Summary
- **HighTask** has the highest priority; it runs whenever it is READY.
- **NormalTask** runs only when HighTask and LowTask are BLOCKED or DELAYED.
- **LowTask** may block on GPIO input or semaphore waiting.

Binary semaphores ensure that tasks do not run at the same time on critical sections and allow proper coordination.

---

## Requirements
- STM32 MCU (tested with STM32F4 series)
- FreeRTOS
- STM32 HAL Library
- UART configured for debugging output

---

## How to Build
Use STM32CubeIDE:

1. Import the project.
2. Ensure FreeRTOS and Semaphore configurations are enabled in `.ioc` file.
3. Build and flash the firmware.

---

## UART Output Example
```
Entered HighTask and waiting for Semaphore
Semaphore acquired by HIGH Task
Leaving HighTask and releasing Semaphore
```

---

## License
This project is provided for educational purposes and may be freely modified or integrated.

