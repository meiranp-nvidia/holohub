# Medical Image Viewer in XR

![Volume Rendering Screenshot](../volume_rendering/screenshot.png)

## Description

We collaborated with Magic Leap on a proof of concept mixed reality viewer for medical imagery built on the Holoscan platform.

Medical imagery is one of the fastest-growing sources of data in any industry. When we think about typical diagnostic imaging, X-ray, CT scans, and MRIs come to mind. X-rays are 2D images, so viewing them on a lightbox or, if they’re digital, a computer, is fine. But CT scans and MRIs are 3D. They’re incredibly important technologies, but our way of interacting with them is flawed. This technology helps physicians in so many ways, from training and education to making more accurate diagnoses and ultimately to planning and even delivering more effective treatments.

You can use this viewer to visualize a segmented medical volume with a mixed reality device.

## Prerequisites

### Host Machine

Review the [HoloHub README document](/README.md#prerequisites) for supported platforms and software requirements.

#### Magic Leap 2 Device

The following packages and applications are required to run remote rendering with a Magic Leap 2 device:

| Requirement | Platform | Version | Source |
|--|------|---------|--|
| Magic Leap Hub | Windows or macOS PC | latest | [Magic Leap Website](https://ml2-developer.magicleap.com/downloads) |
| Headset Firmware | Magic Leap 2 | v1.5.0 | Magic Leap Hub |
| Headset Remote Rendering Viewer (.apk) | Magic Leap 2 | [1.9.192](https://thelab.magicleap.cloud/packages_mlhub/artifacts/com.magicleap.remote_render/1.9.192/ml_remote_viewer.apk) | Magic Leap Download Link |
| Windrunner OpenXR Backend | HoloHub Container | 1.9.194 | Included in Container |
| Magic Leap 2 Pro License | | | Magic Leap |

Refer to the Magic Leap 2 documentation for more information:
- [Updating your device with Magic Leap Hub](https://www.magicleap.care/hc/en-us/articles/5341445649805-Updating-Your-Device);
- [Installing `.apk` packages with Magic Leap Hub](https://developer-docs.magicleap.cloud/docs/guides/developer-tools/ml-hub/ml-hub-package-manager/)

## Building the Application

Run the following commands to build and launch the HoloHub custom development container:
```bash
./dev_container build --img holohub:openxr-base --base_img nvcr.io/nvidia/clara-holoscan/holoscan:v0.6.0-dgpu --docker_file ./applications/volume_rendering_xr/Dockerfile.base # build the base container
./dev_container build --img holohub:openxr-dev --base_img holohub:openxr-base --docker_file ./applications/volume_rendering_xr/Dockerfile.dev # build the dev container
./dev_container launch --as_root --img holohub:openxr-dev -c "./run build volume_rendering_xr cpp" # Build the application
./dev_container launch --as_root --img holohub:openxr-dev # Run the container interactively
```

When the container is launched a QR code will be visible in the console log. Refer to Magic Leap 2 [Remote Rendering Setup documentation](https://developer-docs.magicleap.cloud/docs/guides/remote-rendering/remote-rendering/#:~:text=Put%20on%20the%20Magic%20Leap,headset%20by%20looking%20at%20it.&text=The%20QR%20code%20launches%20a,Click%20Continue.) to pair the host and device in preparation for remote viewing. Refer to the [Remote Viewer](#starting-the-magic-leap-2-remote-viewer) section to regenerate the QR code as needed.

## Running the Application

Run the following command inside the development container to start the XR volume rendering application:
```bash
./run launch volume_rendering_xr
```

## Additional Notes

### Supported Formats

The following medical dataset formats are supported:
* [MHD](https://itk.org/Wiki/ITK/MetaIO/Documentation)
* [NIFTI](https://nifti.nimh.nih.gov/)
* [NRRD](https://teem.sourceforge.net/nrrd/format.html)

### Launch Options

Use the `--extra-args` to see all options, including how to specify a different dataset or transfer function to use.
```bash
./run launch volume_rendering_xr --extra_args --help
...
Holoscan OpenXR volume renderer.Usage: /workspace/holohub/build/applications/volume_rendering_xr/volume_rendering_xr [options]
Options:
  -h, --help                            Display this information
  -c <FILENAME>, --config <FILENAME>    Name of the renderer JSON configuration file to load (default '/workspace/holoscan-openxr/data/volume_rendering/config.json')
  -d <FILENAME>, --density <FILENAME>   Name of density volume file to load (default '/workspace/holoscan-openxr/data/volume_rendering/highResCT.mhd')
  -m <FILENAME>, --mask <FILENAME>      Name of mask volume file to load (default '/workspace/holoscan-openxr/data/volume_rendering/smoothmasks.seg.mhd')
```

To use a new dataset with the application, mount its volume location from the host machine when launching the container and pass all required arguments explicitly to the executable:
```bash
./dev_container launch --as_root --img holohub:openxr-dev --add-volume /host/path/to/data-dir
>>> ./build/applications/volume_rendering_xr/volume_rendering_xr \
      -c /workspace/holohub/data/volume_rendering/config.json \
      -d /workspace/volumes/path/to/data-dir/dataset.nii.gz \
      -m /workspace/volumes/path/to/data-dir/dataset.seg.nii.gz
```

### Starting the Magic Leap OpenXR runtime

OpenXR runtimes are implementations of the OpenXR API that allow the Holoscan XR operators to create XR sessions and render content. The Magic Leap OpenXR runtime including a CLI are by default installed in the dev container. __From a terminal inside the dev container__ you can execute the following scripts:

```
ml_start.sh
```
starts the OpenXR runtime service. After executing this command, the remote viewer on the Magic Leap device should connect to this runtime service. If not, then you still have to pair the device with the host computer running the Holoscan application.

For rapid iteration without a Magic Leap device, pass the argument `debug` to `ml_start.sh` i.e.
```
ml_start.sh debug
```
This will enable a debug view on your computer showing what the headset would see. You may click into this window and navigate with the keyboard and mouse to manipulate the virtual head position.

If you connect an ML2 while the debug view is active, you can continue to view the content on the debug view but can no longer adjust the virtual position, as the real position is used instead.

```
ml_pair.sh
```
displays a QR code used to pair the device with the host. Start the QR code reader App on the device and scan the QR code displayed in the terminal. Note that the OpenXR runtime has to have been started using the __ml_start__ command in order for the paring script to execute correctly.

```
ml_stop.sh
```
stops the OpenXR runtime service.

### Starting the Magic Leap 2 Remote Viewer

When using a Magic Leap 2 device for the first time or after a software upgrade, the device must be provided with the IP address of the host running the OpenXR runtime. From a terminal inside the dev container run the

```
ml_pair.sh
```

command, which will bring up a QR code that has to be scanned using the __QR Code App__ on the Magic Leap 2 device. Once paired with the host, the device  will automatically start the remote viewer which will then prompt you to start an OpenXR application on the host. Any time thereafter, start the remote viewer via the App menu.

### Developing with a Different OpenXR Backend

The Magic Leap Remote OpenXR runtime (`windrunner`) is the default configured by
the dev container. Other backends such as Monado are also available.

Inside the container, set openxr_monado.json as the active runtime:

```bash
/workspace/holohub$ update-alternatives --config openxr1-active-runtime

There are 2 choices for the alternative openxr1-active-runtime (providing /etc/xdg/openxr/1/active_runtime.json).

  Selection    Path                                                   Priority   Status
------------------------------------------------------------
  0            /opt/windrunner/lib/windrunner/openxr_windrunner.json   60        auto mode
  1            /opt/windrunner/lib/windrunner/openxr_windrunner.json   60        manual mode
* 2            /usr/share/openxr/1/openxr_monado.json                  50        manual mode
```

Then start the service to run quietly in the background:

```bash
/workspace/holohub$ monado-service > /dev/null 2>&1 &
```

NOTE: If you switch back to the Magic Leap runtime, don't forget to update the
active runtime alternative again with update-alternatives (above).
