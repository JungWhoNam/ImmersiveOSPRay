# Extending OSPRay Studio
First, we need to integrate our new OSPRay in the build (see 1. Configuration and Run).

We implemented features that allow us to open multiple windows and syncrhonize contents shown in the windows. The implementation is the same as what we have done for the OSPRay demo application (see [Extending OSPRay](../OSPRay/OSPRay.md)), except for these differences:
- We created a separate application forthe OSPRay demo. For OSPRay Studio, we created another mode instead. The mode can be actiavated by passing a command line option (see 2. Introducing another mode).
- In OSPRay Studio, gesture-based interaction part is decoupled as a plugin (see 3. Gesture Plugin). 

# 1. Configuration and Run 
## CMake configuration
OSPRay Studio needs to be built with ```-DUSE_MPI=ON```, ```-DBUILD_PLUGINS=ON```, and ```-BUILD_PLUGIN_GESTURE=ON``` in CMake. We need to use the OSPRay we have customized. After building the OSPRay, set ```ospray_DIR``` so CMake can locate OSPRay, e.g., ```/Users/jnam/Documents/GitHub/ospray/build/install/ospray/lib/cmake/ospray-2.10.0```.

## Run the application
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

```multiwindows```: This option activates our custom mode.

```--mpi```: This option enables the OSPRay Studio's built-in MPI support, which is a required dependency of our custom mode.

````--displayConfig config/display_settings.json````: The JSON configuration file contains information about off-axis projection cameras and windows. Information in the file is used to position and scale windows.

```--scene multilevel_hierarchy```: This option starts the application with the scene opended (optional).

```--plugin gesture```: This option starts the application with the gesture plugin.

```--plugin:gesture:config config/tracking_settings.json```: The JSON configuration file contains information about the gesture tracking server and user tracking data. This file is used by the gesture plugin.

# 2. Introducing another mode
OSPRay Studio provides different modes of running the application. We support another mode called ```MULTIWINDOWS```, which is based on the default ```GUI``` mode with these additional features:
- Position and scale windows based on MPI ranks
- Synchronize processes

To implement the mode, we copied ```app/MainWindow.cpp``` and modified parts. See ```app/MultiWindows.cpp```.

## Position and scale windows based on MPI ranks
This new OSPRay Studio mode takes a command line option, ```--displayConfig```, which points to a JSON configuration file. The file contains specifications about windows and off-axis cameras, and it is the same as the configuration file used in the OSPRay demo application (see [Extending OSPRay](../OSPRay/OSPRay.md)). See ```void MultiWindows::addToCommandLine(std::shared_ptr<CLI::App> app)``` for implementation.

At the start of application, the JSON file is loaded and the data is stored in a JSON object ```nlohmann::ordered_json configDisplay```. Similar to the OSPRay demo application, positioning and scaling GLFW windows are done in a constructor.

## Synchronize processes
At the start of each frame, a shared state object from the master node is broadcasted to other processes. The shared state is then used to update corresponding objects in a scene. 

```
while (true) {
    MPI_Bcast(&sharedState, sizeof(sharedState), MPI_BYTE, 0, MPI_COMM_WORLD);

    { // process changes in the shared state
      if (sharedState.quit) {
        break;
      }

      if (sharedState.camChanged) {
        auto camera = frame->child("camera").nodeAs<sg::Camera>();
        camera->child("transform").setValue(sharedState.transform);
        camera->child("topLeft").setValue(xfmPoint(sharedState.transform, topLeftLocal));
        camera->child("botLeft").setValue(xfmPoint(sharedState.transform, botLeftLocal));
        camera->child("botRight").setValue(xfmPoint(sharedState.transform, botRightLocal));

        sharedState.camChanged = false;
      }
    }

    ...
}
```

In our current implement, a camera location and a closing status are synchronized across processes.
```
struct SharedState
{
  bool quit;

  bool camChanged;
  affine3f transform;

  SharedState();
};
```

The master node, a rank zero process, handles user interaction events, e.g., key pressed events, and processes user tracking data received from the server. After this, the master node updates its scene and camera. And at the end of each frame, the master node updates ```sharedState``` that will be shared across processes. 

```
while (true) {
    
    ...

    // poll and process events
    glfwPollEvents();
    if (sg::sgMpiRank() == 0) {
        // poll and process events from the server
        for (auto &p : pluginPanels)
            p->process("update");

        // update the shared state
        sharedState.camChanged = true;
        sharedState.transform = arcballCamera->getTransform();
        sharedState.quit = glfwWindowShouldClose(glfwWindow) || g_quitNextFrame;
    }
    MPI_Barrier(MPI_COMM_WORLD);
}
```

# 3. Gesture Plugin
![](Gesture%20Plugin.png)

The plugin can be opened by clicking "Plugins/Gesture Panel" in the menu. 

On the top, information about the server is shown. The button is for starting the connection and closing it. The ``configuration`` panel provides GUIs for modifying the values that are used to remap the body tracking data received from the server. These values are written in a JSON configuration file. The ``save`` button on the bottom is used to save the current values in the JSON file. The ``Status`` panel prints out important statues, e.g., indiciating it is connecting to the server or closing the connection.

See ```plugins/gesture_plugin``` for implementation.

## Tracking Manager
This class manages the socket connection and keeps track of the latest user tracking data from the gesture tracking server. See ```plugins/gesture_plugin/tracker```.

At the start, the information about the socket, ip address and port number, is read from a JSON configuration file. The file also contains information about how to process the user tracking data.

```
{
    "ipAddress": "129.114.10.237",
    "portNumber": 8888,
    
    "scaleOffset": [0.001, -0.001, -0.001],
    "translationOffset": [0.0, -0.1, 1.19],
    "confidenceLevelThreshold": 1,
    "leaningAngleThreshold": 1.0,
    "leaningDirScaleFactor": [1.0, 1.0, 1.0]
}
```
In addition to providing methods for starting and closing the connection, it keeps track of the lastest message received from the server. The data can be accessed by calling ```pollState()```. When the method is called, the object that stores the latest information becomes empty, indicating the data has been used. In the plugin, the method is called in ```process(std::string key)```. 

```
void PanelGesture::process(std::string key) {
  if (key == "update") {
    TrackingState state = trackingManager->pollState();
    if (state.mode == INTERACTION_FLYING) {
      context->arcballCamera->move(state.leaningDir);
    }
  }
  else if (key == "start") {
    trackingManager->start();
  }
}
```

The manager also figure out current gestures. When a message is received from the server, ```updateState(std::string message)``` is called to process the message. In our current implementation, we compute a leaning direction and the current gesture mode.


# Bug Fixes
This alpha version fixes few bugs. See the list in https://github.com/jungwhonam-tacc/ospray_studio/issues.
* problems with building the example plugin : https://github.com/jungwhonam-tacc/ospray_studio/issues/1
* problems with installing in Debug mode: https://github.com/jungwhonam-tacc/ospray_studio/issues/2
* problems with building in Debug mode: https://github.com/jungwhonam-tacc/ospray_studio/issues/3