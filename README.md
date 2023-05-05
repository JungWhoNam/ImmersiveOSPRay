# Immersive OSPRay
[![](demo%20-%20rattler.png)](demo%20-%20rattler.MOV)

We present an extension to [Intel's OSPRay Studio](https://www.ospray.org/ospray_studio/) to enable immersive virtual reality experiences on high-res tiled-display walls. 
This extension allows us to do the following:

<!-- two criteria for VR experiences; one criteria for public exhibitions -->
1. Display a single, coherent virtual environment on tiled display walls
   - Extended its core rendering engine to support off-axis projection ([Extending OSPRay](https://github.com/jungwhonam-tacc/ospray/tree/v2.11.0-alpha.x))
   - Added another mode in OSPRay Studio to simultaneously run multiple windows and move/scale these windows to create a large window ([Extending OSPRay Studio](https://github.com/jungwhonam-tacc/ospray_studio/tree/v0.12.0-alpha.x))
2. Use gesture-based interaction techniques
   - Tested two sensors, Microsoft Kinect and Intel RealSense, for their body tracking capabilities ([Sensors](Sensors/Sensors.md))
   - Implemented four example gestures to test their usabilities ([Gestures](Gestures/Gestures.md))
   - Implemented a server that sends body tracking data to connected clients ([Gesutre Tracking Server](https://github.com/jungwhonam-tacc/GestureTrackingServer))
3. Run the application in various public exhibition settings
   - Containerized the application ([Container](Container/Container.md))
   - Used JSON configuration files to modify display and tracking settings without rebuilding the application ([Extending OSPRay Studio](https://github.com/jungwhonam-tacc/ospray_studio/tree/v0.12.0-alpha.x))
   - Created a Unity project to assist users with creating the JSON file for display settings ([Configuration Generator](https://github.com/jungwhonam-tacc/ConfigurationGenerator))

## Demo
With the added features, we created a demo that:
1. Display a scene on [TACC's Rattler system](https://www.tacc.utexas.edu/systems/rattler/) - a tiled-display system driven by a cluster of 19 Linux PCs
2. Enable a user to use a flying gesture to move in a 3D virtual environment
3. Run on systems that do not have all the required dependencies installed

Click the above image to play a video. 

In OSPRay Studio, a new mode supports the above features ([Extending OSPRay Studio](https://github.com/jungwhonam-tacc/ospray_studio/tree/v0.12.0-alpha.x)). For running containerized versions, see [Container](Container/Container.md).


# GitHub Repos

OSPRay
* https://github.com/jungwhonam-tacc/ospray/tree/v2.11.0-alpha.x

OSPRay Studio
* https://github.com/jungwhonam-tacc/ospray_studio/tree/v0.12.0-alpha.x

Gesture Tracking Server
* https://github.com/jungwhonam-tacc/GestureTrackingServer

Configuration File Generator
* https://github.com/jungwhonam-tacc/ConfigurationGenerator

Containers
* https://github.com/GregAbram/TACCOspray


# Build Processes
Follow these steps to build OSPRay and OSPRay Studio and run a special mode in OSPRay Studio. 

1. [Build OSPRay](https://github.com/jungwhonam-tacc/ospray/tree/v2.11.0-alpha.x)
2. [Build OSPRay Studio](https://github.com/jungwhonam-tacc/ospray_studio/tree/v0.12.0-alpha.x#cmake-configuration-and-build)
3. [Run OSPRay Studio with a MULTIWINDOWS mode](https://github.com/jungwhonam-tacc/ospray_studio/tree/v0.12.0-alpha.x#run-the-application)