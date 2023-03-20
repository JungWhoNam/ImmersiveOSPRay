# Example Gestures
In this section, we describe gesture interaction techniques. 

## 1. Rotating a cube
[![](demo%20-%20rotate%20a%20cube.png)](demo%20-%20rotate%20a%20cube.mp4)
| Lifting a hand rotates the stack of slices (click to play a video) |

This example demonstrates a customized OSPRay Studio working with the gesture tracking server.  

### Gestures
By lifting a hand, a user rotates a stack of slice images.
* The image stack is stationary by default.
* Lifting a left hand rotates the stack clockwise, and lifting a right hand rotates the stack counter-clockwise. 
* When the lifted hand is closer to the body, the stack rotates slower. When the hand moves away from the body, the cube rotates faster. 

### Implementation Notes
When the client, OSPRay Studio, recieves a message from the server, it checks positions of the three joints: ```HAND_LEFT```, ```HAND_RIGHT```, and ```SPINE_CHEST```. When y-position of a hand is above that of a chest, the hand is considered as lifted. The distance between a lifted hand and the chest is used to control the rotating speed. 

The example is implemented in OSPRay Studio. See https://github.com/jungwhonam-tacc/ospray_studio/commit/3337b9ce2e34f29bec4df34a75d0658f902caeb7.

## 2. Grabbing, moving, and scaling a sphere
[![](demo%20-%20grab%20a%20sphere%20-%20scale.png)](demo%20-%20grab%20a%20sphere.mov)
| Grabbing, moving, and scaling an object (click to play a video) |

This example was built to see if we can reliably detect grabbing gestures and use these in interaction. Joints from the Kinect SDK are rendered as gray spheres; in the above image, a user is scaling a sphere with both hands.

The example is implemented in OSPRay Studio. See https://github.com/jungwhonam-tacc/ospray_studio/tree/tacc.

### Gestures
A user moves and scales a sphere.
* Grabbing a sphere is done by closing a hand
* Translation is done by grabbing the sphere with one hand and moving the hand
* Scaling is done by grabbing the sphere with both hands and pulling the hands apart
  
The color of the sphere indicates the current state:
* default - green
* moving - blue
* scaling - magenta

### Implementation Notes
When the angle between two vectors - 1) from ```PALM``` to ```TIP``` and 2) ```WRIST``` to ```PALM``` - is above a certain threahold, it is considered as the hand is closed.

We keep track of values of previous states.  
```
// previous states
bool prevClosedLeftHand;
vec3f prevPosLeftHand;
bool prevClosedRightHand;
vec3f prevPosRightHand;
```
These previous values are compared with new values received from the server in order to check how much the sphere should be moved and scaled.

The example is implemented in OSPRay Studio. See ...

### Limitations
As discussed in *Sensors* section, detecting whether a hand is closed or not is difficult to do with Kinect. As shown in the above video, the manipulating sphere freqently stops inbetween hand motions.

## 3. Head-tracking (work-in-progress)
[![](demo%20-%20head%20tracking.png)](demo%20-%20head%20tracking.mp4)
| Looking at the cubes from different perspectives (click to play a video) |

This *work-in-progress* prototype tests the quality of head tracking. The prototype uses the user's head position to change the location of camera. Along with our changes to support off-axis projection, this change should be able to create the illusion the user is looking at 3D environements through transparent windows. 

However, as shown in the video, the tracking is not perfect; it seems the cubes are moving with the head (these should more be stationary). This work is still in progress; one thing that we might be able to do is to do more precise calibrations between a sensor and displays.

## 4. Flying
In this example, a user performs a flying geseture to navigate around the scene. 

[![](../OSPRay/demo%20-%20rattler.png)](../OSPRay/demo%20-%20rattler.MOV)
| Gesture-based scene navigation in tiled-display walls (click to play a video) |

### Gestures
Lifting both hands above a belly button triggers a flying mode.
Once in the mode, a camera moves into a body-leaning direction.

### Implementation Notes
When y-position of ```WRIST``` is above that of ```SPINE_NAVEL```, the hand is considered as lifted up.

The leanging direction is computed by projecting the vector from ```NECK``` to ```SPINE_NAVEL``` to a x-z plane. The projected direction vector is then used to move the camera. As the vector was projected to a x-z plane, the camera does not move up or down.

The example is implemented in both OSPRay and OSPRay Studio. 
- See https://github.com/jungwhonam-tacc/ospray/releases/tag/v2.11.0-alpha.3 for OSPRay
- See https://github.com/jungwhonam-tacc/ospray_studio/tree/devel for OSPRay Studio

# Take Aways
* Detecting whether a hand is closed or not is difficult to do as tracking of corresponding joints, ```HANDTIP``` and ```THUMB```, are not reliable.
* Tracking of ```HAND``` joints are not reliable (its confidance level frequtnly drops to low). Thus, we recommend using ```WRIST``` joints instaed.
* Using gestures that rely on large body parts seem to work okay. In the above examples, the first and fourth examples use large body parts.