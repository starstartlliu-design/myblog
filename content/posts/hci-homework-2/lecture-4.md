---
title: "Beyond Conventional Interfaces: Chris Harrison’s Research on Sensing and Haptic Interaction"
date: 2026-10-31T00:00:00+02:00
draft: false
slug: "lecture-4"
weight: 4
math: true
homework: "hci-homework-2"
lecture: 4
categories: ["Human-Computer Interaction"]
tags: ["HCI", "Homework 2"]
---

# Beyond Conventional Interfaces: Chris Harrison's Research on Sensing and Haptic Interaction

**HCI Homework 2 — Lecture 4**

## 1. Introduction: Who Is Chris Harrison?

Chris Harrison is a professor at Carnegie Mellon University's Human-Computer Interaction Institute (HCII) and the director of the Future Interfaces Group (FIG). His research explores new ways for people to interact with computers by combining sensing technologies, computer vision, physical computing, and interface design.

Traditional human-computer interaction relies heavily on keyboards, mice, touchscreens, and other dedicated input devices. Although these technologies are effective, they also constrain interaction to particular devices and predefined gestures. Emerging technologies such as augmented reality (AR), mixed reality (MR), and virtual reality (VR) create opportunities for more natural interaction involving the human body and the surrounding physical environment.

Harrison's research is particularly interesting because it investigates how physical phenomena can become useful interaction signals. Rather than treating shadows, optical reflections, or mechanical forces as isolated engineering problems, his research group uses them to develop practical interactive systems.

Three recent projects illustrate this approach:

- **EclipseTouch (2025):** Detecting touch on ordinary surfaces using infrared shadows.

- **HiFiGaze (2026):** Improving eye tracking by analyzing reflections of screen content.

- **Kinethreads (2025):** Generating full-body force feedback through a wearable cable-driven system.

These projects represent three complementary aspects of HCI: touch sensing, gaze-based input, and physical feedback.

## 2. EclipseTouch: Touch Detection Through Infrared Shadow Geometry

### 2.1 Research Problem

In conventional touchscreen systems, touch detection depends on sensors integrated into the display surface. This works well for smartphones and tablets but becomes less convenient in mixed reality, where users may want to interact with virtual content positioned on ordinary desks, walls, or other physical objects.

One alternative is mid-air hand tracking. However, interacting with floating interfaces may require users to hold their arms in uncomfortable positions, and accurately distinguishing an intentional touch from a nearby finger can be difficult.

EclipseTouch addresses this problem by detecting contact with everyday surfaces that have no dedicated sensing hardware.

### 2.2 Physical Principle: Shadow Projection

The central idea is that the geometry of a shadow contains information about the three-dimensional position of the object casting it.

When infrared light illuminates a finger above a surface, the finger blocks part of the light and creates a shadow. As the finger moves toward or away from the surface, the relative position and shape of its shadow change.

This can be illustrated using a simplified geometric model.

Suppose a point light source is located at height \(H\) above a horizontal surface. A point on the finger is located at height \(h\), with a horizontal distance \(d\) from the light source's vertical projection.

Using similar triangles, the horizontal displacement between the finger point and its projected shadow is

\[\Delta x=\frac{dh}{H-h}.\]

This equation shows that shadow displacement depends on the finger's height relative to the surface.

As \(h\) approaches zero, the displacement approaches zero in this idealized model. Therefore, shadow geometry can provide information relevant to distinguishing hovering from physical contact.

**This equation is a simplified physical explanation, not a reproduction of the EclipseTouch algorithm.** The actual research system uses structured infrared shadows and image-based processing rather than relying on this single geometric expression.

### 2.3 System Implementation

EclipseTouch integrates a computer-triggered camera with one or more infrared emitters in a head-worn sensing configuration.

The infrared emitters illuminate the interaction region, while the camera observes the resulting shadow patterns. The system processes these images to estimate whether the finger is touching the surface and how far it is hovering above it.

The use of infrared illumination is useful because it provides a sensing signal separate from the visible interface presented to the user.

A key engineering advantage is that the physical surface does not require embedded electronics. Much of the sensing functionality is instead provided by the wearable device.

### 2.4 Experimental Results and Limitations

The researchers reported:

- **98.0% accuracy** for detecting touch contact.

- **6.9 mm mean error** for estimating hover distance.

The evaluation also examined different surface materials, interaction orientations, and lighting conditions.

These results demonstrate the feasibility of infrared shadow-based touch detection. However, the reported accuracy describes the tested conditions rather than a guarantee of identical performance in every environment.

From an engineering perspective, several challenges remain important. Surface reflectance can affect image quality, environmental illumination can influence infrared contrast, and occlusion may complicate the interpretation of overlapping shadows.

### 2.5 Contribution to HCI

EclipseTouch illustrates how ordinary physical environments can become interactive without being modified internally.

Its significance is not merely accurate finger tracking. More importantly, it provides an approach to connecting digital interfaces with physical surfaces, potentially making mixed-reality interaction more natural and less dependent on specialized hardware.

