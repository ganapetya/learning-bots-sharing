# 📓 Learning Log & Memory Snippets
*A living notebook mapping multivariable math, safe C++ concurrency, and robotics theory to physical hexapod controls.*

---

## 📈 My Progress Tracker

- [ ] **Quarter 1: Foundations of Spatial Mechanics & Edge CV**
  - [ ] **Week 1: The Coordinate & Memory Foundation** ── 🎯 *CURRENT SPRINT*
  - [ ] Week 2: Rotations in 3D Space ($SO(3)$ and Quaternions)
  - [ ] Week 3: Camera Extrinsics & Camera Matrix Projection
  - [ ] Week 4: Multi-Threaded Image Acquisition & Ring Buffers
  - [ ] Week 5: OpenCV Image Segmentation & Color Space Invariance
  - [ ] Week 6: Deep Learning Edge Deployment (ONNX Runtime on Jetson)
  - [ ] Week 7: Cat Detection & Multi-threaded Inter-Process Communication
  - [ ] Week 8: Behavioral Trees for Multi-agent Tracking (Masha meets Saveli)
  - [ ] Week 9: Phase 1 Capstone: Autonomous Cat Spotting & Tracking

---

## 🎯 Week 1 Sprint: The Coordinate & Memory Foundation

### 1. Goals for this Week
*   **The Math**: Grasp 3D coordinate frames, basis vectors, and the physical meaning of $SO(3)$ rotation matrices.
*   **The C++**: Bridge your Java JVM memory model to C++'s RAII (Resource Acquisition Is Initialization). Master raw references, pointer dereferencing, and the difference between `std::unique_ptr` and `std::shared_ptr`.
*   **The Exercise**: Setup a simple offline C++ project using CMake to test spatial transformations and smart pointer hand-offs.

---

### 📅 Daily Micro-Lessons (1 hr/day)

#### 🗓️ Day 1: Coordinate Frames and Vector Bases (Math)
*   **Task**: Read *Book 2 (Modern Robotics)*, Section 2.1 & 2.2.
*   **Concept**: What is a coordinate frame? How do we represent a physical point in Masha's base frame vs. her camera frame?
*   **Math Snippet**: A 3D coordinate frame $\\{s\\}$ consists of an origin and three orthogonal unit vectors $\\{\\hat{x}_s, \\hat{y}_s, \\hat{z}_s\\}$. A point $p \\in \\mathbb{R}^3$ is written as a linear combination of these basis vectors: $p = p_x \\hat{x}_s + p_y \\hat{y}_s + p_z \\hat{z}_s$.\

#### 🗓️ Day 2: The $SO(3)$ Rotation Matrix (Math)
*   **Task**: Read *Book 2*, Section 3.1.
*   **Concept**: How do we mathematically rotate one coordinate frame relative to another? The properties of Special Orthogonal Group $SO(3)$.
*   **Math Snippet**: A rotation matrix $R_{ab} \\in SO(3)$ maps a vector written in frame $\\{b\\}$ to its representation in frame $\\{a\\}$ via $p^a = R_{ab} p^b$. $R$ satisfies:\
    1. $R^T R = I$ (Orthogonal)\
    2. $\det(R) = +1$ (No reflections)

#### 🗓️ Day 3: C++ Memory Allocation vs. JVM Heap (C++ Concurrency)
*   **Task**: Contrast Java's garbage-collected heap allocation with C++'s stack and heap.
*   **Concept**: Why does C++ not need a garbage collection thread? The RAII design pattern.
*   **Exercise**: Run a quick local C++ script that dynamically allocates an object and watch it immediately destruct when exiting scope `{}`.

#### 🗓️ Day 4: Modern Smart Pointers - `std::unique_ptr` (C++ Concurrency)
*   **Task**: Learn how to pass exclusive ownership of sensor data (like a high-frequency camera frame) between nodes.
*   **Concept**: `std::unique_ptr` represents *single ownership*. It cannot be copied, only moved (`std::move`). \
    *   **Java Bridge**: Like a Java object that you strictly enforce only one thread is allowed to mutate or reference at a time.

#### 🗓️ Day 5: Modern Smart Pointers - `std::shared_ptr` (C++ Concurrency)
*   **Task**: Learn how to let multiple threads safely read the same camera frame or LiDAR scan simultaneously.
*   **Concept**: `std::shared_ptr` uses *reference counting*. When the last `shared_ptr` pointing to an object goes out of scope, the object is immediately deleted on the spot.

#### 🗓️ Day 6: Setting up the CMake Playground (The Exercise)
*   **Task**: Work with `grok-build` to set up a basic, clean C++ CMake workspace on your laptop or Masha. \
    *   **Exercise**: Write a `CMakeLists.txt` that links standard libraries and compiles a small C++ executable. Include `Google Test (gtest)` to prepare for testing math operations.

#### 🗓️ Day 7: Weekend Review and Sync\
*   **Task**: Combine Days 1-6. Write a small test class in your CMake playground where you instantiate a 3D point inside a custom struct, wrap it in a `std::unique_ptr`, move it to a dummy processing function, and verify its destruction.\
*   **Sync**: Push this code to Git and send me the link. I will review your structure and explain the core mechanics of memory hand-offs!

---

## 🧠 Memory Snippets & JVM-to-C++ Translation Guides

### 1. RAII & Smart Pointer Patterns
In Java, you can pass object references freely between threads, letting the Garbage Collector worry about cleanup. Doing this in C++ with raw pointers (`*`) will trigger segmentation faults or silent memory leaks. \

Here is how you handle memory ownership:

```cpp
// 1. UNIQUE OWNERSHIP (Move semantics)
// Use when passing a heavy sensor frame from a "capture" thread to a "processing" thread.
#include <memory>
#include <iostream>

struct CameraFrame {
    int id;
    std::vector<uint8_t> pixel_data;
    ~CameraFrame() { std::cout << "Frame " << id << " memory freed!\n"; }
};

void processFrame(std::unique_ptr<CameraFrame> frame) {
    // Ownership transferred here. When this function exits, memory is instantly freed.
    std::cout << "Processing frame " << frame->id << "\n";
}

int main() {
    auto frame = std::make_unique<CameraFrame>();
    frame->id = 42;
    
    // processFrame(frame); // COMPILE ERROR: Cannot copy a unique_ptr!
    processFrame(std::move(frame)); // Correct: Ownership moved. frame is now null.
}
```

```cpp
// 2. SHARED OWNERSHIP (Reference Counting)
// Use when multiple concurrent nodes (e.g., Cat Classifier, Object Localizer) need to read the same frame.
void inspectFrame(std::shared_ptr<CameraFrame> frame) {
    std::cout << "Inspecting frame " << frame->id << " (Use count: " << frame.use_count() << ")\n";
}

int main() {
    auto shared_frame = std::make_shared<CameraFrame>();
    shared_frame->id = 101;
    
    inspectFrame(shared_frame); // Copies pointer, increments ref count.
    // When inspectFrame exits, count decrements.
    // When main exits, count reaches 0, memory is instantly freed.
}
```
