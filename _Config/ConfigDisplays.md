# Display Configuration JSON File
The demo application reads a JSON file and configures its cameras and windows (see [Extending OSPRay](OSPRay/OSPRay.md) and  [Extending OSPRay Studio](OSPRayStudio/OSPRayStudio.md)). Here is a snippet of an example JSON file. 

```
[
    ...

    {
        "hostName": "r07",

        "topLeft": [-1.887851, 1.078200, 0.613400],
        "botLeft": [-1.887851, 0.359400, 0.613400],
        "botRight": [-1.887851, 0.359400, -0.613400],
        "eye": [0.000000, 0.000000, 0.000000],
        "mullionLeft": 0.011326,
        "mullionRight": 0.011326,
        "mullionTop": 0.020733,
        "mullionBottom": 0.020733,

        "display": 0,
        "screenX": 0,
        "screenY": 0,
        "screenWidth": 3840,
        "screenHeight": 2160
    },

    ...
]
```

The JSON configuration file comprises an array of JSON objects. Each object - surrounded by curly brackets {} - contains information about an off-axis projection camera and the window that shows the camera view.
- ```hostName``` specifies the host responsible for this JSON object. 
- ```topLeft```, ```botLeft```, and ```botRight``` are positions of three corners of a projection plane, i.e., a physical display.
- ```eye``` is the camera position. 
- Four keys that start with ```mullion``` are sizes of mullions on four sides of a display. These values are used to shrink the projection plane, accounting for display frames.  
- ```display``` sets which display to show the window.
- Four keys that start with ```screen``` set the position and size of a window in screen coordinates.

# Display Configuration Generator
This Unity project is developed to assist in creating the JSON file. For instance, this project allows users to use 3D object manipulation features in Unity Editor to specify the positions of display corners and arrange display objects. Additionally, this project provides features to place these display objects into a spatial layout and scale a master display to fit all other displays. 

## Specifying a Display
![](Config%20Generator%20-%20display.png)

```Display``` object contains values that will be saved in the JSON object in the configuration file. A user uses Unity Editor to specify the positions of the three display corners and the eye by referencing Unity objects in a 3D view. The corners and eye objects are typically placed under the Display object to group these objects as one entity.

> In our current implementation, we assume one camera view per window and support specifying which display to show the window and opening multiple windows per display. Furthermore, as our camera projection plane replicates a physical display, we provide an option to input mullion values to account for display frames. 

## Arraging Displays
<div id="image-table">
    <table>
	    <tr>
    	    <td style="padding:4px">
        	    <img src="Config%20Generator%20-%203D.png" width="800"/>
      	    </td>
            <td style="padding:4px">
            	<img src="Config%20Generator%20-%20set%20up.png" width="290"/>
            </td>
        </tr>
    </table>
</div>

A user arranges these ```Display``` objects to form tiled display walls. ```Config``` object provides features for forming these ```Display``` objects into a single entity and saving it to a JSON file.
- ```Master Display```: This Display object represents a master display. 
- ```Display Containers```: Each GameObject in the list has GameObject(s) with ```Display``` components attached. In other words, each GameObject in the list groups ```Display``` objects to allow users to move these objects as a single unit.
- ```Move the eyes of Display(s)```: This moves the eyes of all the ```Display``` objects to a specific position.
- ```Position containers around Y-Axis```: The system shown in ... comprises six wall displays forming a hemisphere. This feature moves the groups defined in "Display Containers" to form a hemisphere. "From Dir" and "To Dir" set the hemisphere's start and endpoints.
- ```Reset the master display```: As the master node handles user inputs, the view displayed in the node should cover all other views. This feature scales the master display to fit all other displays. Also, the screen height value is scaled accordingly. The checkbox sets the z-value of the display to the minimum or maximum z-value of all other displays. 
- ```Save the configuration to a JSON file```: The button saves the configuration to a JSON file at a location specified in ```File name```. Negating output values can be necessary because a rendering application may use a different coordinate system.