## 3. HiFiGaze: Eye Tracking Using Optical Reflections

### 3.1 Research Problem

Eye tracking estimates where a person is looking and can support gaze-controlled interfaces, attention-aware applications, accessibility technologies, and immersive systems.

High-precision eye-tracking systems often require specialized cameras and illumination. Consumer devices, by contrast, typically use ordinary front-facing cameras, making accurate gaze estimation more challenging.

HiFiGaze investigates whether the display itself can provide additional information for estimating gaze direction.

The researchers exploit a simple observation: light emitted by a screen can be reflected by the user's eyes, and modern high-resolution cameras can capture these reflections.

### 3.2 Optical Principle: Corneal Reflection

The anterior surface of the cornea is a curved optical interface. Because the refractive indices of air and the cornea differ, part of the incident light is reflected.

For conceptual understanding, the corneal surface can be approximated as a convex reflecting surface.

When a person looks in different directions, the orientation of the eye changes. Consequently, the position, shape, and visible extent of the screen reflection in the camera image can also change.

This reflection contains information about the spatial relationship among the display, the eye, and the camera.

However, the reflection alone is difficult to interpret because screen content changes continuously. A bright rectangular window, for example, produces a different reflected pattern from a dark image.

HiFiGaze introduces an important additional constraint: **the computer already knows exactly what it is displaying.**

The displayed image can therefore serve as useful prior information for identifying and interpreting the reflected screen region.

### 3.3 System Implementation

The approach combines three main information sources:

1. **Eye images:** A front-facing camera captures the user's eyes and their visible reflections.

2. **Known screen content:** The computer provides information about the image currently displayed.

3. **Computational gaze estimation:** The system uses reflection-related features and image-based modeling to estimate the user's gaze location.

A major challenge is reflection segmentation: identifying the screen reflection within an eye image that also contains the pupil, iris, eyelids, eyelashes, and other visual features.

Knowledge of the displayed content helps the system distinguish relevant reflected patterns from unrelated image information.

The research evaluates different strategies for incorporating this additional signal into gaze estimation.

Importantly, the system should not be understood as a purely geometric calculation based on a perfectly spherical eye. Its contribution involves combining optical information with computational image analysis.

### 3.4 Experimental Results

The published CHI 2026 project description reports that the best-performing model reduces mean gaze-tracking error by approximately **18%** compared with an appearance-based baseline.

A supplementary study reports an additional **10–20% improvement** when the gaze-tracking camera is positioned at the bottom of the device.

These percentages describe relative improvements over particular experimental configurations. They do not represent absolute tracking accuracy.

The camera-placement result is especially interesting from an optical engineering perspective because it demonstrates that sensor geometry can substantially influence the quality of the information available to the algorithm.

### 3.5 Individual Eye Differences and Technical Limitations

An important question is whether the method performs equally well for users with different eye geometries.

Human eyes vary in corneal curvature, eye orientation, eyelid anatomy, and other optical characteristics. High myopia is often associated with increased axial length, although this does not necessarily imply a corresponding change in corneal curvature.

Corrective glasses may introduce additional reflections and optical distortions.

These factors could affect reflection visibility or the relationship between measured image features and gaze direction.

Therefore, performance across different refractive conditions should be treated as an important validation question rather than assumed to be identical.

Other practical limitations include insufficient screen brightness, small reflection regions, limited camera resolution, and partial occlusion by eyelids or eyelashes.

### 3.6 Contribution to HCI

HiFiGaze demonstrates how information that is already available within a computing device can improve interaction sensing.

Instead of relying entirely on additional hardware, the system makes use of the display, camera, and known screen content.

Its broader contribution is the possibility of making gaze-based interaction more accessible on ordinary computing devices.

## 4. Kinethreads: Full-Body Haptic Feedback Through Mechanical Force

### 4.1 Research Problem

Visual and auditory feedback are well developed in modern VR systems. Physical force feedback, however, remains difficult to reproduce.

Most consumer haptic suits rely on vibration motors. These can indicate events such as impacts or contact, but they cannot fully simulate the sensation of holding a heavy object or experiencing resistance to movement.

This distinction corresponds to two important concepts:

- **Tactile feedback:** Sensations detected primarily through the skin, such as vibration and pressure.

- **Kinesthetic feedback:** Sensations related to forces, body movement, muscle effort, and joint position.

Kinethreads focuses on expanding kinesthetic feedback while avoiding the bulk and expense associated with many rigid robotic exoskeletons.

### 4.2 Mechanical Principle: From Motor Torque to Cable Tension

The system uses motor-driven strings and pulley mechanisms to apply forces to different parts of the body.

A motor generates rotational torque, which is converted into cable tension through a reel.

For a simplified ideal system,

\[F=\frac{\tau}{r},\]

where \(F\) is cable tension, \(\tau\) is motor torque, and \(r\) is the effective reel radius.

For example, a torque of 0.4 N·m applied to a reel with a radius of 0.01 m would produce an ideal cable tension of 40 N.

