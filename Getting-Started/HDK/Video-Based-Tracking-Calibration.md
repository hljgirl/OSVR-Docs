# Video-Based Tracking Calibration

There are two types of calibration involved:

- Calibration of the as-manufactured infrared beacon (IR LED) locations within the tracked device (HDK, etc.) - what we'll call *beacon calibration*.
- Calibration of the video-based tracking coordinate system (specifically, the camera's coordinate system) with the IMU/orientation tracker's coordinate system to allow for fusion of these measurements. We'll call this *room calibration*.

**Note that with the new ("Unified") video tracker plugin, calibration requirements have changed - specifically, they've simplified, and explicit _beacon pre-calibration_ is no longer recommended.**  See below for details.

## Beacon calibration

Tracked devices come with manufacturing specifications as to where the IR LEDs, referred to as "beacons", are located in a device-fixed coordinate system. This is important, however, there is always some degree of variation in manufacturing, and failing to account for the fact that the beacon locations aren't the same as spec can result in noisy, laggy, or unusable tracking.

To the best of our knowledge, **the OSVR video-based tracking system is unique** among consumer-accessible tracking systems in that it uses an "single-constraint-at-a-time"-style algorithm that provides continuous auto-calibration of beacon locations. This means that you don't have to explicitly do any beacon calibration, beacon positions will be updated as required during runtime based on camera measurements/observations.

However, when using the "old" video tracking plugin (videobasedtracker, not unifiedvideotracker), the best tracking data to get very precise autocalibration comes from when the beacons are close to the camera, and the headset is unlikely to be used at those distances, some initial conscious calibration can be helpful. This is really just explicitly exercising the autocalibration routines.

### Pre-calibration - old (non-"Unified") video tracker only

**Do not follow these steps/use the `VideoTrackerCalibrationUtility` if you're using the new video tracker plugin** ("Unified Video-based/Inertial Tracker" aka "UnifiedVideoTracker" aka "uvbi") **as it will actually decrease tracking quality. If you want to see what the tracking camera and algorithm see,** add/modify the `"debugView"` line in your config file so that it reads `"debugView": true`.

If you want to calibrate the IR LEDs prior, there is a separate utility that contains the same autocalibrating tracking algorithm as the plugin, but with a different interface and slightly different operation designed to help you get good measurements fed into the autocalibration system and good calibrated beacon locations saved out.

You run it by starting the application, `VideoTrackerCalibrationUtility` passing the configuration file you use with `osvr_server` as a command line argument. (If you don't explicitly pass one to the server, you don't need to pass one to the calibration utility - they share that logic).

For instance, a sample command line might look like this, if your osvr_server config was `config-testing.json`:

```cmd
VideoTrackerCalibrationUtility.exe config-testing.json
```

This will show an image of the IR camera.

There are two steps: Getting a lock within range, and performing calibration. If you were holding the device up to the camera, you probably skipped right through the first step, otherwise, bring the device to the center of view of the camera and fairly close to it until you see the LEDs and the instructions change.

Follow the instructions until all LEDs enabled on your headset are green. *Note that not all HDK models and IR firmware versions have all the possible LEDs enabled, this is normal. Also note that this utility intentionally does **not** calibrate the rear beacons, since those have to be run-time calibrated only because their position varies with how you wear the headset.* Then hit S to save the calibration file.

Once the config is saved, you don't need to run the calibration utility again unless you want to. The server will automatically load the calibration data every time: you'll see a message in the console to that effect.

The effect of performing this calibration in practical terms is to increase range somewhat, reduce jitter, and improve reliability of tracking.  You can repeat the calibration at any time: the calibration app always starts from scratch. Also, you're not forgoing the benefits of autocalibration by using the calibration utility: autocalibration is still part of the tracking algorithm, you just seed it with more accurate starting data when you have a saved calibration.

See this screencast of a sample calibration to see how it looks when done by a pro (on an HDK 1.x with old IR firmware, so all possible LEDs on):

[![YouTube video link for calibration screencast](https://img.youtube.com/vi/MObPn_U4IYg/0.jpg)](http://www.youtube.com/watch?v=MObPn_U4IYg)

### Notes
You don't need to recalibrate the beacons if you move the camera, it's only if you switch tracked devices (HDKs), or perhaps switch cameras (in case the distortion is subtly different and somehow getting incorporated into the calibration).

It's best to try to nearly fill the field of view, move slowly and also to aim not just for green, but also for consistent small black circles, and lots of beacons active at once as you calibrate (status message at the top of the window). The black circles show the size of the "residual" (error/difference between prediction and measurement) in the last frame for each beacon that's currently being measured.

## Room Calibration

This aligns the internal orientation sensor of the HDK and the video-based tracking. In the new tracking system, combining IMU and video data is integrated in to the single plugin (this is why it's called "unified"), while in the old tracking plugins, this step is part of the "Video-IMU Fusion" plugin. In order to do this, both trackers' outputs must be transformed into some "room coordinate system" - some real-world-based, fixed orientation and position.

For the current configuration with a single camera and a suggested location on the screen, the default is to assume that the room coordinates should be oriented so that the camera is toward the front: that is, facing the camera is facing forward. (This is a configuration option - the alternative is to let the IMU decide what forward is, then potentially use the reset yaw utility later to change the forward direction)  The IMU knows ground truth for all other aspects of orientation. Defaults for position vary between the two tracking systems:

- The new (unified) tracker defaults to placing the camera in the room at (0, 1.2, -0.5) (that's 1.2 meters up, and 0.5 meters back). This was chosen to place the room origin approximately on the real room floor, beneath you, if you are seated in front of the camera at roughly eye level. The camera position is configurable, however, so you may set up your room coordinates differntly if you desire.
- The old tracker plugin used the position of the HMD during room calibration as the origin (0, 0, 0) with vertical position/offset configurable under the name `eyeHeight`.

So, the system needs to know where the camera is relative to some starting location, and more importantly how it is oriented relative to the orientation of the IMU in the headset. Because the mounting system on the camera is flexible, this could change, so **room calibration is performed automatically every time the server starts up**. As a result, if you move the camera, just restart the server.

### Performing the calibration

It's designed to be easy and essentially automatic, though there are some hints printed to the console. The HMD just needs to be reporting orientation and also be visible and tracked by the video-based tracker at the same time. Just hold the HMD still for a few moments in clear view of the camera and it will complete automatically, and positional data will start being reported by the server.

The total amount of time that the algorithm needs to observe the HMD being still is 10 camera frames, which total 1/10th of a second, so this is a very fast calibration process. It's even faster with the new plugin, which does not require the HMD be entirely still (in part because it doesn't set coordinate systems up based on the HMD's location during calibration.)

### Console Messages - Old Tracker
If the HMD is rather far from the camera, a hint (`NOTE: For best results, during tracker/server startup, hold your head/HMD still closer than`...) will be displayed to bring it closer, which both helps the room calibration (by reducing distance, and thus noise, it gives more stable tracking), and also serves as an aid to the automatic beacon calibration. If you bring it within a certain distance, you may see another message saying that the distance is good.

When calibration has detected that the HMD has stopped moving, it will start displaying `Video-IMU fusion: Hold still, measuring camera pose`, adding dots as it collects data. It may restart this message several times as motion or noise is detected; this is harmless.

When room calibration has completed, you'll see a message like `Video-IMU fusion: Camera pose acquired, entering normal run mode!`.
