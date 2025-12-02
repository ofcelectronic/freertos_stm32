# README -- FreeRTOS Structured Queue Example (STM32)

This project demonstrates how to use **FreeRTOS queues** to transfer
**complex data types** between multiple tasks on an STM32
microcontroller.\
Instead of sending primitive data types (like integers or chars), the
application uses a **structured queue** to pass a `struct` containing:

-   an integer counter\
-   a 16-bit large value\
-   a string pointer

This example highlights **why and how** structured queues are necessary
when different variable types must be grouped and transferred together.

------------------------------------------------------------------------

## 📌 Why Use a Structured Queue in FreeRTOS?

FreeRTOS queues normally pass data **by copy**, and each queue item must
have a fixed size.

If you want to send **multiple different variable types at once** (e.g.,
integer + string + uint16), you need to group them inside a `struct`.\
This approach:

-   keeps data organized\
-   simplifies queue usage\
-   avoids sending multiple related values separately\
-   enables dynamic, flexible messaging between tasks

**Therefore, when you need to send multiple related fields → a
structured queue is the correct solution.**

------------------------------------------------------------------------

## 📁 Project Structure

The application contains **three FreeRTOS tasks**:

### 1. Sender1 Task

Allocates memory, fills a structure with data, and sends it to the
queue.

### 2. Sender2 Task

Same logic as Sender1, but sends different string messages.

### 3. Receiver Task

Receives the structure from the queue, prints the content via UART, and
frees memory.

------------------------------------------------------------------------

## 🧱 Data Structure Definition

``` c
typedef struct {
    char *str;
    int counter;
    uint16_t large_value;
} my_struct;
```

This `struct` represents the data unit passed through the queue.

------------------------------------------------------------------------

## 📌 Queue Creation

``` c
ST_Queue_Handler = xQueueCreate(2, sizeof(my_struct));
```

-   Queue length: **2 items**
-   Each item: **full structure size**

Since the queue stores structure **copies**, the queue size must match
`sizeof(my_struct)`.

BUT the structure contains a **pointer**, so the actual pointed string
is **not copied**---only the pointer value is passed.

------------------------------------------------------------------------

## 🚀 Task Descriptions

------------------------------------------------------------------------

### Sender1 Task

#### Steps:

1.  Prints a debugging message.
2.  Allocates memory for a `my_struct` instance:
    `c     ptrtostruct = pvPortMalloc(sizeof(my_struct));`
3.  Fills structure fields:
    `c     ptrtostruct->counter = 1 + indx1;     ptrtostruct->large_value = 1000 + indx1*100;     ptrtostruct->str = "HELLO FROM SENDER 1";`
4.  Sends the structure pointer to the queue:
    `c     xQueueSend(ST_Queue_Handler, &ptrtostruct, portMAX_DELAY);`
5.  Delays 2 seconds.

------------------------------------------------------------------------

### Sender2 Task

Same logic as Sender1 but sends a different string:

``` c
ptrtostruct->str = "SENDER 2 says HIIIII";
```

Also increments its own index counter.

------------------------------------------------------------------------

### Receiver Task

#### Steps:

1.  Prints a message that it is ready to receive.
2.  Waits for a structure pointer:
    `c     xQueueReceive(ST_Queue_Handler, &Rptrtostruct, portMAX_DELAY);`
3.  Allocates temporary memory for formatting a text message.
4.  Prints received data over UART:
    `c     sprintf(ptr,         "Received from QUEUE:\n COUNTER = %d\n LARGE VALUE = %u\n STRING = %s\n\n",         Rptrtostruct->counter,         Rptrtostruct->large_value,         Rptrtostruct->str     );`
5.  Frees the allocated memory of:
    -   temporary string buffer
    -   received structure

------------------------------------------------------------------------

## 🔧 Dynamic Memory Handling

The project uses:

-   `pvPortMalloc()` for allocating memory\
-   `vPortFree()` for releasing memory

This is required because queue items contain **pointers**, not full
deep-copied data.

------------------------------------------------------------------------

## 📡 UART Debug Output

Each task prints:

-   Task startup messages\
-   Queue send success messages\
-   Received structure contents

Helps visually verify FreeRTOS behavior.

------------------------------------------------------------------------

## ✔ Summary

This project teaches:

### 1. How to use FreeRTOS queues with structures

### 2. How to send and receive pointers through a queue

### 3. Why structured queues are required for multiple data types

### 4. How to handle memory allocation safely in FreeRTOS tasks

### 5. How to build sender/receiver task architecture

The result is a clean, scalable messaging system ideal for embedded
applications involving grouped data.