This is an illustrative calculation, not a reported measurement from Kinethreads.

Pulley arrangements can redirect forces and provide mechanical advantage. In an idealized pulley system, the output force may increase at the expense of cable displacement and speed.

This reflects the principle of conservation of energy: mechanical advantage does not create additional energy.

### 4.3 System Design

Kinethreads distributes motor-pulley mechanisms across a soft wearable suit.

Instead of using rigid mechanical arms to control the user's joints, it applies tension through flexible strings.

The researchers explored different pulley configurations to redirect, distribute, and amplify forces. For example, compound pulley arrangements can distribute loading across larger body regions while increasing mechanical advantage.

The system can generate several categories of effects, including simulated object weight, body compression, acceleration-related forces, and vibration.

This makes it possible to combine multiple kinds of haptic feedback in a single wearable system.

### 4.4 Experimental Performance

The research prototype demonstrates several notable characteristics:

- **Weight:** Less than 5 kg.

- **Hardware cost:** Approximately US$400.

- **Maximum reported force:** Up to 120 N in a specified configuration.

- **Wearing time:** Less than 30 seconds.

The reported 120 N capability is associated with a mechanically advantageous pulley configuration; it should not be interpreted as a force continuously available at every point on the body.

The researchers also conducted performance evaluations and a user study to investigate the system's capabilities.

These results are significant because they demonstrate a relatively lightweight and inexpensive approach to force feedback, rather than relying on a large rigid exoskeleton.

### 4.5 Fundamental Mechanical Limitations

The main physical limitation is that a flexible cable can generate tension but cannot directly generate compression.

A cable can pull the user's arm in a particular direction, but it cannot push the arm outward in the same way as a rigid actuator.

Consequently, the system cannot reproduce arbitrary three-dimensional forces.

The available force directions also depend on the attachment points, cable routing, and body posture.

Additional engineering challenges include mechanical friction, actuator response, power consumption, comfort, and the need to prevent excessive forces on the user.

### 4.6 Contribution to HCI

Kinethreads expands the concept of computer output beyond visual, auditory, and vibration-based feedback.

It demonstrates how mechanical systems can communicate information directly through physical forces.

For immersive interaction, this could improve the perceived weight of virtual objects, the sensation of movement, and the physical realism of simulated environments.

## 5. Discussion: A Shared Engineering Philosophy

Although EclipseTouch, HiFiGaze, and Kinethreads address different problems, they share a common research philosophy: using physical principles to create new channels of communication between humans and computers.

EclipseTouch uses infrared shadow geometry to detect physical contact. HiFiGaze transforms optical reflections into gaze-related information. Kinethreads converts motor-generated mechanical forces into bodily feedback.

The first two projects focus primarily on sensing human actions, while the third focuses on generating physical responses.

Together, they illustrate a broader interaction loop:

**Human action → Physical sensing → Computational interpretation → System response → Human perception**

This perspective is especially interesting because it connects HCI with optics, mechanics, electronics, and computational modeling.

From my perspective, one of the most valuable lessons is that new interfaces do not always require entirely new physical principles. Sometimes the innovation comes from recognizing that an existing phenomenon contains information that has not yet been effectively used.

For example, a shadow can reveal contact geometry, and a screen reflection can provide clues about gaze direction.

However, an elegant physical principle alone is insufficient for a successful interface. Practical systems must also account for environmental variation, individual differences, usability, reliability, cost, and comfort.

Therefore, evaluating an HCI technology requires both engineering performance measurements and consideration of the actual user experience.

## 6. Conclusion

Chris Harrison's recent research demonstrates how sensing and actuation technologies can extend human-computer interaction beyond conventional input and output devices.

EclipseTouch explores touch sensing on ordinary surfaces, HiFiGaze improves gaze estimation through screen reflections, and Kinethreads provides distributed force feedback through wearable mechanical systems.

Their shared contribution is the transformation of physical phenomena into practical interaction techniques.

These projects also illustrate an important direction for future HCI research: creating interfaces that are not confined to traditional screens and controllers, but are increasingly integrated with human perception, movement, and the surrounding physical world.

## References

1. Future Interfaces Group, Carnegie Mellon University. *Research Projects*. https://www.figlab.com/research

2. Mollyn, V., DeVrio, N., & Harrison, C. (2025). *EclipseTouch: Touch Segmentation on Ad Hoc Surfaces using Worn Infrared Shadow Casting*. Proceedings of ACM UIST 2025. https://doi.org/10.1145/3746059.3747743

3. Kim, T., Mollyn, V., Arakawa, R., & Harrison, C. (2026). *HiFiGaze: Improving Eye Tracking Accuracy Using Screen Content Knowledge*. Proceedings of ACM CHI 2026. https://doi.org/10.1145/3772318.3791339

4. Shen, V., & Harrison, C. (2025). *Kinethreads: Soft Full-Body Haptic Exosuit using Low-Cost Motor-Pulley Mechanisms*. Proceedings of ACM UIST 2025. https://doi.org/10.1145/3746059.3747755
