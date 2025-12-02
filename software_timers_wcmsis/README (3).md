# README

## Overview

This project demonstrates the use of **FreeRTOS tasks**, **periodic
timers**, **one-shot timers**, UART communication, and button/LED
control on the **STM32F429I-DISC1** board.

Two tasks and two software timers are used:

-   **UART Task:** Sends periodic UART messages\
-   **LED Task:** Reads the user button and controls LED behavior\
-   **Periodic Timer:** Repeats an action at fixed time intervals\
-   **One-Shot Timer:** Executes an action once after a delay

------------------------------------------------------------------------

# FreeRTOS Timers Explained Clearly

FreeRTOS provides two important types of software timers that your code
uses:

-   **Periodic Timer (`osTimerPeriodic`)**\
-   **One-Shot Timer (`osTimerOnce`)**

Below is a very clear explanation of **what the duration parameter
actually means** for each timer type.

------------------------------------------------------------------------

# ✔ Periodic Timer (osTimerPeriodic)

A **periodic timer** repeats its task forever at a fixed interval.

### 🔁 What does the duration mean?

➡ **The duration is the interval between each execution.**\
➡ The callback function runs **every time the interval expires**.\
➡ After the callback runs, the timer **automatically reloads** and
starts counting again.

### Example from your code:

``` c
osTimerStart(periodictimerHandle, 1000); // 1000 ms interval
```

### This means:

-   The timer waits **1000 ms**
-   Executes the callback
-   Waits **another 1000 ms**
-   Executes the callback again
-   Continues **forever** until stopped

### In your application:

The callback prints this message:

    "Sending from PERIODIC TIMER"

And it does this **EVERY 1 second**, continuously.

### ✔ Summary:

  -----------------------------------------------------------------------
  Timer Type         Duration Means            How Many Times?
  ------------------ ------------------------- --------------------------
  **Periodic Timer** Interval between          Infinite loop (repeats
                     executions                automatically)

  -----------------------------------------------------------------------

------------------------------------------------------------------------

# ✔ One-Shot Timer (osTimerOnce)

A **one-shot timer** executes the callback function only **once**.

### ▶ What does the duration mean?

➡ **The duration is the total delay before the callback runs.**\
➡ The callback runs **only one time**, after the specified duration has
passed.\
➡ The timer **does not restart itself**.\
➡ If you want it to run again, you must **manually restart** it.

### Example from your code:

``` c
osTimerStart(onceTimerHandle, 4000); // 4000 ms delay
```

### This means:

-   The timer waits **4000 ms**
-   The callback runs **once**
-   The timer stops

### In your project:

-   When the button is pressed, LED is turned ON immediately
-   Then the one-shot timer starts (4 seconds)
-   After **4 seconds**, the timer callback turns the LED OFF
-   It **does NOT run again** unless the button is pressed again

### ✔ Summary:

  -----------------------------------------------------------------------
  Timer Type         Duration Means            How Many Times?
  ------------------ ------------------------- --------------------------
  **One-Shot Timer** Delay until the one-time  Only once
                     action occurs             

  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Direct Comparison (Very Clear Table)

  ------------------------------------------------------------------------
  Feature        Periodic Timer               One-Shot Timer
  -------------- ---------------------------- ----------------------------
  Type           Repeating                    Single execution

  Duration       **Interval between each      **Wait time before executing
  meaning        execution**                  once**

  Auto-restart   ✔ Yes (always)               ✖ No (must be manually
                                              restarted)

  Used for       Regular repeating tasks      Delayed single operations

  In your        Sends UART message every 1s  Turns LED OFF after 4s
  project                                     
  ------------------------------------------------------------------------

------------------------------------------------------------------------

# How These Timers Behave in Your Project

### 🟦 Periodic timer (1 second interval)

Runs this over and over:

    Sending from PERIODIC TIMER
    (1 second passes)
    Sending from PERIODIC TIMER
    (1 second passes)
    ...

### 🟩 One-shot timer (4 second delay)

Behavior when button is pressed: 1. LED turns ON immediately\
2. Timer starts counting 4 seconds\
3. When 4 seconds are over → LED turns OFF\
4. Timer stops completely

------------------------------------------------------------------------

# Tasks and Timers Running Together

Even though the STM32 is a single-core processor, FreeRTOS schedules:

-   UART task every 2 seconds\
-   LED task every 20 ms\
-   Periodic timer callback every 1 second\
-   One-shot timer callback after 4 seconds

This gives the appearance of simultaneous execution, thanks to context
switching.
