+++
title = "HCI Lab 2 — Unity Mobile Application: From Desktop to Android"
date = 2026-11-08T12:00:00+01:00
draft = false
description = "Adapting the Moving Target Challenge to Android: virtual joystick control, NDK troubleshooting, an ARM64 build, and testing on an OPPO Find X8s."
slug = "hci-lab-2"
tags = ["HCI", "Unity", "Android", "ML-Agents"]
categories = ["Human-Computer Interaction"]
toc = true
+++

<style>
.post-content:has(.hci-lab2-marker) { overflow-wrap:anywhere; }
.post-content:has(.hci-lab2-marker) table { display:block; max-width:100%; overflow-x:auto; }
.post-content:has(.hci-lab2-marker) pre { max-width:100%; overflow-x:auto; overflow-wrap:normal; }
.post-content:has(.hci-lab2-marker) figure { margin:2rem auto; }
.post-content:has(.hci-lab2-marker) figure img { display:block; width:100%; height:auto; margin:auto; }
.post-content:has(.hci-lab2-marker) figure.lab2-phone { max-width:280px; }
.post-content:has(.hci-lab2-marker) figcaption { font-size:.9em; line-height:1.5; text-align:center; margin-top:.7rem; }
</style>
<span class="hci-lab2-marker" hidden></span>

## 1. Introduction and Objectives

In this laboratory, I adapted the Moving Target Challenge from [HCI Lab 1]({{< relref "/posts/hci-lab-1" >}}) into an Android application. The original environment placed a blue spherical Agent and a red moving Target inside a bounded arena. My objective was to make the same task playable on a touchscreen, using a virtual joystick to control pursuit.

The main challenge was to change the interaction and deployment platform while preserving the experiment underneath. I wanted to retain the movement rules, target motion, rewards, and episode resets, rather than introduce a different task. The work therefore covered touch input, screen adaptation, Android dependencies, APK construction, and testing on an OPPO Find X8s.

The final application installed and ran on the phone, and hands-on interaction testing reported no abnormalities. However, two Input System diagnostics remained reproducible during cold startup. I distinguish these unresolved startup errors from the successful rendering and control results throughout this report.

## 2. Reusing the Previous Unity Project

I kept the Lab 1 project at `D:\HCI_Lab1\MovingTargetChallenge` and worked in a separate copy at `D:\HCI_Lab2\MovingTargetChallengeAndroid`. The copy retained `Assets`, `Packages`, `ProjectSettings`, and `TrainingConfigs`; regenerable directories such as `Library`, `Temp`, and `Logs` were excluded. SHA-256 checks covered 96 original files, which remained unchanged after development and testing.

The main scene remained `Assets/Scenes/MovingTargetChallenge.unity`. The existing `MovingTargetAgent.cs` supplied observations, action handling, rewards, and episode management, while `MovingTargetMotion.cs` controlled the moving target. The original ONNX model and training configuration were preserved. I did not retrain PPO or run ONNX inference on Android: this version is a manually controlled mobile application.

The retained physical settings were important for comparison with the desktop task:

| Parameter | Retained value |
|---|---:|
| Arena size | 16 × 16 units |
| Agent position limit | ±7.5 units |
| Target position limit | ±7.4 units |
| Agent acceleration | 18 units/s² |
| Agent maximum speed | 6 units/s |
| Target speed | 1.5 units/s |
| Capture distance | 0.95 units |
| Physics timestep | 0.02 s |
| Decision period | 5 physics steps |
| Maximum episode length | 2,000 physics steps, approximately 40 simulated seconds |

Rewards still account for progress toward the target, a step penalty, capture, and leaving the arena. Target motion and the reward function were not revised. This keeps the mobile work focused on access to the existing task rather than changing the learning experiment or claiming new policy performance.

## 3. Mobile Interaction Design

I placed a fixed virtual joystick in the lower-left corner, outside the main arena display on the tested phone. Its translucent base and blue handle make the control distinguishable from the blue Agent. The current control uses simple square shapes; it is a functional interface rather than a finished visual redesign.

`VirtualJoystick.cs` tracks one pointer, converts its screen position into local UI coordinates, and limits the resulting `Vector2` to the unit circle. Its X component becomes the first continuous action, controlling world X; its Y component becomes the second action, controlling world Z. The Agent still applies these actions through `OnActionReceived`, so the joystick does not move the Transform directly.

```csharp
Vector2 manual = Joystick != null && Joystick.IsHeld ? Joystick.Value : new Vector2(x, z);
actions[0] = manual.x;
actions[1] = manual.y;
```

