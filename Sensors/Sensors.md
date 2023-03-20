# Investigating Tracking Sensors

# Microsoft Azure Kinect
Microsoft provides two SDKs for thier Azure Kinect sensors.
1. *Sensor SDK* provides basic features such as for accessing depth and RGB maps. 
2. *Body Tracking SDK* provides skeleton tracking capabilities.

We use the Body Tracking SDK, as we are interested in using positions of body joints to detect out gestures. The SDK tracks 32 joints including hand-tips and thumbs. 

![](https://learn.microsoft.com/en-us/azure/kinect-dk/media/concepts/joint-hierarchy.png)
| *32 joints tracked by the Body Tracking SDK* |

## Trials
We used the SDK's demo application to see how well these joints are tracked. 

![](MS%20Kinect%20-%20Body%20Tracking%20Viewer.png)
| *Transparency values of the spheres indicate confidence-levels* |

### Observation #1
Tracking of ```HANDTIP``` and ```THUMB``` joints becomes unreliable when
- hands are moving
- the user is far away (about > 1.5 meters away) from the sensor
- hands are below a belly

### Observation #2
When the user is more than 1.5 meters away from the sensor, ```HAND``` tracking becomes unreliable; Its tracking confidence-level becomes ```NONE``` or ```LOW```, meaning that the joint is out of range or not observed. 

### Observation #3
Joints in large body parts, e.g., ```WRIST```, are more reliable even when the user is far away from the sensor. The joints' confidence-levels are at least ```MEDIUM```. 

## Take Aways
Based on the observations, we can derive these insights. 
* Use joints on large body parts, e.g., ```WRIST```
* If interested in tracking hands, use ```WRIST``` joints instead of ```HAND``` joints.
* Detecting pinching gestures may be difficult as tracking of ```HANDTIP``` and ```THUMB``` joints are not reliable.



# Intel RealSense
RealSense sensors are smaller in size and do not require a main power connection. Thus, RealSense sensors might be a better choice if we are interested in attaching a sensor to a handheld device.

Looking at the default viewer and examples in the Unity package, the RealSense SDK provides means to access depth and RGB images. 

![](Intel%20RealSense%20-%20Viewer.png)
| *Default RealSense Viewer* |

![](Intel%20RealSense%20-%20Unity.png)
| *Example from the RealSense Unity package* |

However, Intel does not provide application-level software. Two commericial solutions are available for skeleton tracking.
* [3Divi Nuitrack](https://nuitrack.com/)
* [LIPS LIPSense 3D Body Pose SDK](https://www.lips-hci.com/3d-body-pose-sdk)

Demo videos from thier official websites indicate that thier SDKs 
* only track large body parts (not fingers)
* quality of tracking is not better than that of Kinect

## Decisions
In our case, sensors will be set at a fixed location. Therefore, the size is not an important factor. For skeleton tracking capabilities, RealSense sensors do not provide significant benefits over Kinect sensors, and the commericial SDKs are needed to be used. Thus, we will use the Kinect Body Tracking SDK for this iteration.


# Other Possible Seonsors or SDKs
Both Kinect and RealSense are discontinued. There are other sensors and SDKs may be worth a look.
* [Google MediaPipe](https://google.github.io/mediapipe/)
* [Captury](https://captury.com/)