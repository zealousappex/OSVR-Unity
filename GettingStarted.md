# OSVR for Unity Developers

> Last Updated January 14, 2016

## Why use OSVR?

Open-source Virtual Reality (OSVR) is an open-source software platform for VR/AR applications. It provides abstraction layers for VR devices and peripherals so that a game developer does not need to hardcode support for particular hardware. 
OSVR provides interfaces – pipes of data – as opposed to an API tied to a specific piece of hardware. If you want hand position, for example, OSVR will give you hand position regardless of whether the data comes from a Microsoft Kinect, Razer Hydra, or Leap Motion. 
Game developers can focus on what they want to do with the data, as opposed to how to obtain it.

If you’re not convinced yet, or would like to read a more complete introduction to OSVR, please visit: http://osvr.github.io/whitepapers/introduction_to_osvr/

The rest of this document assumes you generally understand the benefits of OSVR and would like to know how to start using it. Before we get into the Unity plugin, let’s make sure OSVR is up and running.

## OSVR-Core and OSVR Server

When using OSVR, you need to run OSVR server. Download the latest OSVR-Core binary snapshot from: http://osvr.github.io/using/

Note that the Unity Asset Store version may be slightly behind the Github version, but that currently, the Unity Asset Store version contains additional DLLs for RenderManager that are not contained in the OSVR-Unity.unitypackage distributed on Github (these DLLs are available in the OSVR-Unity examples project, however).

Let’s examine the /bin directory of an extracted OSVR-Core binary snahpshot:
(picture)

osvr_server.exe will launch the OSVR Server. By default, it uses osvr_server_config.json as a configuration file. Since the configuration file is crucial to enabling functionality in OSVR, we will explore it in more detail later.

First, launch osvr_server.exe, if you see this message:
(picture)

Then no hardware has been discovered. This document will assume you’re using an HDK. If you need help with HDK setup, please visit the HDK Unboxing and Starting Guide: https://github.com/OSVR/OSVR-General/wiki/HDK-Unboxing-and-Getting-Started

This message shows that the server has found an HDK:
(picture)

## Tracker View

