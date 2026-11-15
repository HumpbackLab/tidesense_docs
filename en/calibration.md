---
title: TideSense Stereo Camera and IMU Calibration Guide
description: Calibrate TideSense stereo cameras and the SC7U22 IMU with TideConfigurator, review calibration reports and apply the generated SchurVINS configuration to your device.
---

# Stereo and IMU Calibration

This page explains how to use TideConfigurator to calibrate the TideSense stereo cameras and onboard SC7U22 IMU, then apply the results to SchurVINS.

A single motion recording is used first for stereo calibration and then for joint camera–IMU calibration. The process estimates camera intrinsics and distortion, extrinsics between the left and right cameras, camera–IMU extrinsics and time offsets, and IMU parameters such as scale factors and axis misalignment. The final output is a ready-to-use `schurvins.yaml`.

Lens installation, focus and assembly vary between devices, so calibrating each device after assembly is recommended. Recalibrate after replacing lenses, refocusing, changing camera mounting positions, or changing the image resolution or cropping settings.

## 1. Prepare the Device and Calibration Environment

Follow the [Quick Start](quick-tutorial.md) to connect the device and focus the lenses. Secure the lenses so they cannot loosen during recording. Prepare a ruler marked in millimeters and a flat-panel monitor that can display the calibration target clearly.

Open <http://127.0.0.1:8765> and connect to the device:

| Item | Setting |
| --- | --- |
| Board type | TideSense |
| SSH address | `192.168.123.99` |
| Username | `root` |
| Password | `tidesense` |

## 2. Select the Lens Type and Capture Dimensions

Open the Camera Calibration page and select the installed lens type under Calibration Settings:

| Lens mode | Distortion model | When to select it |
| --- | --- | --- |
| Wide-angle | Radtan (`pinhole-radtan`) | For standard or wide-angle lenses |
| Fisheye | Equidistant (`pinhole-equi`) | For fisheye lenses |

If you are using the fisheye lenses described in the Quick Start, select Fisheye here. The interface defaults to Wide-angle, so check the selection again before starting calibration.

Click Start Preview and confirm that both cameras produce images and the calibration target details are clear. The right camera on TideSense is mounted upside down, rotated by 180°, so it is normal for the right preview image to have a different orientation from the left. Calibration uses the raw images, with the mounting orientation represented by the extrinsics. Keep the image handling consistent between recording and operation.

Record under bright, even lighting, avoiding reflections and overexposure. The automatic exposure limit defaults to 500 µs. If the image is too dark, improve the lighting first, then adjust the exposure limit while checking image sharpness. Before changing exposure settings, stop capture and VIO. Click Save, then restart the preview. The saved settings also apply to subsequent recording and VIO sessions.

## 3. Display and Measure the Calibration Target

1. Secure the monitor and open the On-screen Calibration Target. If you want to use Full Screen, enter full-screen mode before measuring.
2. Use the ruler to measure the horizontal distance **from the outer left edge of the small black square at the left end to the outer right edge of the small black square at the right end**.
3. Enter the measurement in **millimeters (mm)** as the Measured Span in the target measurement controls.
4. After measuring, keep the browser zoom, window size and displayed target size unchanged.

## 4. Record Calibration Data

**Keep the calibration target stationary and move the entire TideSense device.** Moving only the target does not provide the device motion required for joint camera–IMU calibration.

Start with a 120-second recording. Alternatively, enter `0` for Recording Duration and click Stop Recording manually. A duration of 60–120 seconds is a useful starting point; assess whether the data is sufficient from image coverage, motion excitation and the calibration reports.

Before starting, use the stereo preview to confirm that the target appears large enough in both camera views. For 640 × 400 capture, try an initial distance of approximately 40–60 cm, then adjust according to the actual target size, focus and shared field of view of the two cameras.

After clicking Start Recording, combine the following movements, keeping them smooth to avoid blur:

- Rotate around each axis.
- Move forward and backward.
- Move the target's image across the center and edges of the fisheye field of view.
- Move the device slowly in a figure-eight pattern.

