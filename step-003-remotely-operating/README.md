# Remotely operating your solution <!-- omit in toc -->

In the previous step, we have provisioned our Jetson Nano device running Azure IoT Edge to IoT Central. Now, to demonstrate how Fabrikam can manage their solution fully remotely, we willl be remotely reconfiguring the video input streams used by the computer vision application. You will be setting up your smartphone as an RTSP camera, and use it as a new input camera.

## Learning goals <!-- omit in toc -->

- How to use IoT Central to remotely configure an IoT Edge module.
- How to visualize device/module telemetry in IoT Central.

## Steps <!-- omit in toc -->

- [Configuring and testing your smartphone as an IP camera](#configuring-and-testing-your-smartphone-as-an-ip-camera)
- [Changing input cameras](#changing-input-cameras)

### Configuring and testing your smartphone as an IP camera

Let's first verify that your phone works as an RTSP camera properly:

- Open the the IP Camera Lite application on your smartphone.
- Go to Settings and remove the User and Password on the RTSP feed.
- Click on `Turn on IP Camera Server`.

You can now open VLC and check that the video stream can be accessed:

- Go to `Media` > `Open Network Stream`
- Paste the following `RTSP Video URL`:  `rtsp://your-phone-ip-address:8554/live`
- Click `Play` and verify that phone's camera is properly displaying.

### Changing input cameras

Let's now update your Jetson Nano to use your phone's camera. In IoT Central:

- Go to the `Manage` tab
- Unselect the `Demo Mode`, which uses several hard-coded video files as input of car traffic
- Update the `Video Stream 1` property:
    - In the `cameraId`, name your camera, for instance `My Phone`
    - In the `videoStreamUrl`, enter the RTSP stream of this camera: `rtsp://your-phone-ip-address:8554/live`
- Keep the default AI model of DeepStream by keeping the value `DeepStream ResNet 10` as the `AI model type`.
- Keep the default `Secondary Detection Class` as `person`
- Hit `Save`

This sends a command to the device to update its DeepStream configuration file with these new properties. If you were still streaming the output of the DeepStream application, this stream will be taken down as DeepStream will restart.

While Deepstream restarts, you can have a closer look at the DeepStream configuration file to see what has changed. From your SSH client:

1. Open an SSH connection with your Jetson Nano IP address. Username is `dlinano` and so is the default password.

    ```bash
    ssh dlinano@YOUR_IP_ADDRESS
    ```

2. Open up the default configuration file of DeepStream to understand its structure:

    ```bash
    nano /data/misc/storage/DSconfig.txt
    ```

3. Look for the first `[source0]`, entry and observe how parameters provided in IoT Central UI got copied here.

Within a minute, DeepStream should restart. You can observe its status in IoT Central via the `Modules` tab. Once `deepstream` module is back to `Running`, copy again the `RTSP Video Url` field from the `Device` tab and give it to VLC (`Media` > `Open Network Stream` > paste the `RTSP Video URL` > `Play`).

You should now detect people from your phone's camera! The count of `Person` in the `dashboard` tab in IoT Central should go up. We've just remotely updated the configuration of this intelligent video analytics solution!

## Going further <!-- omit in toc -->

### Understanding NVIDIA DeepStream <!-- omit in toc -->

Deesptream is an SDK based on [GStreamer](https://gstreamer.freedesktop.org/). It is very modular, and has a plug-in based architecture. Each plug-in have `sinks` (outputs) and `sources` (inputs). As part of Deepstream, NVIDIA provides several plug-ins which are optimized for leveraging NVIDIA's GPUs. How these plug-ins are connected with one another is defined in the application's configuration file.

You can learn more about Deepstream's architecture in [NVIDIA's official documentation](https://docs.nvidia.com/metropolis/deepstream/dev-guide/index.html#page/DeepStream_Development_Guide%2Fdeepstream_app_architecture.html) (sneak peek below).

![NVIDIA Deepstream Application Architecture](./assets/DeepStreamArchitecture.png).

To better understand how NVIDIA DeepStream works, let's have another look at a typical configuration file, similar to the one automatically generated in `/data/misc/storage/DSconfig.txt` as soon as you reconfigure your module from IoT Central

```properties
# Copyright (c) 2018 NVIDIA Corporation.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.

[application]
enable-perf-measurement=1
perf-measurement-interval-sec=5
#gie-kitti-output-dir=streamscl

[tiled-display]
enable=1
rows=2
columns=2
width=1280
height=720
gpu-id=0
#(0): nvbuf-mem-default - Default memory allocated, specific to particular platform
#(1): nvbuf-mem-cuda-pinned - Allocate Pinned/Host cuda memory, applicable for Tesla
#(2): nvbuf-mem-cuda-device - Allocate Device cuda memory, applicable for Tesla
#(3): nvbuf-mem-cuda-unified - Allocate Unified cuda memory, applicable for Tesla
#(4): nvbuf-mem-surface-array - Allocate Surface Array memory, applicable for Jetson
nvbuf-memory-type=0

[source0]
enable=1
#Type - 1=CameraV4L2 2=URI 3=MultiURI 4=RTSP
type=3
uri=file:///data/misc/storage/sampleStreams/cam00-na.mp4
num-sources=1
gpu-id=0
# (0): memtype_device   - Memory type Device
# (1): memtype_pinned   - Memory type Host Pinned
# (2): memtype_unified  - Memory type Unified
cudadec-memtype=0

[source1]
enable=1
#Type - 1=CameraV4L2 2=URI 3=MultiURI 4=RTSP
type=3
uri=file:///data/misc/storage/sampleStreams/cam01-na.mp4
num-sources=1
gpu-id=0
# (0): memtype_device   - Memory type Device
# (1): memtype_pinned   - Memory type Host Pinned
# (2): memtype_unified  - Memory type Unified
cudadec-memtype=0

[source2]
enable=1
#Type - 1=CameraV4L2 2=URI 3=MultiURI 4=RTSP
type=3
uri=file:///data/misc/storage/sampleStreams/cam02-na.mp4
num-sources=1
gpu-id=0
# (0): memtype_device   - Memory type Device
# (1): memtype_pinned   - Memory type Host Pinned
# (2): memtype_unified  - Memory type Unified
cudadec-memtype=0

[source3]
enable=1
#Type - 1=CameraV4L2 2=URI 3=MultiURI 4=RTSP
type=3
uri=file:///data/misc/storage/sampleStreams/cam03-na.mp4
num-sources=1
gpu-id=0
# (0): memtype_device   - Memory type Device
# (1): memtype_pinned   - Memory type Host Pinned
# (2): memtype_unified  - Memory type Unified
cudadec-memtype=0

[sink0]
enable=1
#Type - 1=FakeSink 2=EglSink 3=File
type=1
sync=1
source-id=0
gpu-id=0
qos=0
nvbuf-memory-type=0
overlay-id=1

[sink1]
enable=1
#Type - 1=FakeSink 2=EglSink 3=File 4=UDPSink 5=nvoverlaysink 6=MsgConvBroker
type=6
msg-conv-config=/data/misc/storage/ResNetSetup/configs/msgconv_config.txt
#(0): PAYLOAD_DEEPSTREAM - Deepstream schema payload
#(1): PAYLOAD_DEEPSTREAM_MINIMAL - Deepstream schema payload minimal
#(256): PAYLOAD_RESERVED - Reserved type
#(257): PAYLOAD_CUSTOM   - Custom schema payload
msg-conv-payload-type=1
msg-broker-proto-lib=/opt/nvidia/deepstream/deepstream-4.0/lib/libnvds_azure_edge_proto.so
topic=mytopic
#Optional:
#msg-broker-config=../../../../libs/azure_protocol_adaptor/module_client/cfg_azure.txt

[sink2]
enable=0
type=3
#1=mp4 2=mkv
container=1
#1=h264 2=h265
codec=1
sync=0
#iframeinterval=10
bitrate=2000000
output-file=out.mp4
source-id=0

[sink3]
enable=1
#Type - 1=FakeSink 2=EglSink 3=File 4=RTSPStreaming
type=4
#1=h264 2=h265
codec=1
sync=0
bitrate=4000000
# set below properties in case of RTSPStreaming
rtsp-port=8554
udp-port=5400

[osd]
enable=1
gpu-id=0
border-width=1
text-size=15
text-color=1;1;1;1;
text-bg-color=0.3;0.3;0.3;1
font=Serif
show-clock=0
clock-x-offset=800
clock-y-offset=820
clock-text-size=12
clock-color=1;0;0;0
nvbuf-memory-type=0

[streammux]
gpu-id=0
##Boolean property to inform muxer that sources are live
live-source=0
batch-size=4
##time out in usec, to wait after the first buffer is available
##to push the batch even if the complete batch is not formed
batched-push-timeout=40000
## Set muxer output width and height
width=1920
height=1080
##Enable to maintain aspect ratio wrt source, and allow black borders, works
##along with width, height properties
enable-padding=0
nvbuf-memory-type=0

# config-file property is mandatory for any gie section.
# Other properties are optional and if set will override the properties set in
# the infer config file.
[primary-gie]
enable=1
gpu-id=0
model-engine-file=/data/misc/storage/ResNetSetup/detector/resnet10.caffemodel_b4_fp16.engine
batch-size=4
#Required by the app for OSD, not a plugin property
bbox-border-color0=1;0;0;1
bbox-border-color1=0;1;1;1
bbox-border-color2=0;0;1;1
bbox-border-color3=0;1;0;1
interval=4
gie-unique-id=1
nvbuf-memory-type=0
config-file=/data/misc/storage/ResNetSetup/configs/config_infer_resnet4_nano.txt

[tracker]
enable=1
tracker-width=480
tracker-height=272
ll-lib-file=/opt/nvidia/deepstream/deepstream-4.0/lib/libnvds_mot_klt.so
#ll-config-file required for DCF/IOU only
#ll-config-file=iou_config.txt
gpu-id=0

[tests]
file-loop=1
```

Observe in particular:

- The `source` sections: they define where the source videos are coming from. We're using local videos to begin with and will switch to live RTSP streams later on.
- The `sink` sections: they define where to output of the processed videos and the output messages. We use RTSP to stream a video feed out and all out messages are sent to the Azure IoT Edge runtime.
- The `primary-gie` section: it defines which AI model is used to detect objects. It also defines how this AI model is applied. As an example, note the `interval` property set to `4`: this means that inferencing is actually executed only once every 5 frames. Bounding boxes are displayed continuously though because a tracking algorithm, which is computationally less expensive than inferencing, takes over in between. The tracking algorithm used is set in the `tracker` section. This is the kind of out-of-the-box optimizations provided by DeepStream that enables us to process 240 frames per second on a $100 device. Other notable optimizations are using hardware decoders, doing zero in-memory copy, pushing the vast majority of the processing to the GPU, batching frames from multiple streams, etc.

## Wrap-up and Next steps <!-- omit in toc -->

If you've reached this point, you should now be familiar with the remote operation of a device running Azure IoT Edge from IoT Central.

In the [next section](../step-004-train-and-deploy-vision-ai/) we will be going further by training our own computer vision model—this time targeted at soda cans!— and remotely deploying it to our Jetson Nano.
