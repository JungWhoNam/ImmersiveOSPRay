# Investigating motion tracking devices
> This documentation is part of a project called [Immersive OSPray](../README.md).

We tested two sensors: 1) Microsoft Azure Kinect and 2) Intel RealSense. To come up with reliable interaction techniques, we used thier demo applications to test the tracking capability.

# 1. Microsoft Azure Kinect
Microsoft provides two SDKs for Azure Kinect sensors.
1. *Sensor SDK* provides basic features for accessing depth and RGB maps. 
2. *Body Tracking SDK* provides skeleton tracking capabilities.

We use *Body Tracking SDK* as we are interested in using positions of body joints to detect out gestures. The SDK tracks 32 joints, and each joint data object has position, orientation, and confidence-level values.

<div id="image-table">
    <table>
	    <tr>
    	    <td style="padding:0px">
        	    <img src="https://learn.microsoft.com/en-us/azure/kinect-dk/media/concepts/joint-hierarchy.png" width="388"/>
      	    </td>
            <td style="padding:0px">
            	<img src="MS%20Kinect%20-%20Body%20Tracking%20Viewer.png" width="600"/>
            </td>
        </tr>
    </table>
</div>

> (left) 32 joints tracked by Kinect's Body Tracking SDK; (right) transparency values of the spheres indicate confidence-levels

## Trials
We used the SDK's demo application to see how well these joints are tracked. 
* Tracking of `HANDTIP` and `THUMB` joints becomes unreliable when hands are moving, below a belly, or far away (more than 1.5 meters away from the sensor).
* When a user is far away, tracking of `HAND` becomes unreliable; confidence-levels become *NONE* or *LOW*, meaning that the joints are out of range or not observed.
* Joints on large body parts, e.g., `WRIST` and `CHEST`, can reliably be tracked even when a user is far away from the sensor. The joints' confidence-levels are at least *MEDIUM*. 

## Take Aways
Based on these observations, we can derive these insights. 
* Use joints on large body parts, e.g., ```WRIST```
* If interested in tracking hands, use ```WRIST``` joints instead of ```HAND``` joints.
* Detecting pinching gestures may be difficult as tracking of ```HANDTIP``` and ```THUMB``` joints are not reliable.


# 2. Intel RealSense
RealSense sensors are smaller and do not require a main power connection. Thus, RealSense sensors might be a better choice if we are interested in attaching a sensor to a handheld device.

Looking at the default viewer and examples in the Unity package, the RealSense SDK provides means to access depth and RGB images. However, Intel does not provide application-level software. Two commercial solutions are available for skeleton tracking: 1) [3Divi Nuitrack](https://nuitrack.com/) and 2) [LIPS LIPSense 3D Body Pose SDK](https://www.lips-hci.com/3d-body-pose-sdk). Demo videos from their official website indicate that their SDKs only track large body parts (not fingers).

<div id="image-table">
    <table>
	    <tr>
    	    <td style="padding:0px">
        	    <img src="Intel%20RealSense%20-%20Viewer.png" width="400"/>
      	    </td>
            <td style="padding:0px">
            	<img src="Intel%20RealSense%20-%20Unity.png" width="288"/>
            </td>
        </tr>
    </table>
</div>

> (left) Default RealSense Viewer; (right) Example from the RealSense Unity package

## Decision
In our case, sensors will be set at a fixed location. Therefore, the size is not an essential factor. However, for skeleton tracking capabilities, RealSense sensors do not provide significant benefits over Kinect sensors, and commercial SDKs are needed to be used. Thus, we decided to use the Kinect Body Tracking SDK for this iteration.