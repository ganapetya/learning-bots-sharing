# 🏃‍♂️ Sprint 1: The Coordinate & Memory Foundation
**Sprint Start Date**: 2026-08-24  
**Target Platform**: Masha (Hiwonder ROSpider Jetson Orin NX) and Local Development PC  
**Syllabus Reference**: Quarter 1, Week 1  

---

## 🎯 Sprint Objectives
By the end of this sprint, you and Grok will establish both the mathematical and C++ memory paradigms required for Masha's 3D navigation and control. You will bridge JVM heap-allocation patterns to C++ RAII smart pointers and write a spatial coordinate frame translation test using modern C++.

---

## 📐 Mathematical Cornerstones (Sprint 1 Summary)

### 1. Coordinate Frames & Vector Bases
A 3D coordinate frame $\{s\}$ consists of:
* An origin point in space.
* Three mutually orthogonal unit vectors (basis vectors): $\{\hat{x}_s, \hat{y}_s, \hat{z}_s\}$.

Any physical point $p \in \mathbb{R}^3$ is written as a linear combination of these basis vectors:
$$p = p_x \hat{x}_s + p_y \hat{y}_s + p_z \hat{z}_s$$

### 2. Degrees of Freedom (DoF) & Constraints
We calculate the mobility of a physical system using the algebraic rule:
$$\text{DoF} = (\text{Number of variables}) - (\text{Number of independent equations})$$

* **Implicit Representation (Global Particle Model)**: Tracks multiple individual points in global space (e.g., points $A, B, C$ on a coin). Rigidity is enforced using distance constraint equations (e.g., $\text{Distance}(A,B) = \text{constant}$). This is how physics engines operate.
* **Explicit Representation (Local Frame Model)**: Welds a single coordinate frame (origin, $x$, $y$) directly to the coin surface. Because the points are defined as static constants relative to this frame, there are no active constraints. We only track the pose of the frame $(x, y, \theta)$, representing exactly $3\text{ DoF}$. This is how robot state estimators (like ROS 2 `/tf`) operate.

### 3. Closed-Loop "Zero Relative Motion" Rule
In linkage systems (like the Four-Bar Linkage):
* A **Link** is defined strictly by relative motion.
* **The Rule**: If two components (e.g., two distinct physical mounting brackets bolted to the table) have **zero relative motion** between them, they mathematically collapse into a single kinematic link (the **Ground/Earth Link**).
* This is why a four-bar linkage has exactly **4 links** (1 Ground + 3 moving bars), and is controlled deterministically with exactly **1 actuator (1 DoF)**.

### 4. Helical Joint & Joint Coupling
A helical (screw) joint allows both rotation ($\theta$) and translation ($z$). However, it has **only 1 DoF** because these two variables are locked together by the physical pitch ($h$) of the thread:
$$z = h \cdot \theta \quad (\text{1 Independent Constraint Equation})$$
$$\text{DoF} = 2 \text{ variables} - 1 \text{ constraint} = 1\text{ DoF}$$

### 5. Kinematic Redundancy (The "Elbow Wiggle")
When a robot's Configuration Space DoF is greater than its Task Space dimensions (e.g., a 4R planar manipulator arm operating in a 3-dimensional 2D workspace $[x, y, \theta]$), the robot is **redundant**. 
* **The Visual**: If you lock the end-effector (palm) completely flat on a table, the remaining degree of freedom allows you to swing or "wiggle" the elbow joints without moving the hand.

---

## 💻 C++ Concurrency & Memory Foundations (JVM to C++ Bridge)

In Java, the JVM garbage collector continuously sweeps the heap in a background thread to reclaim unused objects. In C++, there is no Garbage Collector. We use **RAII (Resource Acquisition Is Initialization)** and **Smart Pointers** to manage memory deterministically.

### 1. Stack Allocation (The Default)
Variables allocated inside a function reside on the stack. When execution leaves the scope `{ ... }`, stack memory is instantly popped and reclaimed.
* **JVM Equivalent**: Primitive types, or local variable pointers.

### 2. Smart Pointers (Deterministic Heap Management)
Heap variables must be wrapped in smart pointers to prevent memory leaks and segmentation faults:

* **`std::unique_ptr<T>` (Single Ownership)**:
  * Equivalent to a Java object that is strictly thread-confined.
  * Cannot be copied. Can only be moved (`std::move`) to transfer exclusive ownership of a dataset (such as an incoming camera frame) from an acquisition thread to a processing thread.
* **`std::shared_ptr<T>` (Shared Ownership)**:
  * Uses a reference counter. Each time it is copied, the ref count increases; when a copy goes out of scope, it decrements.
  * When the ref count hits zero, the heap memory is deleted instantly on the spot. Useful when multiple concurrent threads (e.g., cat detector and odometry logger) must read the exact same sensor frame.

---

## 🛠️ Sprint 1 Coding Challenge: The Coordinate Transformation Playground

### Setup Instructions for `grok-build`
Work with Grok in your local IDE to set up a clean, minimal CMake project in your repository under `sprint-1-2026-08-24/src/`.

#### 1. CMake Project Structure:
```text
learning-bots-sharing/
├── sprint-1-2026-08-24/
│   ├── CMakeLists.txt
│   └── src/
│       ├── main.cpp
│       ├── spatial_transform.hpp
│       └── spatial_transform.cpp
└── sprint-1-progress-report.md
```

#### 2. The Task: Coordinate Translation & Memory Hand-off
Write a clean C++ program that:
1. Defines a 3D Point struct `struct Point3D { double x, y, z; };`.
2. Implements a helper class `SpatialTransformer` that translates a point from Masha's Deptrum depth camera frame $\{c\}$ to her chassis frame $\{b\}$ using a known 3D translation offset vector:
   $$p^b = p^c + t^{bc}$$
   *(Assume the camera is offset from the center of Masha's chassis by $t^{bc} = [0.1, 0.0, 0.15]$ meters).*
3. Instantiates a 3D coordinate point on the heap wrapped in a `std::unique_ptr<Point3D>`.
4. Successfully moves (`std::move`) ownership of this point through a memory pipeline to a processing function that outputs the transformed point in the chassis frame.

---

## 📝 Grok's Progress Report Template
Once Grok helps you compile, run, and test your code, have him fill out this template and commit it to your repository as `sprint-1-progress-report.md`:

```markdown
# 🏁 Sprint 1 Progress Report
**Sprint Date**: 2026-08-24  
**Status**: [In Progress / Completed]  

## 📐 Math Verification Checkpoints
1. In your own words, what is the fundamental difference between representing Masha's body orientation using a local explicit frame vs. implicit coordinate constraints on her physical particles?
   * **Grok/User Answer**: [Write answer here]
2. If we lock Masha's foot tip flat on the ground (locking $x, y, z$ in workspace), but her leg has 3 joint actuators (3 DoF in C-space), can the leg still "wiggle" its joints? Why or why not?
   * **Grok/User Answer**: [Write answer here]

## 💻 C++ Compilation & Test Results
* **Project Location**: `sprint-1-2026-08-24/`
* **Compilation Command**: `cmake .. && make`
* **Output Log**:
```text
[Paste executable stdout here showing the translated coordinates and memory destruction logs]
```

## 🚀 Key Reflections & Next Steps
* [What went well, what compilation hurdles you solved, and what you are ready to tackle next sprint!]
```
