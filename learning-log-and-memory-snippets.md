Here is the complete, fully detailed **Week 1 Study Plan** in plain text. You can copy and paste this entire block directly to `grok-build` so it has all the mathematical context, C++ memory safety targets, and daily exercises it needs to assist you in your local IDE:

***

```markdown
# 🚀 Week 1 Sprint: The Coordinate & Memory Foundation

This document defines the goals, daily micro-lessons, and coding exercises for Week 1 of our robotics developer program. Our objective is to bridge Peter's senior-level JVM knowledge with safe C++ memory management, while setting up the mathematical foundation for representing Masha's physical limbs in 3D space.

---

## 🎯 Week 1 Learning Targets

*   **The Math**: Grasp 3D coordinate frames, basis vectors, and the physical meaning of SO(3) rotation matrices.
*   **The C++**: Bridge the JVM memory model to C++'s RAII (Resource Acquisition Is Initialization). Master raw references, pointer dereferencing, and the functional differences between std::unique_ptr and std::shared_ptr.
*   **The Exercise**: Set up a simple offline C++ project using CMake on Masha's Jetson Orin NX to test spatial transformations and thread-safe smart pointer hand-offs.

---

## 📅 Daily Micro-Lessons (1 hr / day)

### 🗓️ Day 1: Coordinate Frames and Vector Bases (Math)
*   **Task**: Read "Modern Robotics", Section 2.1 & 2.2.
*   **Concept**: What is a coordinate frame? How do we represent a physical point in Masha's base frame vs. her camera frame?
*   **Math**: A 3D coordinate frame {s} consists of an origin and three orthogonal unit vectors {x_s, y_s, z_s}. A point p in 3D space is written as a linear combination of these basis vectors: p = p_x * x_s + p_y * y_s + p_z * z_s.

### 🗓️ Day 2: The SO(3) Rotation Matrix (Math)
*   **Task**: Read "Modern Robotics", Section 3.1.
*   **Concept**: How do we mathematically rotate one coordinate frame relative to another? The properties of the Special Orthogonal Group SO(3).
*   **Math**: A rotation matrix R_ab in SO(3) maps a vector written in frame {b} to its representation in frame {a} via: p^a = R_ab * p^b.
    The matrix R satisfies two critical properties:
    1. R^T * R = I (It is orthogonal; its inverse is its transpose).
    2. det(R) = +1 (It represents a pure rotation, preserving orientation without reflections).

### 🗓️ Day 3: C++ Memory Allocation vs. JVM Heap (C++ Concurrency)
*   **Task**: Contrast Java's garbage-collected heap allocation with C++'s deterministic stack and heap.
*   **Concept**: Why does C++ not need a garbage collection thread? The RAII design pattern (Resource Acquisition Is Initialization).
*   **Exercise**: Run a quick local C++ script that dynamically allocates an object and watch it immediately destruct when exiting a local scope block `{}`.

### 🗓️ Day 4: Modern Smart Pointers - std::unique_ptr (C++ Concurrency)
*   **Task**: Learn how to pass exclusive ownership of sensor data (like a high-frequency camera frame) between threads without copying memory.
*   **Concept**: `std::unique_ptr` represents *single, exclusive ownership*. It cannot be copied; it can only be moved (`std::move`). 
*   **Java Analogy**: Like a Java object reference that you strictly enforce only one thread is allowed to mutate or reference at a time.

### 🗓️ Day 5: Modern Smart Pointers - std::shared_ptr (C++ Concurrency)
*   **Task**: Learn how to let multiple threads safely read the exact same camera frame or LiDAR scan simultaneously.
*   **Concept**: `std::shared_ptr` uses *reference counting*. When the last `shared_ptr` pointing to an object goes out of scope, the object is immediately deleted on the spot.
*   **Java Analogy**: Close to standard Java garbage collection, but deterministic: deletion happens the *exact instant* the reference count drops to 0.

### 🗓️ Day 6: Setting up the CMake Playground (The Coding Exercise)
*   **Task**: Work with `grok-build` to set up a basic, clean offline C++ CMake workspace on Masha's Jetson (or your laptop).
*   **Exercise**: Write a clean `CMakeLists.txt` that links standard libraries and compiles a small C++ executable. Include `Google Test (gtest)` to prepare for unit-testing our math operations.

### 🗓️ Day 7: Weekend Review & Code Sync
*   **Task**: Combine Days 1-6. Write a small test class in your CMake playground where you instantiate a 3D point inside a custom struct, wrap it in a `std::unique_ptr`, move it to a dummy processing function, and verify its automatic destruction.
*   **Sync**: Push this code to Git and share the link with our Guru (Gemini Notebook) to review the thread safety and memory layout!

---

## 🧠 Memory Snippets & JVM-to-C++ Translation Guides

### 1. Unique Ownership (Move Semantics)
Use this pattern when passing a heavy camera frame from an acquisition thread to a processing thread.

```cpp
#include <memory>
#include <iostream>
#include <vector>

struct CameraFrame {
    int id;
    std::vector<uint8_t> pixel_data;
    ~CameraFrame() { std::cout << "Frame " << id << " memory instantly freed!\n"; }
};

void processFrame(std::unique_ptr<CameraFrame> frame) {
    // Ownership transferred here. When this function exits, memory is instantly freed!
    std::cout << "Processing frame " << frame->id << "\n";
}

int main() {
    auto frame = std::make_unique<CameraFrame>();
    frame->id = 42;
    
    // processFrame(frame); // COMPILE ERROR: std::unique_ptr cannot be copied!
    processFrame(std::move(frame)); // Correct: Ownership moved. "frame" is now null.
}
```

### 2. Shared Ownership (Reference Counting)
Use this pattern when multiple concurrent nodes (like a Cat Classifier and a Leg Balancer) need to read the same frame.

```cpp
#include <memory>
#include <iostream>

struct CameraFrame {
    int id;
    ~CameraFrame() { std::cout << "Shared Frame " << id << " destroyed!\n"; }
};

void inspectFrame(std::shared_ptr<CameraFrame> frame) {
    std::cout << "Inspecting frame " << frame->id << " (Use count: " << frame.use_count() << ")\n";
}

int main() {
    auto shared_frame = std::make_shared<CameraFrame>();
    shared_frame->id = 101;
    
    inspectFrame(shared_frame); // Copies pointer, increments ref count to 2 inside the function.
    // When inspectFrame exits, count decrements back to 1.
    // When main exits, count reaches 0, and memory is instantly freed.
}
```
```

***

Once you copy this over to `grok-build`, let me know how it goes setting up your Day 6 CMake playground, or if you want to walk through the math of coordinate frames to get a head start!
