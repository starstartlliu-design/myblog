---
title: "HCI Lab 4 — 3D Gaussian Splatting in Unity"
date: 2026-11-09
draft: false
tags: [HCI, Unity, Gaussian Splatting, 3DGS]
slug: "hci-lab-4"
categories: ["Human-Computer Interaction"]
toc: true
description: "Importing and exploring a pretrained 3D Gaussian Splatting bouquet in Unity, with practical troubleshooting and observations."
---

## Introduction and Objectives

In this lab, I explored how **3D Gaussian Splatting (3DGS)** can be displayed and inspected in Unity. The objectives are to import a genuine pretrained Gaussian model, convert it into a `GaussianSplatAsset`, bind it to a `GaussianSplatRenderer`, and examine the scene from different viewpoints. I followed the Unity workflow introduced in Mohamed ABDUL GAFOOR's *3D Gaussian splatting in unity* lecture slides.

I used an existing pretrained model rather than collecting photographs or training a reconstruction. The focus of this exercise was importing, rendering, and observing the Gaussian scene in Unity.

## Tools and Environment

| Component | Actual configuration |
| --- | --- |
| Operating system | Windows 11 Home, version 10.0.26200 |
| CPU and RAM | Intel Core i7-14650HX, approximately 16 GB RAM |
| GPU | NVIDIA GeForce RTX 5060 Laptop GPU, 8,151 MiB reported VRAM |
| NVIDIA driver | 592.19 |
| Unity Editor | 6000.0.85f2 |
| Render pipeline | Built-in Render Pipeline |
| Graphics configuration | Direct3D 12, MSAA disabled |
| UnityGaussianSplatting | v1.1.1, commit `9310dce438da726244ace17eaf6f768826435fa4` |
| Resolved packages | Burst 1.8.30, Collections 2.7.1, Mathematics 1.3.2 |

The lecturer demonstrates Unity 6000.0.23f1. I reused the installed 6000.0.85f2 editor and evaluated compatibility through the actual project. I kept this experiment in a separate Unity project.

The renderer's current main branch uses a newer direct PLY/SPZ loading workflow. I pinned **v1.1.1** because it retains the `Tools > Gaussian Splats > Create GaussianSplatAsset` workflow shown in the slides. This kept the asset-conversion steps consistent with the lecture. I used the repository’s Built-in Render Pipeline example project, without adding a URP renderer feature. On Windows, the plugin requires Direct3D 12 or Vulkan and does not support Direct3D 11 for this workflow.

## Gaussian Splatting Workflow

### Representation and rendering

A Gaussian primitive has a 3D center, an anisotropic spatial extent and orientation, an opacity, and color parameters. The PLY stores scales and a rotation quaternion that determine its covariance. Spherical harmonics encode a color function of viewing direction. During rendering, the system projects the Gaussian footprints into the image, orders the splats, and composites their contributions.

A mesh describes surfaces through connected triangles. This model instead stores overlapping Gaussian primitives with no triangle connectivity. Although the file contains positions, treating it as an ordinary point cloud would discard its scale, rotation, opacity, and directional appearance. Those additional fields, and the Gaussian renderer that consumes them, are essential to this experiment.

### Model selection, provenance, and licensing

I used **“3D gaussian model of a bouquet of flowers” by Jean-Eudes Marvie**, distributed through NVIDIA's Vulkan Gaussian Splatting sample. I chose the bouquet because it provides a manageable Gaussian PLY with detailed flowers and leaves that are useful for comparing different viewpoints. Using an existing model also kept the exercise focused on Unity visualization.

| Model check | Recorded result |
| --- | --- |
| Original archive | 125,568,178 bytes |
| Extracted PLY | 139,619,083 bytes, binary little-endian |
| Gaussian count | 562,974 |
| Properties | 62 float fields, including opacity, 3 scales, 4 rotation values, 3 DC and 45 higher SH coefficients |
| Numerical check | All stored scalar values are finite |
| PLY SHA-256 | `3641192d70d598894f4d6ad92e0890b4a7336b43aa6af47bd35842c3e2ee8765` |

**License discrepancy:** the author's license file inside the downloaded archive states **CC BY 4.0**, while NVIDIA's dataset page labels the bouquet **CC BY-SA 4.0**. I retained both source records rather than silently treating them as identical. I credit the author and distributor and record both notices. For any redistribution of the converted model data, the applicable terms should be confirmed with the rights holder or distributor. The renderer itself remains MIT licensed. Conversion changes storage and precision; it does not constitute retraining.

Model attribution: *Bouquet of flowers Gaussian model © Jean-Eudes Marvie, distributed by NVIDIA; rendered with UnityGaussianSplatting by Aras Pranckevicius. Archive license: CC BY 4.0; distributor page: CC BY-SA 4.0. The applicable terms for redistributing converted data require clarification.*

