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

---

## Build Processes
Follow these steps to build OSPRay and OSPRay Studio and run our example mode in OSPRay Studio. 

### Build OSPRay
Make sure to use the branch ```v2.11.0-alpha.x``` and use the commit ```6a27b0a19c6711c56c1269b7a2494bc4963cbeae``` is used.
```
git clone https://github.com/jungwhonam-tacc/ospray.git
cd ospray

# switch to the alpha branch
git switch v2.11.0-alpha.x

# revert to the previous commit (currently fixing a bug)
git reset --hard HEAD~1

# create "build/release"
mkdir build
cd build
mkdir release

cmake -S ../scripts/superbuild -B release -DCMAKE_BUILD_TYPE=Release -DBUILD_OSPRAY_MODULE_MPI=ON -DBUILD_EMBREE_FROM_SOURCE=OFF

cmake --build release -- -j 5

cmake --install release
```

### Build OSPRay Studio
Make sure to set ```ospray_DIR``` so CMake can locate OSPRay. Run following command. 

```
git clone https://github.com/jungwhonam-tacc/ospray_studio.git
cd ospray_studio

# switch to the devel branch
git switch devel

# create "build/release"
mkdir build
cd build
mkdir release

cmake -S .. \
-B release \
-DCMAKE_BUILD_TYPE=Release \
-DUSE_PYSG=OFF \
-DUSE_MPI=ON \
-DBUILD_PLUGINS=ON \
-DBUILD_PLUGIN_GESTURE=ON \
-Dospray_DIR="/Users/jnam/Documents/Test/ospray/build/release/install/ospray/lib/cmake/ospray-2.10.0"

cmake --build release -- -j 5

cmake --install release
```

### Run our example mode
First, put the [```config```](config) folder to ```build/release/install/bin```, and run the following command. See [Extending OSPRay Studio](OSPRayStudio/OSPRayStudio.md) for the inputs.

```
mpirun -n 3 \
./ospStudio \
multiwindows \
--mpi \
--displayConfig config/display_settings.json \
--scene multilevel_hierarchy \
--plugin gesture \
--plugin:gesture:config config/tracking_settings.json
```