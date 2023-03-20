# "Hello, World!"
![](OSPRay/demo%20-%20rattler.png)
We present an extension to 
Intel OSPRay Studio to enable immersive virtual reality experiences on high-res tiled-display walls. 
This extension allows us to do the following:
<!-- two criteria for VR experiences
one criteria for public exhibitions -->
1. Display a single, coherent virtual environment on tiled display walls
   - Extended its core rendering engine to support off-axis projection (see [Extending OSPRay](OSPRay/OSPRay.md))
   - Added another mode in OSPRay Studio to simultaneously run multiple windows and move/scale these windows to create a large window (see [Extending OSPRay Studio](OSPRayStudio/OSPRayStudio.md))
2. Use gesture-based interaction techniques
   - Tested two sensors, Microsoft Kinect and Intel RealSense, for their body tracking capabilities (see [Sensors](Sensors/Sensors.md))
   - Implemented four example gestures to test their usabilities (see [Gestures](Gestures/Gestures.md))
   - Implemented a server that sends body tracking data to connected clients (see [GesutreTrackingServer](Server/GesutreTrackingServer.md))
3. Run the application in various public exhibition settings
   - Containerized the application (see [Container](Container/Container.md))
   - Used JSON configuration files to modify display and tracking settings without rebuilding the application (see [Extending OSPRay](OSPRay/OSPRay.md) and [Extending OSPRay Studio](OSPRayStudio/OSPRayStudio.md))
   - Created a Unity project to assist users with creating the JSON file for display settings (see [Configuration Generator](ConfigGenerator/ConfigGenerator.md))

## Demo Applications

With the added features, we created demo applications that:
1. Display a scene on TACC's Rattler system - a tiled-display system driven by a cluster of 19 Linux PCs
2. Enable a user to use a flying gesture to move in a 3D virtual environment
3. Run on systems that do not have all the required dependencies installed

See our demos in [Extending OSPRay](OSPRay/OSPRay.md) and  [Extending OSPRay Studio](OSPRayStudio/OSPRayStudio.md). In OSPRay Studio, a new mode supports the above features (not created as a separate application like the OSPRay demo application). For running containerized demos, see [Container](Container/Container.md).

---

## GitHub Repos

OSPRay
* https://github.com/jungwhonam-tacc/ospray

OSPRay Studio
* https://github.com/jungwhonam-tacc/ospray_studio

Gesture Tracking Server
* https://github.com/jungwhonam-tacc/GestureTrackingServer

Config File Generator
* https://github.com/jungwhonam-tacc/ConfigurationGenerator

Containers
* https://github.com/GregAbram/TACCOspray