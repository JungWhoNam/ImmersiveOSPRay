# Containerize the application
As other demo applications might be running on systems, we should be able to deploy our application without breaking existing demos. For this, we containerized our application. We create Dockerfiles that set dependencies for building OSPRay and OSPRay Studio (tested with OSPRay v2.10.0 and OSPRay Studio v0.11.1).

In [the repo](https://github.com/GregAbram/TACCOspray), there are three docker files:
1. ```Dockerfile.nvdevel```: It sets dependencies for a PC with NVidia cards.
2. ```Dockerfile.nvdevel-cmake```: It installs other dependencies and CMake 3.24.3. 
3. ```Dockerfile.nvdevel-cmake-ompi```: It installs OpenMPI 4.1.1.

> The versions of CMake and OpenMPI from *apt-get* are too low for OSPRay Studio. That's why there are steps to build CMake and OpenMPI.

## Build Docker images
We create Docker images from the files.
```
docker build . -t gregabram/nvdevel -f Dockerfile.nvdevel

docker build . -t gregabram/nvdevel-cmake -f Dockerfile.nvdevel-cmake

docker build . -t gregabram/nvdevel-cmake-ompi -f Dockerfile.nvdevel-cmake-ompi
```

## Build OSPRay from the Docker container
This sub-section shows steps for building OSPRay from the container.

To set up, run the following commands:
```
# clone OSPRay (we will build this later from a container)
git clone git@github.com:jungwhonam-tacc/ospray.git
```

Let's run Docker with X enabled. 
```
docker run -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=host.docker.internal:0 -v /Users/jnam/Documents/2022/OSPRayIntegration/ospray:/home/ospray/ --rm -it gregabram/nvdevel-cmake-ompi 
```
> Replace ```/Users/jnam/Documents/2022/OSPRayIntegration/ospray``` with the folder that contains the cloned OSPRay.

> If you do not have X-Server installed, please see the next section.
> 
Build OSPRay from the container.
```
mkdir build-container
cd build-container
cmake ../scripts/superbuild -DCMAKE_BUILD_TYPE=Release -DBUILD_OSPRAY_MODULE_MPI=ON
cmake --build .
```

There are serveral example programs, which are located in ```build-container/install/ospray/bin```.
```
# non-GUI example
./install/ospray/bin/ospTutorial
# GUI example
./install/ospray/bin/ospExamples
# MPI GUI example
mpirun -n 2 ./install/ospray/bin/ospMPIDistribTutorialSpheres
```

## Build OSPRay Studio from the Docker container
Similar to what we have done for building OSPRay, we build the OSPRay Studio. The commands for CMake configuration and run are same as what we described in [Extending OSPRay Studio](../OSPRayStudio/OSPRayStudio.md). 
```
# clone OSPRay Studio
git clone git@github.com:jungwhonam-tacc/ospray_studio.git

# run a Docker container with X-enabled
docker run -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=host.docker.internal:0 -v /Users/jnam/Documents/2022/OSPRayIntegration/ospray_studio:/home/ospray_studio/ --rm -it gregabram/nvdevel-cmake-ompi 

# build OSPRay Studio
mkdir build-container
cd build-container
cmake -S .. -B . \
-DCMAKE_BUILD_TYPE=Release \
-DUSE_MPI=ON \
-Dospray_DIR=/Users/jnam/Documents/2022/OSPRayIntegration/ospray/build-container/install/ospray/lib/cmake/ospray-2.10.0 \
-DBUILD_PLUGINS=ON \
-DBUILD_PLUGIN_GESTURE=ON
cmake --build .

# run the application 
mpirun -n 3 \
./install/bin/ospStudio \
multiwindows \
--mpi \
--displayConfig config/display_settings.json \
--scene multilevel_hierarchy \
--plugin gesture \
--plugin:gesture:config config/tracking_settings.json
```

# Run the MPI-Application in multiple nodes
Assuming that the Docker image is pushed to a Docker Hub, we build an Apptainer image from the Docker image. The command below will create a file ```nvdevel-cmake-ompi.sif```.
```
apptainer build docker://gregabram/nvdevel-cmake-ompi
```

The command below runs the application on TACC's Rattler system. The python script ```run-all.py``` is provided to help with creating the command line.

```
mpirun -host localhost,r01,r02,r03,r04,r05,r06,r07,r08,r10,r11,r12,r13,r14,r15,r16,r17,r18,r19 \
apptainer run nvdevel-cmake-ompi.sif \
/bin/bash/ \
/home/jnam/ospray/wrapper.sh \
/home/jnam/GitHub/ospray_studio/build-container/install/bin/ospStudio \
multiwindows \
--mpi \
--displayConfig /home/jnam/GitHub/ospray_studio/build-container/install/config/display_settings.json \
--scene multilevel_hierarchy \
--plugin gesture \
--plugin:gesture:config /home/jnam/GitHub/ospray_studio/build-container/install/config/tracking_settings.json
```
- ```apptainer run nvdevel-cmake-ompi.sif``` runs the application inside the container
- ```/bin/bash/ /home/jnam/ospray/wrapper.sh``` adds OSPRay  PATH and LD_LIBRARY_PATH
- ```/home/jnam/GitHub/ospray_studio/build-container/install/bin/ospStudio...``` runs the application with the parameters. 
> We found that using relative paths in a command line produces errors. Thus, we suggest using absolute paths.

It is important to pass ```wrapper.sh``` when running the container as the script sets environment variables for other nodes.
```
# wrapper.sh
export PATH=/home/vislab/ospray/local/bin:$PATH 
export LD_LIBRARY_PATH=/home/vislab/ospray/local/lib:$LD_LIBRARY_PATH
```

# About ```X11```
```X11``` is a remote-display protocol by Linux/Unix machines. This is used to run GUI programs in a Docker container and show the programs in Windows/Mac (mainly used for development).

Linux has X11, which comes with default. But we need to install 3rd party software for Mac and Windows.

## Install X-Server on Windows (using VCXsrv)

Follow steps in [this tutorial](https://medium.com/javarevisited/using-wsl-2-with-x-server-linux-on-windows-a372263533c3). 

Or if you already installed WSL2 (type ```wsl``` on CMD to check).

1. Download and install [VCXsrv](https://sourceforge.net/projects/vcxsrv/).
2. Open CMD and type the following line to start the X-Server: ```"C:\Program Files\VcXsrv\vcxsrv.exe" :0 -ac -terminate -lesspointer -multiwindow -clipboard -wgl -dpi auto```

## Install X-Server on Mac (using XQuartz)
Download and install [XQuartz](https://www.xquartz.org/). 

However, you may run into these two errors:

```
xterm: Xt error: Can't open display: host.docker.internal:0
```

If you type ```xterm``` from the container and see this error, change the security settings for XQuartz.
   - Turn off _Authenticate connections_
   - Turn on _Allow connections form network clients_

```
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
error 65543: GLX: Failed to create context: BadValue (integer parameter out of range for operation)
terminate called after throwing an instance of 'std::runtime_error'
what():  Failed to create GLFW window!
```

When you run an application from the container and see these errors, you need to enable _iglx_ (see [Nicola's post](https://unix.stackexchange.com/questions/429760/opengl-rendering-with-x11-forwarding)).


# Future work
Since the project is under development, we have built the application after running a Docker container. In the future, in the deployment phase, we plan to create a release in our GitHub repo. A Docker image will then download the repo and install our OSPRay and OSPRay Studio versions. This Dockerfile below is an example demonstrating how this could work out. 

## legacy/Dockerfile.ospray

```Dockerfile.ospray``` downloads an OSPRay release and builds it from the environment set by ```Dockerfile```. Currently, it downloads a pre-release version that our team at TACC is working on. 

To build the specific docker file run the following command:

```
docker build . -t jungwhonam/ospray-multidisplays:0.2 -f Dockerfile.ospray
```

Docker can be used in different ways. 

> If X is not needed, remove the following line: ```-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=host.docker.internal:0```.

> When running an example program without specifying an interactive ```-it``` run, give *absolute* paths to configuration files. This applies for both ```display_settings.json``` and ```tracking_settings.json``` files.

Run Docker interactively.
```
docker run \
-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=host.docker.internal:0 \
--rm -it  jungwhonam/ospray-multidisplays:0.2
```

Run an example, ospMPIMultiDisplays, without MPI.
```
docker run \
-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=host.docker.internal:0 \
--rm -it  jungwhonam/ospray-multidisplays:0.2 \
/home/ospray/ospray-2.11.0-alpha.1/build/install/ospray/bin/ospMPIMultiDisplays \
/home/ospray/ospray-2.11.0-alpha.1/build/install/ospray/bin/config/display_settings.json \
/home/ospray/ospray-2.11.0-alpha.1/build/install/ospray/bin/config/tracking_settings.json
```

Run an example, ospMPIMultiDisplays, with MPI.
```
docker run \
-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=host.docker.internal:0 \
--rm -it  jungwhonam/ospray-multidisplays:0.2 \
mpirun -n 3 /home/ospray/ospray-2.11.0-alpha.1/build/install/ospray/bin/ospMPIMultiDisplays \
/home/ospray/ospray-2.11.0-alpha.1/build/install/ospray/bin/config/display_settings.json \
/home/ospray/ospray-2.11.0-alpha.1/build/install/ospray/bin/config/tracking_settings.json
```