---
title: 'TideSense Quick Start: SD Card, RNDIS and VIO'
description: Flash the TideSense SD card image, connect over USB RNDIS, focus the stereo cameras in TideConfigurator and start VIO, with Windows 10 and 11 driver instructions.
---

# Quick Start

After completing component assembly and soldering, install the lens holders and lenses.

1. Download the latest SD card release files and flash them to an SD card: [TideSense releases](https://github.com/HumpbackLab/tidesense/releases/latest).
2. Connect the device to your computer with a USB cable. It should appear as an RNDIS device. If it does not, see the appendix below.
3. Download the [TideSense source](https://github.com/HumpbackLab/tidesense) and set up the development environment. Using Podman or Docker with the published TideSense development image is strongly recommended to keep the environment consistent. See [Development Environment Setup](development-environment.md).
4. Build and run [TideConfigurator](https://github.com/HumpbackLab/tide_configurator).
5. Open the TideConfigurator web interface and connect to the device at `192.168.123.99`, using username `root` and password `tidesense`.
6. On the Camera Calibration page, select the lens mode (Fisheye) and click Start Preview. Images from both cameras should appear.
7. Turn the lenses while watching the preview to adjust the focus. Once both views are sharp, secure each lens with its locking ring to prevent it from loosening.
8. Open the VIO Deployment page and click Start VIO. The device will run SchurVINS using the pre-calibrated files on the SD card. Differences in lens installation and focus between devices can cause the estimate to diverge at this stage; each device needs its own [calibration](calibration.md). For this initial check, confirm that a VIO trajectory appears in the web interface.

## Appendix: Installing the RNDIS Driver on Windows 10/11

### Manually Select the RNDIS Driver

If the device appears under Other devices with a yellow warning icon, or is identified as an unknown USB device, you can manually select the RNDIS driver included with Windows:

1. Keep TideSense connected to your computer. Press `Win + X` and open Device Manager.
2. Find the TideSense device under Other devices or Network adapters. If you are unsure which entry it is, unplug and reconnect the USB cable and watch which entry disappears and reappears.
3. Right-click the device and select Update driver.
4. Select Browse my computer for drivers.
5. Select Let me pick from a list of available drivers on my computer.
6. If Windows asks you to select a device type, choose Network adapters.
7. Select Microsoft as the manufacturer, then choose Remote NDIS Compatible Device or USB RNDIS Adapter as the model and click Next.
8. If a compatibility warning appears, confirm that you selected an RNDIS network adapter before continuing with installation.
9. Once installation finishes, the device should appear under Network adapters. Unplug and reconnect the USB cable once, then wait for the network connection to return.
