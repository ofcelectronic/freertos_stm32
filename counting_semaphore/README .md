# **FreeRTOS Counting Semaphore Example on STM32F4**

This project demonstrates the use of a **Counting Semaphore** in
FreeRTOS on an **STM32F4** microcontroller.\
It is based on the tutorial from Controllerstech:

-   **Website Tutorial:**
    controllerstech.com/freertos-tutorial-4-using-counting-semaphore\
-   **YouTube Video:** "FreeRTOS Tutorial 4 -- Using Counting Semaphore"

The example uses **four tasks** with different priorities and shows how
a UART interrupt can release semaphore tokens that allow tasks to access
shared resources.

------------------------------------------------------------------------

## 🚀 **Project Overview**

The code implements:

-   A **Counting Semaphore** with max count **3**
-   Four tasks:
    -   **HPT** -- High Priority Task (priority 3)
    -   **MPT** -- Medium Priority Task (priority 2)
    -   **LPT** -- Low Priority Task (priority 1)
    -   **VLPT** -- Very Low Priority Task (priority 0)
-   Each task tries to **take a semaphore token**, access a shared
    resource array, and prints information over UART.
-   UART interrupt (`HAL_UART_RxCpltCallback`) listens for character
    `'r'`
    -   When `'r'` is received, **3 semaphore tokens** are released from
        ISR using `xSemaphoreGiveFromISR()`.
    -   A context switch is triggered using `portEND_SWITCHING_ISR()`.

This example shows how FreeRTOS handles **task synchronization**,
**priority scheduling**, and **interrupt-safe API usage**.

------------------------------------------------------------------------

## 📌 **Features**

### ✔️ Counting Semaphore

-   Created with:

    ``` c
    CountingSem = xSemaphoreCreateCounting(3, 0);
    ```

-   Maximum tokens = **3**

-   Initial tokens = **0**\

-   HPT task gives 3 initial tokens at startup:

    ``` c
    xSemaphoreGive(CountingSem);
    xSemaphoreGive(CountingSem);
    xSemaphoreGive(CountingSem);
    ```

### ✔️ UART Interrupt Token Release

When character `'r'` is received:

``` c
xSemaphoreGiveFromISR(CountingSem, &xHigherPriorityTaskWoken);
...
portEND_SWITCHING_ISR(xHigherPriorityTaskWoken);
```

This allows **an ISR to wake a higher-priority task** immediately.

### ✔️ Shared Resource Access

Tasks access:

``` c
int resource[3] = {111, 222, 333};
```

Only if a semaphore token is available.

### ✔️ Task Priorities

  Task   Function      Priority
  ------ ------------- -------------
  HPT    `HPT_TASK`    3 (highest)
  MPT    `MPT_TASK`    2
  LPT    `LPT_TASK`    1
  VLPT   `VLPT_TASK`   0 (lowest)

------------------------------------------------------------------------

## 🧵 **Task Workflow**

Each task follows the same pattern:

1.  Print "Entered TASK"

2.  Call:

    ``` c
    xSemaphoreTake(CountingSem, portMAX_DELAY);
    ```

3.  Access a resouce entry:

    ``` c
    resource[indx]
    ```

4.  Print accessed value

5.  **Does NOT release the semaphore**

6.  Wait using:

    ``` c
    vTaskDelay(x);
    ```

------------------------------------------------------------------------

## 🔄 **UART Interrupt Behavior**

When receiving `'r'`, the ISR will:

1.  Release **three semaphore tokens**
2.  Possibly wake a blocked task
3.  Trigger a context switch if necessary

### Key Code:

``` c
if (rx_data == 'r') {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(CountingSem, &xHigherPriorityTaskWoken);
    xSemaphoreGiveFromISR(CountingSem, &xHigherPriorityTaskWoken);
    xSemaphoreGiveFromISR(CountingSem, &xHigherPriorityTaskWoken);

    portEND_SWITCHING_ISR(xHigherPriorityTaskWoken);
}
```

------------------------------------------------------------------------

## 🛠️ **Dependencies**

### Firmware / HAL

-   STM32Cube HAL (UART / GPIO / Clock)

### FreeRTOS Includes

``` c
#include "FreeRTOS.h"
#include "task.h"
#include "timers.h"
#include "queue.h"
#include "semphr.h"
#include "event_groups.h"
```

### Standard C Includes

``` c
#include "stdlib.h"
#include "string.h"
```

------------------------------------------------------------------------

## 📡 **UART Configuration**

-   **Baud Rate:** 115200\

-   **Data Bits:** 8\

-   **Parity:** None\

-   **Stop Bits:** 1\

-   **Mode:** TX/RX\

-   **Interrupt Reception Enabled:**

    ``` c
    HAL_UART_Receive_IT(&huart1, &rx_data, 1);
    ```

------------------------------------------------------------------------

## 📋 **How to Use**

1.  Flash the firmware onto an STM32F4 board.
2.  Open a serial terminal at **115200 baud**.
3.  Observe task logs.
4.  Press **`r`** on the keyboard → ISR releases 3 semaphore tokens.
5.  Tasks waiting for tokens will wake up according to priority rules.

------------------------------------------------------------------------

## 🏁 **Expected Behavior**

-   HPT wakes first because it has the **highest priority**.
-   After HPT consumes a token, MPT wakes next, then LPT, then VLPT.
-   If no semaphore tokens are available, tasks **block indefinitely**.
-   Pressing `'r'` from UART will instantly unlock 3 blocked tasks.

------------------------------------------------------------------------

## 📁 **Files Included**

  File                    Description
  ----------------------- -----------------------------------------------
  `main.c`                Application code, tasks, semaphores, UART ISR
  `main.h`                MCU-specific includes and definitions
  FreeRTOS source files   RTOS kernel
  HAL files               STM32 HAL drivers

------------------------------------------------------------------------

## 📘 **References**

-   Controllerstech FreeRTOS Tutorial No. 4\
-   FreeRTOS API Documentation\
-   STM32 HAL Documentation
