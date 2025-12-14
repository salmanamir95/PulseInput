Project Name:

PulseInput: Real-Time USB Controller Firmware

Tagline:
“Sub-millisecond deterministic input for competitive and high-performance PC games.”

1️⃣ Software Requirements Specification (SRS)
1.1 Purpose

PulseInput provides hard real-time USB controller input capture for PC gaming systems. It ensures deterministic input polling, DMA-based buffering, and low-latency delivery of controller signals to higher-level game engines or runtime systems.

Target Audience:

Game developers working on competitive PC games

Indie developers building custom hardware controllers

Esports platforms requiring low-latency input

1.2 Scope

Firmware runs on standard USB HID devices or microcontroller-based custom controllers.

Captures button presses, analog sticks, triggers, and haptic feedback commands.

Guarantees frame-accurate input with sub-millisecond latency.

Provides a simple API for higher-level systems to query input buffers.

Modular, maintainable, and extensible for new input devices.

1.3 Functional Requirements

Input Capture

Poll USB HID devices at configurable rates (up to 1 kHz).

Support multi-device simultaneous capture.

DMA-Based Buffering

Store input events in circular DMA buffers for minimal CPU overhead.

Handle buffer overflows gracefully.

Deterministic Delivery

Guarantee order and timing of events to the application layer.

Hot-Plug Detection

Detect controller connect/disconnect events without crashing.

API

Functions for reading buffered events, resetting buffers, and querying device state.

Configuration

Adjustable sampling frequency, buffer size, and latency thresholds.

1.4 Non-Functional Requirements

Performance: Input-to-application latency < 1 ms.

Memory: Minimal stack/heap usage; DMA preferred.

Reliability: 99.999% uptime during gameplay.

Portability: Compatible with Windows, Linux, and macOS (via USB HID).

Scalability: Support up to 8 simultaneous controllers.

Safety: Graceful handling of invalid USB packets or device errors.

1.5 Constraints

Must run on standard PC USB stack or small MCU (e.g., STM32).

Must not use dynamic memory allocations in hot paths.

Must integrate cleanly with higher-level engine or runtime systems.

2️⃣ Developer Notes / Architecture
2.1 High-Level Architecture
┌─────────────────────────────┐
│  USB Controller / HID Device│
└─────────────▲───────────────┘
              │ USB Poll / Interrupt
┌─────────────┴───────────────┐
│  DMA Input Buffer           │  <- Circular buffer, lock-free
└─────────────▲───────────────┘
              │
┌─────────────┴───────────────┐
│  C Firmware API Layer       │
│  - getNextInputEvent()      │
│  - resetBuffer()            │
│  - queryDeviceStatus()      │
└─────────────▲───────────────┘
              │
┌─────────────┴───────────────┐
│  Higher-Level Game Engine   │
│  - Polls API for inputs     │
│  - Uses deterministic input │
└─────────────────────────────┘

2.2 Modules

USB Polling Module

Interrupt-driven

Detect device connect/disconnect

Normalize raw HID input

DMA Buffer Module

Circular buffer

Lock-free, zero-copy design

Handles overflow by dropping oldest or latest (configurable)

API Module

Provides simple read functions

Exposes device status, input events

Configuration Module

Sampling rate, buffer size

Latency monitoring thresholds

Logging / Debug

Optional serial output for debugging device events

2.3 Key Design Decisions

Language: C for low-level memory & hardware access

DMA: Chosen to reduce CPU overhead and ensure deterministic timing

Circular buffer: Ensures minimal memory footprint and predictable event ordering

Interrupt-driven polling: Guarantees input capture under variable CPU load

3️⃣ UX Flow / Developer Interaction

Since this is firmware, “UX” is really API usage flow for game engine developers.

3.1 Initialization Flow

initializeDevice(deviceID)

Detects controller and allocates DMA buffer

configureDevice(samplingRate, bufferSize)

Sets sampling frequency and buffer parameters

startPolling()

Begins capturing input events

3.2 Runtime Flow
Game Loop (Every Frame)
   │
   ├─> call getNextInputEvent()
   │      - returns button/analog state with timestamp
   │
   ├─> process input deterministically
   │
   └─> use input in game logic / physics / network

3.3 Hot-Plug / Error Handling

onDeviceConnected(callback)

onDeviceDisconnected(callback)

onBufferOverflow(callback)

This allows seamless integration into engines without blocking.

4️⃣ Developer Notes / Best Practices

Avoid heap allocations in hot paths; pre-allocate buffers at init.

Use volatile for DMA-accessed memory to prevent compiler reordering.

Minimize ISR work; DMA handles bulk transfer.

Always timestamp inputs for rollback or deterministic replay.

Test with variable frame rates to ensure determinism.

5️⃣ Optional UI / Debug Flow

Even though firmware, you can provide a simple debug tool in C# or Python:

UI Components:

Device List: Shows connected controllers

Event Log: Streams real-time button/analog events

Latency Monitor: Shows buffer depth, input-to-API latency

Replay / Simulation: Playback recorded input events

Flow:

User connects controller → device detected

Starts polling → events displayed live

Latency and buffer stats monitored → alerts if over threshold

6️⃣ Metrics / Performance Goals
Metric	Target
Input-to-API latency	<1 ms
Buffer overflow frequency	0% under normal load
Max simultaneous devices	8
CPU usage (input capture only)	<2% per device
Determinism	100% ordered input per frame
✅ Summary

Project Name: PulseInput

Layer: Firmware / C / Low-level PC Input

Goal: Deterministic, low-latency USB controller input for games

Key Strength: Hard real-time capture, DMA buffering, deterministic API

Value to Games: Competitive multiplayer, precise physics, arcade/VR support