### Importing the model into Unity

1. I prepared the UnityGaussianSplatting Built-in example project using the selected v1.1.1 revision.
2. I converted the bouquet Gaussian PLY using the plugin’s `Create GaussianSplatAsset` functionality with the Medium quality preset.
3. I configured a GameObject with `GaussianSplatRenderer`, assigned the converted asset and the required shader resources, and saved the scene.
4. I set up a camera and Direct3D 12 rendering, then inspected the bouquet from different viewpoints.

The conversion used the plugin’s existing asset-creation implementation, not a custom Gaussian renderer. The generated resources contain position, color, scale/rotation, spherical harmonics, and chunk data. The original PLY was kept unchanged.

## Unity Implementation

The scene is `Assets/Lab4/Scenes/Flowers.unity`. Its model GameObject references `Assets/Lab4/GaussianAssets/flowers_1.asset`. The renderer uses the supplied splat, composite, debug, and utility compute shaders. The imported asset retains the original Gaussian count and uses the upstream Medium compression preset.

The camera uses a solid dark background, a 42° field of view, a 0.01–100 clipping range, HDR, and no MSAA. The model transform converts its stored axes for inspection in Unity. No directional light is necessary: the plugin displays the stored radiance appearance and does not relight the Gaussian model using Unity scene lights.

To inspect the result, I added a small camera-control component with two preset views and mouse controls for orbit and zoom. Preset switching and wheel zoom were tested; right-drag orbit was implemented but not independently verified. This is a viewing interface; it does not add collision geometry, object manipulation, or navigation through a reconstructed surface.

## Experimental Results

| Validation stage | Evidence and outcome |
| --- | --- |
| C# compilation and Player build | Editor import and compilation completed; final graphics-enabled Player build succeeded with 0 errors and 0 warnings; earlier logs retain the upstream obsolete-API warning |
| Gaussian asset import | 562,974 Gaussians; five data files plus the asset total 27,272,095 bytes; saved scene and shader references validated |
| Actual GPU rendering | Unity Editor and final Windows Player used Direct3D 12 on the RTX 5060; actual 1280 × 800 Player screenshots were saved and visually inspected |
| Changed viewpoint | Two camera positions and orientations recorded; both images show the bouquet with different occlusions |
| Play Mode interaction | Keys 1 and 2 changed the displayed preset; wheel input changed the apparent model size; full-window captures retained |

Compilation, asset import, GPU rendering, and viewpoint changes are separate checks. A successful batch exit alone cannot establish that the model appeared on screen. Here, the evidence includes the actual rendered images as well as a runtime report of valid GPU resources and the saved scene's asset references.

![Bouquet Gaussian model rendered from the main camera](/myblog/images/hci-lab4/render-view-1.png)

*Figure 1. Actual Unity Windows Player screenshot at 1280 × 800. The camera is approximately (−1.36, 1.12, 3.21), with Euler rotation (12°, 155°, 0°). The main view shows identifiable petals, leaves, and overlapping flowers.*

![Bouquet Gaussian model rendered from a second camera](/myblog/images/hci-lab4/render-view-2.png)

*Figure 2. Actual output from the same scene with camera position approximately (2.41, 1.61, 2.59) and rotation (20°, 220°, 0°). Occlusion changes, and some peripheral flower regions show stretched or blurred splats. This is an observed limitation rather than an image-quality success claim.*

![Unity Play Mode and GaussianSplatRenderer asset binding](/myblog/images/hci-lab4/unity-play-view-1.jpg)

*Figure 3. Actual Unity Play Mode Game View and Inspector. The renderer binds flowers_1, uses Splats mode, and reports 562,974 splats. The displayed controls provide preset changes and zoom. The upstream Inspector warning concerns editing with a lossy compression preset, not a missing model.*

All three figures use Jean-Eudes Marvie's bouquet model, distributed by NVIDIA. Model and license attribution appears above. Figures 1–2 are unaltered Player framebuffer screenshots; Figure 3 is an unaltered full-window Editor capture.

No frame-rate benchmark, memory benchmark, participant study, or subjective quality score is claimed. The Gaussian count comes from the PLY and imported asset rather than an estimate from a screenshot.

## Problems Encountered and Solutions

### Plugin version and Unity compatibility

The latest repository revision uses a different model-loading workflow from the one demonstrated in class. I selected v1.1.1 to retain the `Create GaussianSplatAsset` procedure. Because the lecture used an earlier Unity 6 version, I checked the chosen version through compilation and actual rendering rather than assuming compatibility.

### Renderer initialization and screenshot capture

During an initial run without a graphics device, the asset and scene were saved, but initializing the renderer produced an `InitDeviceRadixSort` kernel exception. A persistent-allocation warning also appeared on shutdown. The issue was resolved for the visual demonstration by opening the scene with an active Direct3D 12 graphics device. The Gaussian model then appeared correctly in the rendered view.

