# FreeRTOS Queue Demonstration – Detailed README

This README explains the overall structure and logic of your FreeRTOS example project. It also breaks down the code into understandable sections and clarifies how FreeRTOS queues behave, especially regarding blocking, priorities, and ISR interactions.

---

# 📌 **Project Overview**
This project demonstrates how three FreeRTOS tasks (two sender tasks and one receiver task) communicate through a queue. It also shows how UART interrupts can attempt to send data into the same queue using an ISR-safe API.

The core purposes of this demo are:
- Understanding task priorities and queue behavior
- Observing how blocking works when a queue becomes full
- Seeing how an ISR attempts to send to a queue
- Learning which task wins when multiple tasks are blocked on the same queue

---

# 🧱 **Project Structure**
The main functional pieces include:

- **Two sender tasks**:
  - A High Priority Task (HPT)
  - A Low Priority Task (LPT)
- **One receiver task** that pulls data from the queue
- **One UART interrupt callback** that tries to insert data at the *front* of the queue
- **A FreeRTOS queue** with space for 5 integers

---

# 🪄 **Queue Creation**
```c
SimpleQueue = xQueueCreate(5, sizeof(int));
```
A queue capable of holding **5 integers** is created. If creation fails, a UART message is printed.

---

# ▶️ **Task Creation**
Three tasks are created with different priorities:

| Task Name          | Function              | Priority |
|--------------------|------------------------|----------|
| Sender High Prio   | `Sender_HPT_Task`      | 3        |
| Sender Low Prio    | `Sender_LPT_Task`      | 2        |
| Receiver Task      | `Receiver_Task`        | 1        |

Higher priority values mean higher priority.

---

# 📤 **Sender Tasks Explanation**

## 🔹 High Priority Sender (HPT)
```c
xQueueSend(SimpleQueue, &i, portMAX_DELAY);
```
- Sends integer values to the queue
- Uses **`portMAX_DELAY`**, meaning:
  - If the queue is full → **task BLOCKS indefinitely**
  - It goes to the *Blocked* state and allows other tasks to run

---

## 🔹 Low Priority Sender (LPT)
```c
xQueueSend(SimpleQueue, &ToSend, portMAX_DELAY);
```
- Behaves the same as the HPT task regarding queue usage
- Lower priority means it will only run when higher tasks are not ready

---

# 📥 **Receiver Task Explanation**
```c
xQueueReceive(SimpleQueue, &received, portMAX_DELAY);
```
- Waits for an item from the queue
- If the queue is empty → **this task BLOCKS indefinitely**
- When data becomes available, it wakes up and prints it

This also means **the receiver controls how fast the queue empties**, which affects all senders.

---

# ⚡ **UART Interrupt (ISR) Queue Send**
Inside the UART receive interrupt callback:
```c
xQueueSendToFrontFromISR(SimpleQueue, &ToSend, &xHigherPriorityTaskWoken)
```
This function attempts to insert data at the **front** of the queue.

### 🔴 IMPORTANT ISR RULES
- An ISR **cannot block**, so it cannot wait for queue space.
- If the queue is full:
  - ISR returns **`pdFAIL`**
  - ISR **does NOT block** (blocking in ISR would freeze the system)

---

# 🧠 **Queue Blocking Behavior (VERY IMPORTANT)**

## 🟦 When the queue is FULL
All tasks trying to send an item using `xQueueSend()` with:
```c
portMAX_DELAY
```
will **block indefinitely**.

Example situation:
- Queue has 5/5 items
- LPT tries to send → BLOCKED
- HPT tries to send → BLOCKED

Now **both sender tasks are blocked**.

Only when the receiver task removes an item will the queue have space again.

---

# 🏆 **Which Task Sends First When Queue Becomes Free?**
If both tasks are blocked waiting to send and the queue gets one empty slot:

### ✔ The **highest priority waiting task** wakes up FIRST.

Example:
- HPT (priority 3) blocked
- LPT (priority 2) blocked
- Queue becomes empty

➡ **HPT wakes up first** and sends its item  
➡ LPT still stays blocked until another space becomes free

This is the FreeRTOS scheduling rule.

---

# ⚡ ISR Queue Send Behavior When Queue is Full
If the queue is full:
- ISR attempts `xQueueSendToFrontFromISR()`
- It returns **`pdFAIL`**
- You correctly print `"QUEUE NULL!"`
- ISR does **not** block, because blocking in ISR is forbidden

---

# 📚 Summary of Queue Mechanics

### ✔ Task sending with `portMAX_DELAY` will BLOCK if:
- Queue is FULL
- It waits forever for space

### ✔ Multiple tasks sending to the same full queue:
- All become BLOCKED
- When queue frees up → **highest priority wins**

### ✔ ISR sending to a full queue:
- Cannot block
- Immediately returns `pdFAIL`

### ✔ Receiver controls the flow:
- Only the receiver makes space
- All senders depend on it

---

# 🎯 Conclusion
This project is an excellent demonstration of:
- FreeRTOS task priorities
- Queue blocking behavior
- How ISRs interact with queues
- Correct use of `xQueueSendFromISR()` and `portYIELD_FROM_ISR()`

The behavior you observe is exactly how FreeRTOS is designed to operate in real-time scheduling environments.

If you want, I can also prepare:
- A flowchart diagram for the queue logic
- A timing diagram showing blocking/unblocking
- A simplified version of the code
- A Turkish version of this README
