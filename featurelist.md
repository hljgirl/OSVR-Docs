# Key OSVR Features

# open-source
- Free and open-source
- Apache 2.0 license
- Supports closed-source plugins

# Universal device support
OSVR supports over 100 devices. The OSVR philosophy is to define an **interface**, essentially a pipe of data that provides reports of a certain type. The following interface types are available:
- Display
- Tracker
- Analog
- Button
- Config
- Eye tracker
- Imager
- Gesture
- Locomotion
- Skeleton
- Poser

Just like a multi-function printer can be viewed by Windows as one physical device that has multiple software interfaces (e.g. printer, fax, scanner), a physical device such as a Razer Hydra can map into one or more OSVR interfaces.

# Semantic Paths
- Flexible remapping of buttons from different hardware to the same application function.
- Configuration of tracker-sensor position to provide consistent coordinate systems between devices.
- Compile-free conversion from tracker-based to controller-based head tracking.

# High-performance rendering support
- Time warp
- Direct render
- Front-buffer rendering
- Distortion correction
- Predictive tracking
- Vsync timing reports

# Plugin architecture
- Plugins loaded at runtime
- Supports both open-source and closed-source plugin

# Portable
- Cross-platform code that can compile on multiple operating systems: Windows, Linux, Android, OS X and others.
- Cross-library rendering that can display using Direct3D11, OpenGL, Unity, Unreal, WebGL, Blender Game Engine, and others.
- Cross-display configuration that can enable an application to run on different HMDs without change, and also run in debug mode on a mono window.

# Programming model
OSVR supports both:
- Synchronous (blocking) calls
- Asynchronous (callback)

# Distributed
- OSVR can run on a single machine, but can also be distributed over a network. This allows many-to-many relationship between OSVR clients (applications) and OSVR servers (sensor processors). Clients and servers may also reside on different operating systems.