The keyboard Heuristic remains available for desktop debugging when the joystick is not held. Pointer release, cancellation, disabling, focus loss, and application pause reset the joystick input. Releasing the control resets the requested acceleration, not the Rigidbody velocity: the original damping allows the Agent to slow down with inertia. The retained decision interval is nominally 0.1 s; I did not measure end-to-end touch latency.

The scene contains a Canvas with Canvas Scaler and GraphicRaycaster, a SafeArea container, the joystick and handle, and one EventSystem using `InputSystemUIInputModule`. The Canvas uses a 1920 × 1080 reference resolution. `MobileViewport.cs` follows `Screen.safeArea` and adjusts the orthographic camera size to retain the arena on narrower views. Landscape orientation keeps the play area and thumb control readable together.

<figure class="lab2-landscape">
  <img src="/myblog/images/hci-lab2/phone-gameplay.png" alt="Actual landscape Android application showing the blue Agent, red Target, arena and lower-left joystick" loading="lazy" width="2344" height="1080">
  <figcaption>Figure 1. The application running on the OPPO Find X8s. The arena and joystick are visible in a real 2344 × 1080 landscape capture. A frame demonstrates display, not touch accuracy or frame rate.</figcaption>
</figure>

## 4. Android Environment and Build Configuration

I retained the project's Unity and package versions and added the Android platform components for the same Editor installation. The verified environment was:

| Component | Version or configuration |
|---|---|
| Unity Editor | 6000.0.85f2 |
| Universal Render Pipeline | 17.0.4 |
| ML-Agents | 4.1.0 |
| Input System | 1.20.0 |
| App UI dependency | 1.3.6 |
| Inference dependency | `com.unity.ai.inference` 2.6.1 |
| Android NDK | r27c, 27.2.12479018 |
| OpenJDK | 17.0.18 |
| SDK Build-tools | 36.0.0 |
| Android architecture / scripting backend | ARM64 / IL2CPP |
| Minimum / target Android API | 23 / 36 |
| Application identifier | `com.hcilab.movingtarget.android` |
| Build type | Development APK |

Unity identified the SDK, NDK, and OpenJDK beneath `D:\HCI_Lab1\UnityEditor\6000.0.85f2\Editor\Data\PlaybackEngines\AndroidPlayer`. The NDK needed a directory repair, described below, but the successful SDK and JDK installations were retained. I did not reinstall Unity to correct the Hub's NDK status.

The Android build explicitly selected `MovingTargetChallenge.unity`, avoiding the template `SampleScene` entry. The separate Android Builder configured ARM64, IL2CPP, landscape orientation, and the application identifier. The saved scene was reopened to check its references before building, and the APK was written outside the project to `D:\HCI_Lab2\Builds\stage3-20261108`.

## 5. Problems Encountered and Solutions

### 5.1 Desktop Input Adaptation

The desktop Heuristic read WASD or arrow keys, which did not provide an on-screen control for the phone. I investigated the existing two-action path before adding the joystick. Since both actions already represented horizontal acceleration, replacing their input source was sufficient; rewriting the Rigidbody movement would have changed the task unnecessarily.

The solution was to feed the joystick into the same Heuristic action buffer and preserve keyboard fallback. On-device evidence showed rightward movement and handle recentering, and hands-on testing covered directional control, release behavior, multiple touches, background return, and episode reset without reported abnormalities. These checks do not quantify precision, accessibility, or latency.

### 5.2 ML-Agents Control Mode

The original scene used `Default` behavior with no model reference. That mode can select trainer communication, model inference, or Heuristic control depending on availability. For a touch-controlled application, I made the intended route explicit with `HeuristicOnly` and an empty model reference.

I also checked `LabExperimentRuntime` and `LabBuildSceneProcessor`. Their experiment-policy branches could intercept manual input, and the former used a default time scale of 20 for non-training policies. The Android build no longer injects that runtime component; the Lab 2 default evaluation time scale was changed to 1, and joystick input bypasses the baseline branch. Reloading the scene confirmed the joystick reference, `HeuristicOnly`, and the absence of `LabExperimentRuntime`. ML-Agents and inference dependencies remain installed, but PPO does not control this application.

### 5.3 Mobile Screen Adaptation

The desktop camera and screen assumptions were unsuitable for arbitrary phone proportions. A portrait or narrow view could reduce the visible horizontal area, while a control placed against the physical screen edge could overlap an inset or system gesture region.