An early Player run also failed to save its requested screenshots, even though GPU resources were reported as valid. I therefore checked the image through a Unity Camera and RenderTexture, inspected the result in Editor Play Mode, and captured the final screenshots from a visible Windows Player. The final graphics-enabled build completed with **0 errors and 0 warnings**.

The final Player log still included Direct3D diagnostic information and an upload-buffer sizing message (16 MiB requested capacity versus an approximately 18 MB spherical-harmonics resource). These did not prevent the two observed views from rendering. However, this short experiment does not establish long-session stability.

### Choosing a useful observation direction

The initial presets faced the less complete side of the bouquet, exposing large stretched regions. Moving the camera 180° around the same model revealed a substantially clearer main view. The model data stayed unchanged. The second selected view still exhibits peripheral artifacts, so this adjustment improves the demonstration without establishing that the reconstruction is complete or equally clear from every direction.

I also encountered a Unity project lock when another Editor process was still using the project. Closing or waiting for the active process before reopening the project avoided the conflict.

### Conflicting model license labels

The model archive and distributor page use different Creative Commons labels. Preserving the actual bundled notice, the distributor's statement, attribution, and conversion details makes this uncertainty visible and avoids claiming that the plugin's MIT license covers the model.

## HCI Discussion

### Free viewpoints and spatial understanding

Interactive camera movement can reveal relative depth through motion parallax and changing occlusion. Compared with a fixed photograph, it lets a viewer inspect how petals and stems occupy space. Preset views provide repeatable reference points, while orbit and zoom support exploration. These are interface affordances and potential benefits; this lab does not measure improved spatial understanding in users.

### View-dependent appearance

Spherical harmonics allow the stored color to vary with the viewing direction. This matters for appearances that a single constant RGB value cannot describe. It also means a plausible image from one camera is insufficient to assess the scene across viewpoints. Moving the camera can expose reconstruction gaps, floating splats, or transparency artifacts. A two-view inspection is a limited functional check, not a complete evaluation of directional reflectance or reconstruction accuracy.

### Interaction limits

The Gaussian representation supplies visual appearance without a conventional collision surface or semantic object structure. Orbiting the bouquet is possible, but grabbing an individual flower or walking against its surface would require additional representation and interaction work. The plugin's alpha compositing also limits interaction with ordinary transparent objects, and it does not provide dynamic relighting. These constraints matter when designing an immersive application where users expect consistent contact, occlusion, and lighting feedback.

## Limitations and Conclusion

This experiment covers one pretrained object, one Windows machine, one renderer version, and a desktop viewing interface. It does not validate model training, VR/mobile support, quantitative image quality, general hardware compatibility, or user usability. Raw logs and captures define the scope of the claims; unfinished checks remain explicit.

The experiment successfully converted a genuine pretrained Gaussian model into a Unity asset and displayed it in a saved scene. Two viewpoints and basic Play Mode controls were verified. Right-drag orbit is implemented but was not separately confirmed by an input test. The visible artifacts also show why checking more than one view matters when presenting a reconstructed scene.

## References

1. Mohamed ABDUL GAFOOR. *3D Gaussian splatting in unity*. VAR master program lecture slides, especially slides 5–17. Local course material supplied for this lab.
2. Aras Pranckevicius. [UnityGaussianSplatting v1.1.1](https://github.com/aras-p/UnityGaussianSplatting/tree/9310dce438da726244ace17eaf6f768826435fa4), [MIT license](https://github.com/aras-p/UnityGaussianSplatting/blob/9310dce438da726244ace17eaf6f768826435fa4/LICENSE.md), and [pipeline integration](https://github.com/aras-p/UnityGaussianSplatting/blob/9310dce438da726244ace17eaf6f768826435fa4/docs/render-pipeline-integration.md).
3. Bernhard Kerbl, Georgios Kopanas, Thomas Leimkühler, and George Drettakis. [3D Gaussian Splatting for Real-Time Radiance Field Rendering](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/), ACM Transactions on Graphics, 2023. [Training implementation](https://github.com/graphdeco-inria/gaussian-splatting).
4. Hugging Face. [Gaussian Splatting](https://huggingface.co/blog/gaussian-splatting), explanatory resource also cited in the lecture.
5. Jean-Eudes Marvie / NVIDIA. [Bouquet model archive](https://developer.download.nvidia.com/ProGraphics/nvpro-samples/flowers_1.zip), [author](https://jmarvie.com), and [distributor dataset and license statement](https://github.com/nvpro-samples/vk_gaussian_splatting/blob/main/docs/datasets.md). The archive includes the author's CC BY 4.0 notice.
6. Creative Commons. [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) and [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
