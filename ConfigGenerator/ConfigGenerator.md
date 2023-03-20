# Configuration Generator
The JSON display configuration file is used by our OSPRay demo application and new OSPRay Studio mode (see [Extending OSPRay](OSPRay/OSPRay.md) and  [Extending OSPRay Studio](OSPRayStudio/OSPRayStudio.md)). Specifying the positions of display corners can be difficult without graphical assistance. Thus, we created this Unity project to take advantage of UI capabilities provided by Unity Editor to set the position values. In addition, we added features to make a JSON file from these set objects. 

Check the GitHub repo: https://github.com/jungwhonam-tacc/ConfigurationGenerator.

# Specifying a Display
In OSPRay and OSPRay Studio, we assume that an off-axis projection camera is associated with a GLFW window. In Unity, ```Display``` script sets the camera and the window. A user can use graphical UI tools in Unity Editor to set the values for the camera and the window. For example, the image below shows a display configuration for a node "r07" window.

![](Config%20Generator%20-%20display.png)

This information is then saved as a JSON object in an array format.

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

# Creating a Configuration
A user arranges these ```Display``` objects to form tiled-display walls. The image below shows a 3D view in the Editor. 
![](Config%20Generator%20-%203D.png)
```Config``` script provides features for forming these ```Display``` objects into a single entity and saving it to a JSON file.
The editor script provides these additional features:

<img align="left" width="400" src="Config%20Generator%20-%20set%20up.png" style="padding-right:20px;">

- ```Display Containers```: Each GameObject in the list has GameObject(s) with ```Display``` components attached. In other words, each GameObject in the list groups ```Display``` objects so a user can move these objects together. When saving a JSON file, the script goes through each GameObject in the list and finds all Display components.
- ```Move the eyes of Display(s)```: It moves the eyes of all the ```Display``` objects to a specific position.
- ```Position containers around Y-Axis```: Rattler at TACC, for instance, comprises six wall displays forming a hemisphere. This feature moves "Display Containers" to create a hemisphere. "From Dir" and "To Dir" set the sphere's start and endpoints.
- ```Reset the master display```: A master node is typically used for user interaction. Thus the master display should cover all the views captured by other displays. This feature scales the master display to fit it to cover all the displays. The checkbox is for setting the z-value of the master display to max or min.
- ```Save the configuration to a JSON file```: The button saves the configuration to a JSON file at a location specified in ```File name```. Negating output values is necessary because OSPRay uses a different coordinate system.