I used landscape layout, Canvas Scaler, a Safe Area container, and an aspect-aware orthographic camera rather than changing the arena dimensions. The phone captures show the complete arena, both characters, and the joystick. After a background return, one capture caught the system's orientation transition; a later stable capture confirmed the landscape layout. This verifies the tested device, not every Android screen or display cutout.

### 5.4 Android NDK Installation Failure

Unity Hub failed twice while installing Android NDK r27c. Both attempts reached the end of extraction, then failed during directory renaming. The log contained:

```text
unzip finished
EPERM: operation not permitted, rename
Module Android NDK after installation step failed
```

The errors occurred at approximately 14:28:03 and 14:28:56 on 8 November 2026, local time (UTC+1). The files remained under `AndroidPlayer\NDK\android-ndk-r27c`, rather than directly under the NDK root. Hub was trying to move that nested directory to a temporary sibling before completing its layout. The failure was therefore an installation-finalization problem, not evidence that the download or extraction had failed.

I checked the cache ZIP, directory contents, free space, permissions, and file-use information after Hub and its installer processes had exited. The archive contained 7,935 files, with an uncompressed total of 2,361,056,613 bytes. All extracted files matched their ZIP counterparts by SHA-256. The permission check found no explicit deny rule in the checked directories, and the file-use query reported no owners. These observations did not establish the original cause of `EPERM`.

The repair copied the verified contents into Unity's expected `AndroidPlayer\NDK` root while leaving the original nested directory intact as a rollback copy. No different existing file was overwritten, and the download ZIP was retained. The 7,935 files at the repaired root also matched the ZIP by SHA-256. `source.properties` reported `27.2.12479018`; Clang and LLVM reported 18.0.3, while Make and `ndk-build.cmd --version` reported GNU Make 4.3, all with exit code 0. Unity later recognized this NDK path and completed the Android build. The Hub's display state was not repaired or treated as the final toolchain test.

### 5.5 Unity Compilation and Build Issues

The Android Builder initially contained a path-character escaping error and a mismatch between its build-summary field types and Unity's API. The compiler identified the problems directly:

```text
error CS1010: Newline in constant
error CS0266: Cannot implicitly convert type 'int' to 'uint'. An explicit conversion exists (are you missing a cast?)
```

I corrected path trimming to use `Path.DirectorySeparatorChar` and changed the error/warning count fields from `uint` to `int`. Compilation then passed. The build output boundary was also adjusted to allow the separate `D:\HCI_Lab2\Builds` directory. These changes affected the Editor build code, not rewards, movement, target behavior, or the ONNX model.

The final build reported:

```text
Build Finished, Result: Success.
```

BuildReport recorded 0 errors, 971 warnings, and 498.19 s, approximately 8 min 18 s. The Editor process, including startup and shutdown, took 522.00 s, approximately 8 min 42 s. The main log contained 970 Sentis shader warning entries, including 920 `ConvGeneric` variant warnings; the Gradle log also contained a Jetifier deprecation notice. I retained these warnings rather than remove inference dependencies. The build process exited with code 0, and APK signature verification passed. A successful build does not establish that every inference backend or device configuration works.

### 5.6 Unity Input System Startup Warnings

The application displayed correctly, but its first device startup produced an exception and a duplicate-touchscreen assertion. Although grouped here as startup diagnostics, these were logged as errors rather than harmless information:

```text
ArgumentException: 'InputUpdateType.None' is not a valid update mask
Already added touchscreen
```

The available stack traces point to `InputStateHistory.set_updateMask`, `EnhancedTouch.Finger`, `Touch.AddTouchscreen`, and native-device discovery in Input System. The duplicate assertion appears when `EnhancedTouchSupport.SetUpState` adds the screen again. The traces do not enter UGUI, `VirtualJoystick`, or the Agent script. Local source inspection showed that a screen is added to the list before finger-history creation; a failure at the latter stage can leave a screen entry for the next addition.

App UI enabling Enhanced Touch and Input System resetting its static state during the same startup phase provide a possible initialization-order explanation. However, the exact triggering order was not recorded, so I do not claim a confirmed root cause. There was no demonstrated safe project-only fix. I kept the package versions and code unchanged rather than suppress errors or force internal state changes.

The same APK was cold-started without clearing application data. The two messages each appeared once in the initial startup and once in the cold-start retest. Duplicate snapshots of the same event were not counted as new occurrences. Cold startup and background recovery succeeded, and no application crash or ANR was found within the captured windows. **The errors remain unresolved known limitations; normal touch behavior is not proof that they are fixed.**

## 6. Deployment and Device Testing

