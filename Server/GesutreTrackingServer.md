# Gesture Tracking Server
See the implementation in https://github.com/jungwhonam-tacc/GestureTrackingServer/tree/windows.

The server gets user tracking data from sensors. It then parses the data (removing unecessary information, e.g., orientations of joints), and send the data to connected clients.

![](Gesture%20Server.png)
| *the server sends the tracking data to connected clients at every frame* |

```k4abt_frame_t``` is a struct from Kinect SDK, and contains skeleton(s) at the current frame. A skeleton contains a list of joints; each joint contains a position, an orientation, and a confidence_level.

```GestureDetector``` extracts important information from ```k4abt_frame_t```, and packages the information into a JSON format. Currently, it extracts positions and confidence_levels of all the joints provided by the SDK. 

```async-sockets``` opens a TCP socket and keeps tracks of connected clients. It sends the JSON string from ```GestureDetector``` to client(s).

## Four Dependencies
### 1. Azure Kinect SDK (currently using v1.4.1)
https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md

```cmake/k4a.cmake``` looks for the library installed in your computer. 

### 2. Azure Kinect Body Tracking SDK (currently using v1.1.2)
https://learn.microsoft.com/en-us/azure/kinect-dk/body-sdk-download

```cmake/k4abt.cmake``` looks for the library installed in your computer. 

### 3. JSON for Modern C++ (currently using v3.10.4)
https://github.com/nlohmann/json

It's a header-only library. v3.10.4 is used as it is the same version used by rkcommon, one of dependencies for OSPRay Studio. The library is placed under ```external/json```.

### 4. async-sockets (using the version from the last commit on 2/21/2022)
https://github.com/eminfedar/async-sockets-cpp

We use this library for TCP networking. We update the library to support Windows. This is a header-only library, and is placed under ```external/async-sockets```.