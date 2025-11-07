# Projects  

The project assignment for 2026 focuses on the development of new scheduling algorithms for FreeRTOS


# 🕒 Precise Scheduler for FreeRTOS

> A deterministic, timeline-driven scheduler replacing the default FreeRTOS priority-based model.

---

## 📘 Table of Contents

1. [Overview](#overview)  
2. [Key Features](#key-features)  
   - [Major Frame Structure](#1-major-frame-structure)  
   - [Task Model](#2-task-model)  
   - [Task Categories](#3-task-categories)  
   - [Periodic Repetition](#4-periodic-repetition)  
   - [Configuration Interface](#5-configuration-interface)  
   - [Trace and Monitoring System](#6-trace-and-monitoring-system)  
   - [Production and Regression Testing](#7-production-and-regression-testing)  
   - [Target Platform](#8-target-platform)  
3. [Learning Objectives](#learning-objectives)  
4. [Deliverables](#deliverables)

---

## 🧭 Overview

This project aims to **develop a precise, timeline-based scheduler** that **replaces the standard priority-based FreeRTOS scheduler**.  
The scheduler enforces **deterministic task execution** based on a **major frame and sub-frame structure**, following the principles of **time-triggered architectures**.

Each task runs at **predefined times** in a global schedule, ensuring predictable real-time behavior and repeatability — without relying on dynamic priority changes.

---

## ⚙️ Key Features

### **1. Major Frame Structure**
- The scheduler operates within a **major frame**, with a duration **defined at compile time** (e.g., 100 ms, 1 s).
- Each major frame is divided into **sub-frames**, smaller time windows that host groups of tasks.
- Each sub-frame can **contain multiple tasks** with assigned timing and order.

**Example**
```
Major frame = 100 ms
Sub-frames = 10 × 10 ms
→ The scheduler repeats this 100 ms timeline cyclically.
```

---

### **2. Task Model**
- Each task is implemented as a **single function** that executes from start to end and then terminates.
- Tasks do **not** include periodicity or self-rescheduling logic — the scheduler controls activation timing.

---

### **3. Task Categories**

#### 🧱 Hard Real-Time (HRT) Tasks
- Assigned to a specific sub-frame.  
- The scheduler knows the **start** and **end times** of each task within the sub-frame.  
- A task is **spawned at the beginning** of its slot and either:
  - runs until completion, or  
  - is **terminated** if it exceeds its deadline.  
- **Non-preemptive:** once started, it cannot be interrupted.

**Example**
```
Task_A → Sub-frame 2 (20–30 ms)
Start = 21 ms, End = 27 ms
→ Finishes at 26 ms → OK
→ Still running at 27 ms → Terminated
```

#### 🌿 Soft Real-Time (SRT) Tasks
- Executed **during idle time** left by HRT tasks.  
- Scheduled in a **fixed compile-time order** (e.g., Task_X → Task_Y → Task_Z).  
- **Preemptible** by any hard real-time task.  
- **No guarantee** of completion within the frame.

**Example**
```
Sub-frame 5 (40–50 ms)
Task_B (HRT) uses 4 ms → Remaining 6 ms used by SRT tasks (Task_X → Task_Y → Task_Z)
```

---

### **4. Periodic Repetition**
- At the end of each major frame:
  - All tasks are **reset and reinitialized**.
  - The scheduler **replays the same timeline**, guaranteeing deterministic repetition across frames.

---

### **5. Configuration Interface**

All scheduling parameters (start/end time, sub-frame, order, category, etc.) are defined in a **dedicated OS data structure**.

**Example**
```c
typedef struct {
    const char* task_name;
    TaskFunction_t function;
    TaskType_t type; // HARD_RT or SOFT_RT
    uint32_t start_time;
    uint32_t end_time;
    uint32_t subframe_id;
} TimelineTaskConfig_t;
```

A system call (e.g. `vConfigureScheduler(TimelineConfig_t *cfg)`) will:
- Parse this configuration,
- Create required FreeRTOS data structures,
- Initialize the timeline-based scheduling environment.

---

### **6. Trace and Monitoring System**
A **trace module** must log and visualize scheduler behavior with **tick-level precision**.

#### Logged information:
- Task start and end ticks  
- Deadline misses or forced terminations  
- CPU idle time  

**Example Output**
```
[ 21 ms ] Task_A start
[ 26 ms ] Task_A end
[ 40 ms ] Task_B start
[ 47 ms ] Task_B deadline miss → terminated
```

---

### **7. Production and Regression Testing**
Develop an **automated test suite** to validate scheduler correctness and robustness.

#### The suite must include:
- Stress tests (e.g., overlapping HRT tasks).  
- Edge-case tests (e.g., minimal time gaps).  
- Preemption and timing consistency checks.  

Each test must:
- Produce **human-readable summaries**, and  
- Include **automatic pass/fail checks** for regression testing.

**Example**
```
Test 3 – Overlapping HRT Tasks: FAILED (Task_A killed at 32 ms)
Test 4 – SRT Preemption: PASSED
```

---

### **8. Target Platform**
- The scheduler runs under **QEMU** emulating **ARM Cortex-M** (M3/M4/M7).  
- It must **not depend on board-specific features** or hardware registers.  
- The code should remain **portable and maintainable** across architectures.

---

## 🎯 Learning Objectives
- Understand and implement **timeline-based real-time scheduling**.  
- Integrate **custom scheduling logic** into FreeRTOS.  
- Apply **tick-level tracing** for timing verification.  
- Design and automate **production-grade regression tests**.  
- Gain hands-on experience with **embedded system simulation** in QEMU.

---

## 📦 Deliverables
1. ✅ Modified FreeRTOS kernel with timeline-based scheduler.  
2. ✅ Configuration data structure and system call for schedule definition.  
3. ✅ Trace and monitoring system with tick-level resolution.  
4. ✅ Automated test suite for validation and regression checking.  
5. ✅ Documentation and example configurations.

---

## 🧰 Example Repository Structure
```
├── src/
│   ├── timeline_scheduler.c
│   ├── timeline_scheduler.h
│   ├── trace.c
│   ├── trace.h
│   └── tests/
│       ├── test_overlap.c
│       ├── test_preemption.c
│       └── test_results.log
├── include/
│   └── timeline_config.h
├── qemu/
│   └── run_qemu.sh
├── README.md
└── LICENSE
```

---

## 🧪 Example Test Run
```
$ make run-tests
[INFO] Starting timeline scheduler tests...
[OK] Test 1 – HRT task timing
[OK] Test 2 – SRT preemption
[FAIL] Test 3 – Overlapping HRT tasks (Task_A killed at 32 ms)
[INFO] 2 passed, 1 failed.
```

---

> **Note:**  
> The development and testing environment must rely entirely on **QEMU**, ensuring reproducibility and hardware independence.  
> Code should be **portable** across multiple ARM Cortex-M cores.

---

**Author:** *[Your Name]*  
**Institution:** *Politecnico di Torino*  
**Course/Module:** *Real-Time Embedded Systems Laboratory*  
**License:** MIT