I used the ADB executable included with Unity's Android SDK. The initial connection was `unauthorized`; after accepting USB debugging on the phone, it changed to `device`. The queried hardware and software matched the APK requirements:

| Device property | Observed value |
|---|---|
| Phone | OPPO Find X8s; ADB model PKT110 |
| Android version / API | 16 / 36 |
| CPU ABI | `arm64-v8a` |
| Physical screen | 1080 × 2344 pixels |
| Landscape application capture | 2344 × 1080 pixels |
| Physical / override density | 560 / 480 dpi |

The package had not previously been installed. Installation returned `Success`, and launching `com.hcilab.movingtarget.android` returned `Status: ok`. The relevant commands were:

```powershell
adb devices -l
adb install "D:\HCI_Lab2\Builds\stage3-20261108\MovingTargetChallenge.apk"
adb shell am start -W -n com.hcilab.movingtarget.android/com.unity3d.player.appui.AppUIGameActivity
```

<figure class="lab2-phone">
  <img src="/myblog/images/hci-lab2/phone-home-01.png" alt="Real OPPO home screen with the installed Moving Target Challenge Unity icon" loading="lazy" width="1080" height="2344">
  <figcaption>Figure 2. The installed Moving Target Challenge icon on the phone's home screen. The complete desktop capture retains the original wallpaper and surrounding icons.</figcaption>
</figure>

I checked the running scene, joystick response, release behavior, and return from the background. Hands-on testing reported no abnormalities for the requested directions, recentering and inertia, multiple touches, background return, and new episodes after capture or leaving the arena. Screenshots support the rendering checks; they do not independently prove all of these interactions. The later cold-start retest reused the same APK and did not repeat the hands-on checklist.

The first Activity launch reported 382 ms, while the cold-start retest reported 525 ms. These are Android Activity timings, not complete Unity loading times or touch-response measurements. During background-return checks, the same application process remained alive. Short observation windows showed no crash or ANR, but they do not constitute a long-duration stability study.

## 7. Results and Discussion

The final artifact and verification results were:

| Item | Result |
|---|---|
| Unity import and C# compilation | Successful after the Builder corrections |
| Mobile scene | Joystick bound; one EventSystem; `HeuristicOnly` |
| Android build | Successful; process exit code 0 |
| BuildReport | 0 errors; 971 warnings |
| Build duration | Approximately 8 min 18 s |
| APK size | 60,857,736 bytes, approximately 58.04 MiB |
| ABI / backend | `arm64-v8a` / IL2CPP |
| Minimum / target API | 23 / 36 |
| Installation / launch | `Success` / `Status: ok` |
| Hands-on interaction | No abnormalities reported |
| Cold startup / background recovery | Successful, with the startup diagnostics still present |
| PPO retraining / Android ONNX inference | Neither performed |

The APK remains at `D:\HCI_Lab2\Builds\stage3-20261108\MovingTargetChallenge.apk`. Its SHA-256 is:

```text
6DE3F65D9C86A5E37AFA7AB84599FB11CA16AD50C3FE5D933C7D55C75A6BC028
```

The installed `base.apk` matched this hash, and signature verification succeeded with v1 and v2. This links the tested application to the retained build artifact. The APK size comes from the file itself, rather than BuildReport's larger aggregate output size.

The main result is a usable touch-controlled port of the same environment, with clear separation between input and movement logic. Equally important, the NDK investigation showed why a failed installer status should be traced to its actual failing step. Copying verified files into the required layout solved that specific installation obstacle without changing the task or replacing successful SDK and JDK components.

The remaining limitations are substantial enough to record explicitly. The Input System startup errors are reproducible and unresolved. Only one physical device was tested, and I did not benchmark frame rate, temperature, energy use, or end-to-end input latency. There was no emulator evaluation, long-duration stability run, or Android PPO inference test. The retained 0.1 s decision interval and square joystick may deserve later usability evaluation, but I did not change them or claim a measured improvement.

## 8. Conclusion

I brought the Moving Target Challenge from desktop Unity to a working Android application while preserving its arena, physics, rewards, target motion, and original training materials. The mobile version uses explicit Heuristic control, a virtual joystick, landscape UI, and a verified ARM64 / IL2CPP APK tested on an OPPO Find X8s.

The most useful lesson was to separate successful installation, compilation, packaging, visible execution, and interaction from a claim of error-free operation. The port meets its basic mobile interaction objective, and the NDK installation obstacle was repaired with verified files and a rollback copy. The reproducible Enhanced Touch startup diagnostics remain a documented defect that needs further targeted investigation, rather than being hidden by the application's otherwise successful behavior.