## 5. Import IMU Noise Parameters

Under Prepare Calibration, click Import IMU Noise Parameters and select a YAML file. These are the IMU's statistical noise parameters, used to weight the inertial observations during calibration.

The file must contain the following fields, each with a finite value greater than zero:

| Field | Meaning | Unit |
| --- | --- | --- |
| `update_rate` | Actual sampling rate; should be close to 200 Hz | Hz |
| `gyroscope_noise_density` | Gyroscope white noise density | rad/s/√Hz |
| `gyroscope_random_walk` | Gyroscope bias random walk | rad/s²/√Hz |
| `accelerometer_noise_density` | Accelerometer white noise density | m/s²/√Hz |
| `accelerometer_random_walk` | Accelerometer bias random walk | m/s³/√Hz |

The TideConfigurator repository includes [`calibration/sc7u22.yaml`](https://github.com/HumpbackLab/tide_configurator/blob/master/calibration/sc7u22.yaml), containing Allan estimates from an existing SC7U22 recording. You can use it for an initial check of the workflow.

## 6. Start Joint Calibration

Click Start Calibration. The interface opens Calibration History and proceeds through data preparation, stereo calibration, joint camera–IMU calibration and a results summary. Both calibration stages use the same recording, so no second capture is needed.

Joint calibration uses the `scale-misalignment` IMU model.

Keep the local TideConfigurator service running throughout calibration. Click View Logs to check progress. If a job fails, retain the logs, correct the inputs or record new data according to the cause, then create a new job.

## 7. Review the Calibration Results

After completion, the job card displays the Measured Stereo Baseline, Left Camera Time Offset and configuration SHA-256. It also provides a Stereo Report, Camera–IMU Report and Download VIO Configuration.

**Calibration Complete means that computation and configuration export succeeded; the quality is still Pending Review.** Review both PDF reports first:

| Check | What to look for |
| --- | --- |
| Corner coverage | Both cameras should have enough observations covering the edges and all four corners, rather than only the center. |
| Reprojection error | Check both cameras, the error distribution and outliers. As a general guideline, reprojection error should be below 0.5 px. |
| Inertial residuals | Look for obvious anomalies in gyroscope and accelerometer residuals, and check that the motion covered every axis. |
| Stereo baseline | TideSense has a designed baseline of 50 mm. If the result differs substantially, first check the measured target span and image quality. |
| Time offsets | Check for obvious anomalies and whether the estimates remain stable across independent calibrations. |
| Repeatability | Calibrate again with an independent recording and check whether the results are broadly consistent. |

The 50 mm baseline is a design value for comparison. Keep the actual estimated extrinsics in the final configuration and validate them through independent VIO runs.

Output files are saved by default under TideConfigurator's `data/jobs/<job-id>/result/` directory.

Keep the raw recording, measured target dimensions, IMU noise file, reports and generated configuration together for traceability and repeat calibration.

## 8. Apply the Configuration and Validate VIO

1. Confirm that you are connected to the device used for this calibration and that its lenses and capture dimensions have not changed.
2. Stop VIO on the VIO Deployment page.
3. Return to Calibration History and click Apply to Device for this job. TideConfigurator backs up the device's old configuration before writing the new one.
4. Check that the calibration configuration's SHA-256 matches the current configuration on the device. When they match, the job card displays Currently in Use.
5. Open the VIO Deployment page, click Start VIO and inspect the trajectory and tracking status.

Use the complete generated `schurvins.yaml`. Avoid manually combining camera and IMU parameters from different jobs, or applying time offsets and IMU compensation twice. Clicking Download VIO Configuration only saves the file to your computer; it does not update the device's configuration.

For validation, move to a scene with clear texture and stable lighting. Initialize while stationary, then perform slow translations and rotations before returning near the starting position. Check whether tracking remains continuous and whether the pose exhibits obvious jumps or rapid divergence. Assess the results over multiple runs.
