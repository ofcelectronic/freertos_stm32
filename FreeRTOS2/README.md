# 🧠 STM32 FreeRTOS Example — Multi-Tasking with SWV Console Output

## 📘 Overview
This project demonstrates a **basic FreeRTOS setup on an STM32 microcontroller**, showing how multiple tasks can run concurrently while printing debug messages via the **SWV (Serial Wire Viewer)** console — without using UART.

It is based on the tutorial from [Controllerstech – FreeRTOS Tutorial 2.0](https://controllerstech.com/free-rtos-tutorial-2-0-with-stm32/), adapted and extended with real task management operations and SWV/ITM output.

---

## ⚙️ Hardware
- **Board:** STM32F429I-DISC1  
- **MCU:** STM32F429ZI  
- **Clock:** 180 MHz (HSE + PLL, OverDrive mode enabled)
- **Peripherals Used:**  
  - GPIO (basic init)  
  - SWD (for SWV/ITM data console)  
  - SysTick (used as RTOS timebase)

---

## 💻 Software
- **IDE:** STM32CubeIDE  
- **RTOS:** FreeRTOS (CMSIS-OS wrapper)  
- **Debugger:** ST-LINK (onboard debugger with SWO pin)  
- **Language:** C (C99 standard)  

---

## 🧩 Project Description

The firmware creates **two FreeRTOS threads**:

| Task Name | Priority | Stack | Description |
|------------|-----------|--------|--------------|
| `defaultTask` | Normal | 128 words | Prints a simple message every 1s |
| `Task2` | Above Normal | 128 words | Prints counter value every 2s and demonstrates RTOS APIs like suspend, resume, and delayUntil |

> ⚙️ **Important Configuration Note:**  
> In order to use `osDelayUntil()`, you must enable it in the **.ioc** configuration file:  
> - Go to **FreeRTOS → Configuration → Include Parameters**  
> - Enable the option **`vTaskDelayUntil`**  
> - This will allow the use of `osDelayUntil()` function in your tasks.  

---

## 🧵 Task Functionality

**`defaultTask()`**  
Runs continuously and prints `"DefaultTask..."` every 1 second using `osDelay(1000)`.

**`task2_init()`**  
Runs concurrently and:
- Prints `Task2 = X` every 2 seconds.
- Demonstrates:
  - **`osThreadSuspend()` / `osThreadResume()`** — to pause and resume `defaultTask`
  - **`osThreadTerminate()`** — to completely kill a task  
  - **`osDelayUntil()`** — to implement precise periodic delays independent of system load

---

## 🔍 SWV (Serial Wire Viewer) Console Output

Instead of using UART, this project sends `printf()` data to the **SWV Data Console** through the **ITM (Instrumentation Trace Macrocell)** interface.

The `_write()` function is redirected to send characters via ITM:

```c
int _write(int file, char *ptr, int len)
{
    for (int i = 0; i < len; i++)
        ITM_SendChar(*ptr++);
    return len;
}
```

### ✅ To enable SWV in STM32CubeIDE:
1. Connect your board with **ST-LINK** (ensure SWO pin is available and connected).  
2. Go to:  
   **Run → Debug Configurations → Debugger → SWV tab**
3. Enable *Serial Wire Viewer* and set **Core Clock = 180 MHz** (match your project clock).  
4. Open **SWV Data Console** from the *Debug* perspective.  
5. Click **Start Trace** → Now you’ll see `printf()` outputs live.  

---

## ⏱️ Delay Functions: `osDelay()` vs `HAL_Delay()`

| Function | Source | Description | Usage Context |
|-----------|---------|--------------|----------------|
| `HAL_Delay(ms)` | HAL Library | Busy-waits inside SysTick interrupt | **Blocks** the CPU — not recommended in RTOS tasks |
| `osDelay(ms)` | FreeRTOS API | Puts the **current task to sleep** for `ms` ticks | Allows other tasks to run concurrently |
| `osDelayUntil(&t, ms)` | FreeRTOS API | Delays a task until a specific tick count | For deterministic, periodic operations |

> ⚠️ Always use `osDelay()` or `osDelayUntil()` inside FreeRTOS tasks — never `HAL_Delay()`, as it blocks the scheduler.

---

## 🏗️ Build & Run

1. Open the project in **STM32CubeIDE**  
2. Check that **FreeRTOS** is enabled under **Middleware → FreeRTOS**  
3. Build (`Ctrl + B`)  
4. Launch **Debug** (`F11`)  
5. Open **SWV Console** and click *Start Trace*  
6. Observe live task messages like:

```
starting ...
DefaultTask...
Task2 = 0
Task2 = 1
Task2 = 2
DefaultTask...
Task2 = 3
```

---

## 📁 Directory Overview
```
Core/
 ├── Src/
 │    ├── main.c        # Main program entry
 │    ├── freertos.c    # RTOS configuration and hooks
 │    └── ... 
 ├── Inc/
 │    ├── main.h
 │    ├── freertos.h
 │    └── ...
```

---

## 🚀 Future Improvements
- Add queue and semaphore examples  
- Integrate hardware peripherals (ADC, PWM, CAN) with RTOS tasks  
- Implement task notifications for inter-task signaling  

---

## 🧾 License
This project is based on **STMicroelectronics HAL & FreeRTOS** examples.  
Distributed under the terms in the included `LICENSE` file.  

---

## 👤 Author
**Genç Mühendis**  
Embedded Systems Developer — STM32 & FreeRTOS Enthusiast  
