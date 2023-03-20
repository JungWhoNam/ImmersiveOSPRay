# Extending OSPRay
We extend OSPRay v2.10.0 to support off-axis projection and create a proof-of-concept prototype demonstrating gesture-based interaction techniques (see the GitHub repo https://github.com/jungwhonam-tacc/ospray/tree/v2.11.0-alpha.x)

# Off-Axis Projection
```PerspectiveCamera``` is extended so that one can 
- define an image plane that is not orthogonal to the camera's viewing direction
- specify an image plane by giving positions of three corners of the plane [^1]

Four variables are added to the structure. ```offAxisMode``` indicates the current viewing mode. The three ```vec3f``` variables are the positions of the three corners. 
```
// modules/cpu/camera/PerspectiveCamera.h

bool offAxisMode{false};
vec3f topLeft;
vec3f botLeft;
vec3f botRight;
```

 If ```offAxisMode``` is ```false```, the camera behaves like the original mode; it uses the field of view, viewing direction, etc., to compute an image plane. However, when ```offAxisMode``` is ```true```, the camera creates an image plane using the three corners. 

```
// modules/cpu/camera/PerspectiveCamera.cpp

if (offAxisMode) {
  getSh()->du_size = botRight - botLeft;
  getSh()->dv_up = topLeft - botLeft;
  getSh()->dir_00 = botLeft - getSh()->org;
}
```

[^1]: In our case, three corners of an image plane would be corners of a physical display.

# Demo Application
We created an example called ```ospMPIMultiDisplays```. Compared to examples in ```modules/mpi/tutorials```, ```ospMPIMultiDisplays``` does
* display a single, coherent 3D virtual environemnt on non-planar, tiled-display walls
  * open multiple windows  (each window shows the scene from a camera view using the off-axis projection technique)
  * move and scale the windows so that these windows form a single large window
* support gesture interaction techniques to move around a 3D virtual environemnt

The implemention is based on existing example codes, such as ```ospExamples``` and ```ospMPIDistributedExamples```. The codes for the example are placed under ```modules/mpi/multiDisplays```.

[![](demo%20-%20rattler.png)](demo%20-%20rattler.MOV)
| Move around a 3D scene using a flying gesture (click to play a video) |

## Running the application
```
mpirun -n 3 \
ospMPIMultiDisplays \
config/display_settings.json \
config/tracking_settings.json
```
The application uses MPI to run multiple processes and synchronize these (each process is an instance of the application - a window on a display). The application takes two JSON configuration files, which are set in command line inputs. The first file configures displays. The other file configures user body tracking data. If these two command line arguments are note provided, the example looks at default locations.
- ```config/display_settings.json```
- ```config/tracking_settings.json```

## Uses MPI to run multiple processes
Existing examples under tutorials use a special MPI-enabled renderer. However, their examples only show example usages of MPI where multiple processes have the same camera properties and the difference is a portion of data the process is responsible for. In other words, each process renders a portion of data. 

Our need is different; we need each process to have the same data but have different camera properties. We extended ```ospExamples```, a non-MPI example, to include MPI support that do these things based on the current rank.
* scale and place the window 
* update the camera properties
* add user interaction callbacks

Setting up these properties can only happen in an initialization (see the constructor of ```modules/mpi/multiDisplays/GLFWOSPRayWindow.cpp```).

## JSON Configuration Files
The example takes two configuration files.

### JSON Configuration File for Cameras and Windows
The below snippet is from ```modules/mpi/multiDisplays/config/display_settings.json```.

```
[
{
  "hostName": "localhost",

  "topLeft": [0.178950, 0.122950, 1.000000],
  "botLeft": [0.178950, -0.122950, 1.000000],
  "botRight": [-0.178950, -0.122950, 1.000000],
  "eye": [0.000000, 0.000000, 0.000000],
  "mullionLeft": 0.006320,
  "mullionRight": 0.006320,
  "mullionTop": 0.015056,
  "mullionBottom": 0.015056,

  "display": 0,
  "screenX": 0,
  "screenY": 0,
  "screenWidth": 1024,
  "screenHeight": 640
},
...
]
```
The configuration file is composed of an array of objects, and each object - surrounded by curly braces {} - contains information about a single, off-axis projection camera. In our example application, a window is dedicated to a camera. Thus, each JSON object contains information about both the camera and the window.  

Key/value pairs on camera (in meters)
- ```topLeft```, ```botLeft```, and ```botRight``` set the locations of three corners of a display. 
- ```eye``` is the default camera location.
- Four keys that start with ```mullion``` set the sizes of mullions on four-sides of a display.

Key/value pairs on window
- ```display``` sets which display to show the window (useful when there are multiple display devices connected).
- Four keys that with ```screen``` set the position and size of a window.

### JSON Configuration File for Body Tracking
The below snippet is from ```modules/mpi/multiDisplays/config/tracking_settings.json```.
```
{
    "ipAddress": "129.114.10.237",
    "portNumber": 8888,
    
    "multiplyBy": [0.001, -0.001, -0.001],
    "positionOffset": [0.0, -0.1, 1.19],
    "leaningAngleThreshold": 1.0
}
```
- the first two key/value pairs are information about the gesture tracking server.
- ```multiplyBy``` is multiplied to position values of joints. This process is needed as Kinect and OSPRay are in two different coordinate systems. 
- ```positionOffset``` are used to offset the sensor's center. The offset is applied to calibrate the sensor and displays. 
- ```leaningAngleThreshold``` is a threshhold for activating the flying mode. When a user's body is leaning more than the angle, the flying mode is activated. 

### Implementation
For the configuration files, we use *JSON for Mondern C++* library. v3.10.4 is used as it is the same version used by rkcommon, one of dependencies for OSPRay Studio. The library is placed under ```modules/mpi/multiDisplays/external/json```.
- See ```modules/mpi/multiDisplays/tracking/TrackingManager.cpp``` for loading the configuration for connecting to the server. 
- See ```modules/mpi/multiDisplays/GLFWOSPRayWindow.cpp``` for loading the displays configuration. 

## Gesture-based Controls
The flying gesture is integrated to support scene navigation using gestures. After receiving user tracking data from the server, the demo application processes the data in order to figure out the current gesture mode and navigating direction (details about computing this information are described in ```Gestures.md```).

See ```modules/mpi/multiDisplays/tracking/TrackingManager.cpp``` for the implementation.

# Bug Fixes
This alpha version fixes few bugs. See the list in https://github.com/jungwhonam-tacc/ospray/issues.
* problems with snappy: https://github.com/jungwhonam-tacc/ospray/issues/1
* problems with finding libtbb: https://github.com/jungwhonam-tacc/ospray/issues/2
* problems with finding openvkl: https://github.com/jungwhonam-tacc/ospray/issues/2
* program crashes when pressing 'g': https://github.com/jungwhonam-tacc/ospray/issues/4

# Future Work
Currently the off-axis projection is implemented on top of ```PerspectiveCamera.cpp```. Depending on the current mode, there are variables not being used. We plan to create another struct object, called ```OffAixProjectionCamera``` in order to keep the original camera intact and move the off-axis projection related stuff.

Also we plan to remove the existing demo application as OSPRay focuses on rendering. The existing demo application can be implemented in OSPRay Studio. To demonstrate the off-axis projection capabilities, we can instead create a simpler demo, e.g., creating two windows that look like a window.