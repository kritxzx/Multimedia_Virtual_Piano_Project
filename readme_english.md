# 🎹 Air Piano — 5-Finger Hand-Tracking Virtual Piano

A real-time, browser-based **Air Piano** web application powered by Computer Vision and Web Audio. Using only a standard webcam, users can play piano in mid-air with all 5 fingers on both hands—no external hardware or sensors required.

---

## ✨ Key Features

* **Full 5-Finger, 2-Hand Tracking:** Tracks all fingertips (Thumb, Index, Middle, Ring, Pinky) across both hands simultaneously, enabling polyphonic chords.
* **Dual Detection Modes:**
  * **Trigger Line Mode:** Hover above the dashed line to aim and dip your finger past the line to strike a note (high accuracy, minimal accidental triggers).
  * **Tap Velocity Mode:** Detects rapid downward finger flicks to trigger notes (natural piano-like feel).
* **Adjustable Note Sustain (Short / Long):** Instantly toggle between **Long (Resonant)** and **Short (Staccato)** note decay.
* **Studio-Quality Audio with Fallback:** Uses real **Salamander Grand Piano** audio samples enhanced with Reverb, paired with a zero-wait fallback `PolySynth` while samples load.
* **Rich Visual Feedback:** Mirrored webcam feed, translucent hand skeleton overlay, color-coded fingertip indicators, animated key depression, and ripple effects upon note strikes.

---

## 🛠️ Tech Stack

| Layer | Technology / Library | Role & Justification |
| :--- | :--- | :--- |
| **UI & Styling** | **HTML5 / CSS3** | Lightweight glassmorphic control bar with responsive canvas layout. |
| **Real-Time Graphics** | **HTML5 Canvas API** | Renders the mirrored video feed, translucent piano keys, hand skeleton, and visual effects at 60 FPS. |
| **Computer Vision (AI)** | **MediaPipe Hand Landmarker** (`@mediapipe/tasks-vision`) | Google's on-device ML model running via WebGL/GPU delegate, outputting 21 3D hand landmarks per hand in real time. |
| **Audio Engine** | **Tone.js** (Web Audio API wrapper) | Handles polyphony, low-latency note scheduling, envelope release control, and real-time Reverb. |
| **Sound Samples** | **Salamander Grand Piano** | Open-source acoustic grand piano recordings; `Tone.Sampler` pitch-shifts a small subset of samples across the octave. |
| **Core Logic** | **Vanilla JavaScript (ES Modules)** | Zero build step required; single-file architecture for clean readability and easy maintenance. |

---

## 🚀 Getting Started (How to Run)

> **Note:** Modern browsers restrict webcam access (`getUserMedia`) to secure contexts (`localhost` or `HTTPS`). Opening the `.html` file directly via `file://` will not work.

1. Save the prototype code as `index.html`.
2. Open the project folder in **VS Code**.
3. Install the **Live Server** extension, right-click `index.html`, and select **"Open with Live Server"**.
4. Click **"เปิดกล้อง & เริ่มเล่น" (Start Camera)** and grant webcam permission when prompted.

---

## 📂 Code Architecture

The codebase is cleanly organized into **6 modular sections**:

1. **CONFIG & DATA:** Piano note definitions ($C4 - C5$), fingertip landmark indices (`[4, 8, 12, 16, 20]`), bone connections, and sensitivity thresholds.
2. **AUDIO ENGINE:** Initializes `Tone.Sampler`, fallback `PolySynth`, and `Tone.Reverb`, handling Short vs. Long note playback.
3. **KEY GEOMETRY:** Dynamically computes white and black key bounding boxes and performs hit-testing (prioritizing black keys over white keys).
4. **HAND TRACKING & TRIGGER:** Mirrors coordinates, applies EMA smoothing, and executes Trigger Line (with Hysteresis) or Tap Velocity logic.
5. **CANVAS RENDERING & FX:** Draws the mirrored video, 3D-styled keys, glowing trigger line, hand skeleton, fingertip dots, and ripple animations.
6. **INIT & MAIN LOOP:** Binds UI toggle buttons, initializes MediaPipe and webcam stream, and runs the `requestAnimationFrame` loop.

---

## 📝 Project Report & Technical Documentation Guide

Use the sections below as a foundation for writing an academic paper, capstone report, or technical presentation.

### 1. Problem Statement & Motivation
Building a playable virtual instrument using a monocular 2D webcam introduces three core Human-Computer Interaction (HCI) challenges:
1. **Lack of Tactile Feedback:** Without physical keys, users cannot feel key resistance, making it difficult to separate hovering/aiming from intentional key presses.
2. **Landmark Jitter:** Raw ML landmark predictions fluctuate slightly even when a hand is stationary, causing unintended rapid note retriggers.
3. **Unreliable Depth Estimation:** Standard 2D webcams lack dedicated depth sensors (like LiDAR), making raw $Z$-axis depth values noisy for press detection.

### 2. System Pipeline
Each video frame (30–60 FPS) passes through a deterministic 5-stage pipeline:

$$\text{Webcam Frame} \rightarrow \text{MediaPipe AI (21 Landmarks)} \rightarrow \text{Mirror \& EMA Filter} \rightarrow \text{Trigger Logic} \rightarrow \text{Tone.js Audio + Canvas UI}$$

### 3. Core Algorithms & Signal Processing

* **Coordinate Mirroring:**
  To match natural mirror intuition, normalized horizontal coordinates ($x \in [0, 1]$) are inverted:
  $$x_{\text{mirrored}} = 1 - x_{\text{raw}}$$

* **Jitter Reduction via Exponential Moving Average (EMA):**
  Raw fingertip coordinates are smoothed to balance stability and responsiveness:
  $$P_t = P_{t-1} + \alpha \cdot (P_{\text{raw}} - P_{t-1})$$
  *(where $\alpha = 0.45$ is `CFG.smooth`; lower values increase smoothness, higher values reduce latency).*

* **Anti-Bounce Hysteresis (Trigger Line Mode):**
  Once a fingertip crosses below the trigger line $Y_{\text{trigger}}$, its state locks to `pressed = true` and fires one note. It will not re-arm until the finger lifts above a deadzone threshold:
  $$y < Y_{\text{trigger}} - \Delta_{\text{deadzone}}$$

* **Velocity-Based Tap Detection (Tap Mode):**
  Computes frame-to-frame downward vertical velocity $v_y = y_t - y_{t-1}$. A note triggers only when $v_y > V_{\text{threshold}}$ and a minimum cooldown period ($140\text{ ms}$) has elapsed since the previous tap.

### 4. Limitations & Future Work
* **Current Limitations:** Tracking accuracy degrades in low-light environments or during severe finger self-occlusion (e.g., fingers overlapping from the camera's perspective).
* **Future Enhancements:**
  * Add touchless **Air UI Buttons** to shift octaves up/down without touching the keyboard or mouse.
  * Implement **Performance Recording & MIDI Export** so users can save and replay melodies.
  * Build a **Guided Learning / Falling-Notes Mode** for interactive music education.