The Tracker View utility (avaialable from http://osvr.github.io/using/) is helpful for quickly checking if tracking is working as expected. It shows the position and orientation of tracked objects. Here we see the orientation of the HDK without positional tracking. Z (blue) points towards the back of the head, Y (green) up, and X (red) to the right.
(picture)

## Getting Started with OSVR-Unity
Let’s examine the OSVR-Unity plugin. 
- Download the latest Unity snapshot build from http://osvr.github.io/build-with/.
- Create a new Unity 5.3 project.
- Import OSVR-Unity.unitypackage into the project.
- Open VRFirstPerson.unity scene.

This scene demonstrates a first-person controller VR setup. Let’s examine some of the objects in the scene hierarchy:

### ClientKit
The ClientKit object communicates with OSVR Server and must be in every scene. It requires an app ID, which can be any string identifier.
(picture)

### VRDisplayTracked
This prefab contains a DisplayController component, which constructs our tracked camera rig at runtime.
(picture)

Press Play. The stereo camera rig is created at runtime and updates its orientation as the HDK moves around. With my HDK connected as an extended display, I can click-and-drag the Game View to the HDK display and select Maximize on Play to view the game in VR.
(picture)

### Recenter
(picture)
The SetRoomRotationUsingHead script “recenters” the room when a key is pressed (“R” by default in this sample), so that your head is facing the forward direction. There is another key (“U” by default in this sample) for undoing that rotation.

### VRFirstPersonController
(picture)
This prefab provides very similar controls to the Unity first-person controller scripts. Mouse-look can be controlled with the “M” key, by default.

That covers the basics! There are examples of tracked controllers, eyetrackers, and locomotion in other example scenes (see TrackerView.unity). If you want to enable features like positional tracking and Direct Mode rendering, that all happens in the server configuration file.

## OSVR Server Configuration - Adding Positional Tracking and RenderManager 
Want to enable positional tracking? Enable DirectMode rendering support? Use a device plugin? All of this requires some setup in your server configuration file. Many sample configuration files are provided in the sample-configs directory of the Core Snapshot. Some devices, such as the Hacker Development Kit (HDK) and Razer Hydra can be auto-detected and will work with a default (empty) config file.

osvr_server.exe can be run from the command line with an optional parameter for the server config file. If no parameter is specified, osvr_server_config.json will be used instead. Since the HDK is detected automatically, we don’t need to edit osvr_server_config.json in order to obtain tracking data from the HDK. If we want to enable positional tracking, however, we’ll need to edit the config file.

Copy the contents of osvr_server_config.HDK13DirectMode.sample.json into osvr_server_config.json.
(picture)

Let’s examine our new osvr_server_config.json:
(picutre)
These two lines determine which HMD display to use, and which RenderManager config file to use. If you didn’t want positional tracking, these are the only two lines you need to enable RenderManager with the HDK 1.3.
(picture)
The rest of the file deals with video-based tracking and sensor fusion. There are options for showing a camera debug view (impacts game performance if this is running in the background), enabling or disabling the LEDs located on the back of the head-strap, changing head circumference, and selecting a calibration file.
(picture)

## Positional Tracking - Calibration
You’ll want to calibrate the camera before using positional tracking, and every time the camera moves. For a full guide on calibration, visit: https://github.com/OSVR/OSVR-Core/wiki/Video-Based-Tracking-Calibration

### Tracker View - Positional Tracking
Let’s run the server and check Tracker View again to see if we’re getting positional tracking.
(picture)
Yes! The gizmo is no longer fixed at the origin, and moves around with the HMD. 

## RenderManager
For a more information about RenderManager, including an overview of configuration options, visit: 
(link)

### Enabling RenderManager in Unity
The following additional DLLs are required in Assets/Plugins/x86_64:
- glew32.dll
- SDL2.dll
- osvrUnityRenderingPlugin.dll -- built from https://github.com/OSVR/OSVR-Unity-Rendering
- osvrRenderManager.dll

The easiest way to obtain working binaries is to copy them over from one of the example projects Assets/Plugins/x86_64 folders:
https://github.com/OSVR/Unity-VR-Samples/tree/master/Assets/Plugins/x86_64
https://github.com/OSVR/OSVR-Unity-Palace-Demo/tree/master/Assets/Plugins/x86_64

The OSVR-Unity Asset Store package also contains these DLLs, but they may be more out-of-date than on Github.

Going back to our original Unity project, if we update our Plugins folder with the DLLs, make sure the server is running, and press play, we are now in DirectMode with positional tracking! 

### Direct Mode Preview
You can see the game view mirrors the HDK display because the “Show Direct Mode Preview” option is checked on the VRDisplaytTracked prefab. It is a rather suboptimal implementation of mirror mode, and will soon be replaced by an equivalent RenderManager feature.
(picutre)

### Optimizing RenderManager in Unity
There are a number of configuration options available when using RenderManager. We’ve found that settings may vary per application, so developers should experiment until an acceptable balance of visual quality and fast rendering is achieved. Generally, the configuration below is a good starting point. See the RenderManager document for more information: 
(link)
(picture)

### Troubleshooting RenderManager in Unity
If you see a “failed to load RenderManager” message print to the Unity debug console or output log, try running the RenderManager example programs from a command line to see the output.

If you get an error that says something like “Could not open display (are you whitelisted?)” Try running the whitelist_OSVR.reg file for HDK 1.3, whitelist_OSVRPre.reg for HDK 1.2 or earlier, or whitelist_Vuzix.reg for Vuzix devices, then restart the computer.

If you see this error, then the server isn’t configured for RenderManager.
(picture)
If the example programs work in direct mode, but Unity still fails to load RenderManager, make sure all DLLs are up to date. It is best to close Unity, replace the DLLs in Windows Explorer, and re-open Unity so that the new libraries are loaded.

## Quality Settings
Quality settings will differ per machine/graphics card capabilities. Make sure V Sync Count is set to Don’t Sync. Otherwise, we generally recommend that using as much antialiasing as can be afforded without negatively affecting performance. Disable shadows unless your game mechanics demand it. Use Lightmapping instead of real-time lights. Use occlusion culling.
(picture)

## Player Settings
Generally, use Static and Dynamic batching to reduce draw calls. Note that the “Virtual Reality Supported” checkbox does not need to be checked, since Unity currently only supports Oculus and GearVR with this feature. Leaving it checked should not impact OSVR performance, but you will see a “[VRDevice] Initialization of device oculus failed” message that can be ignored.
(picture)

## Building for Android
Building an OSVR Android app requires libraries from https://github.com/OSVR/OSVR-Android-SDK.

Copy the .so files to the Assets/Plugins/libs/armeabi-v7a folder:
(picture)
Make sure your project is set to build for Android:
(picture)
In this case, OSVR server is built into the app and should run automatically at startup.