---
title: "HCI Homework 3 — Enhancing Human Abilities in the Age of Augmented Reality"
date: 2026-11-01
publishDate: 2026-11-01
lastmod: 2026-11-01
draft: false
description: "Lecture 7–9: AR ideation, an interactive optical learning prototype, and qualitative user evaluation."
tags: ["HCI", "Augmented Reality"]
categories: ["Human-Computer Interaction"]
math: true
toc: true
---

## Lecture 7 — Ideation

### Concept 1 — AR Optical Learning Assistant

An AR-based learning assistant helps students understand optical principles by overlaying virtual light rays, optical axes, focal points, and focal lengths onto real optical setups. As users change the object-to-lens distance, the system updates the ray diagram to illustrate image formation.

### Concept 2 — AR Indoor Navigation Assistant

An AR indoor navigation assistant helps users find their way through unfamiliar buildings. After entering a destination, users see directional arrows, floor information, and classroom labels overlaid onto the real environment through their smartphone cameras.

### Concept 3 — AR Real-Time Communication Assistant

An AR real-time communication assistant supports face-to-face conversations across language barriers. Through AR glasses, users can see translated subtitles positioned near the speaker, allowing them to maintain eye contact while understanding the conversation. An optional captioning mode can also support users with hearing difficulties.

{{< hw3-figure src="images/hci-hw3/concept-sketches.png" caption="Figure 1. Concept sketches for the optical learning, indoor navigation, and real-time communication assistants. Original bilingual annotations are preserved." >}}

## Lecture 8 — Interactive Prototype

**[Launch Interactive AR Prototype]({{< ar-optics-url >}})**

### 1. Introduction

For this assignment, I selected the AR Optical Learning Assistant from the three concepts proposed in Lecture 7 and developed an interactive prototype.

The purpose of this application is to help students understand geometrical optics by displaying virtual optical information over a real-world experimental environment.

In traditional optics education, students often learn about image formation through textbook diagrams and mathematical equations. However, it can be difficult to connect these abstract representations with physical optical experiments, especially because the paths of light rays are not normally visible.

My proposed AR application addresses this problem by allowing students to visualize light rays, focal points, and image formation directly within their view of an optical experiment.

### 2. Prototype Design

I developed a simulated AR prototype using HTML, CSS, and JavaScript.

Instead of implementing a complete AR system with a camera and spatial tracking, I used an AI-generated, photorealistic optical laboratory scene as the background. Interactive virtual elements are overlaid on this scene to simulate what a user might see through AR glasses.

The prototype includes three main components:

- Real-world scene simulation: An optical bench containing a convex lens, an object, and an image screen.
- AR visualization layer: Virtual light rays, focal points, an optical axis, and optical annotations displayed over the experimental scene.
- Interactive interface: Controls that allow users to modify the object distance, display or hide optical information, and reset the experiment.

This approach allows users to experience the proposed interaction without requiring specialized AR hardware.

### 3. Interactive Features

#### 3.1 Dynamic Ray Visualization

The prototype displays virtual light rays passing through a convex lens.

These rays illustrate how light from an object is refracted by the lens and converges to form an image.

Users can enable or disable the ray visualization to compare the experimental scene with and without the additional optical information.

#### 3.2 Adjustable Object Distance

Users can adjust the object distance using an interactive slider.

The prototype uses a convex lens with a fixed focal length of 10 cm. The object distance can be adjusted between 15 cm and 40 cm.

When the object distance changes, the system recalculates the image distance and magnification and updates the virtual ray diagram.

The calculations are based on the thin lens equation:

$$
\frac{1}{f}=\frac{1}{u}+\frac{1}{v}
$$

and the magnification equation:

$$
m=-\frac{v}{u}
$$

where:

- \(f\) is the focal length.
- \(u\) is the object distance.
- \(v\) is the image distance.
- \(m\) is the lateral magnification.

This interaction allows users to explore how changing the position of an object affects the location and size of its image.

#### 3.3 Optical Information and Labels

The prototype provides additional information about the optical system, including the focal length, object distance, image distance, magnification, and image properties.

Users can show or hide optical labels and formulas according to their needs.

This feature is intended to connect visual observations with the corresponding physical concepts and mathematical relationships.

#### 3.4 Language Selection and Reset

The interface supports both English and Chinese, making the prototype accessible to users with different language preferences.

A reset button allows users to return the experiment to its initial configuration.

### 4. Example Interaction

Consider a convex lens with a focal length of 10 cm.

When the object is placed 30 cm from the lens, the calculated image distance is 15 cm and the magnification is −0.5.

The image is therefore real, inverted, and reduced.

If the user moves the object to 20 cm from the lens, the image distance becomes 20 cm and the magnification becomes −1.

The image is now real, inverted, and the same size as the object.

Through the interactive visualization, users can observe these changes while exploring the relationship between object distance and image formation.

### 5. Why This Is an AR Prototype

The main concept of the application is to enhance the user's perception of a physical optical experiment by adding information that would otherwise be invisible.

In the proposed final application, AR glasses would overlay calculated light paths, focal points, and optical parameters onto real experimental equipment.

The current prototype simulates this experience by combining a realistic laboratory background with interactive virtual graphics.

It is important to clarify that this is a simulated AR prototype, not a fully implemented AR application. It does not perform real-time object recognition, camera tracking, or automatic spatial registration.

Its purpose is to demonstrate the interaction concept and provide an initial experience for user evaluation.

### 6. Prototype Limitations and Future Improvements

The current prototype has several limitations.

First, the experimental background is a fixed, AI-generated image rather than a live camera feed. Therefore, the application cannot respond to physical movement of real laboratory equipment.

Second, the virtual optical elements are positioned within a predefined simulated scene rather than automatically aligned with detected real-world objects.

Third, the optical model uses an ideal thin lens and does not account for effects such as lens aberrations or real experimental measurement errors.

Future versions could incorporate a camera feed, object recognition, and spatial tracking to align virtual information with real optical components. Additional optical elements could also be introduced to support more complex experiments.

Despite these limitations, the prototype provides an initial interactive demonstration of how AR could support optics education.

### 7. Conclusion

Through this prototype, I explored how augmented reality could make abstract optical concepts easier to visualize and understand.

By combining an experimental scene with interactive ray diagrams and optical information, the application aims to connect theoretical knowledge with physical observation.

The prototype is intended for an initial user experience rather than a complete technical implementation.

In Lecture 9, I will use this prototype for user evaluation to investigate whether the proposed visualization and interactions are understandable and useful.


{{< hw3-figure src="images/hci-hw3/prototype-30cm.png" caption="Figure 2. Supplied prototype screenshot at an object distance of 30 cm: image distance 15 cm and magnification −0.50." >}}

{{< hw3-figure src="images/hci-hw3/prototype-40cm.png" caption="Figure 3. Supplied prototype screenshot at an object distance of 40 cm: image distance approximately 13.3 cm and magnification −0.33. These screenshots document the evaluated interface before minor label-position fixes." >}}

## Lecture 9 — User Evaluation

### 1. Evaluation Objective
Following the development of the interactive prototype in Lecture 8, I conducted an initial user evaluation of the AR Optical Learning Assistant.

The objective was to understand how a potential user perceived the concept, whether the interactive visualization was useful, and what limitations became apparent during the evaluation.

Particular attention was given to the relationship between the user's expectations of an AR learning assistant and the actual experience provided by the prototype.

### 2. Evaluation Method
I invited one participant to evaluate the prototype and discuss its design.

The evaluation focused on the application's purpose, virtual ray visualization, interaction methods, interface clarity, and potential use in real optical laboratories.

Five open-ended questions were used to collect feedback. Additional discussion emerged naturally, particularly regarding the limited functionality of the prototype and the accessibility of future AR glasses.

The evaluation was qualitative, focusing on the participant's opinions and reactions rather than quantitative performance measurements.

### 3. Observations and User Feedback

#### 3.1 Initial Expectations and Overall Experience
The participant initially expressed considerable interest in the proposed AR concept and was looking forward to its potential applications.

However, after experiencing the prototype, the participant was somewhat disappointed by its limited functionality.

This contrast between initial expectations and actual experience was an important observation. Although the concept was appealing, the current prototype did not yet provide the range of capabilities the participant expected from an AR learning assistant.

#### 3.2 Understanding the Concept and Visualization
When asked about the application's purpose, the participant explained that it helped visualize how image formation through a convex lens changes with object distance.

The participant also found the virtual ray overlays somewhat helpful, explaining that they recalled optics knowledge from middle school and made the light paths more noticeable than in a real experiment.

These responses suggest that the main concept was understandable and that the visualization could support the recall of previously learned optical principles.

#### 3.3 Interaction and Interface Problems
The participant appreciated the ability to adjust numerical parameters directly by dragging the controls, describing this interaction as convenient.

However, several problems were identified.

First, the participant did not understand the meaning of the formulas displayed in the upper-left and lower-right areas of the interface or how they related to the visualization.

Second, some text in the upper-left area was obstructed, reducing readability.

Finally, the participant considered the available functions too limited.

These findings indicate that the prototype needs clearer explanations, improved information layout, and a broader range of interactions.

#### 3.4 Potential Applications in Real Laboratories
Despite the limitations of the current prototype, the participant expressed willingness to use a more developed version through real AR glasses.

They suggested that visual guidance could help students adjust optical paths and become familiar with experimental procedures more quickly.

For practical optical alignment, the participant considered this approach potentially more useful than conventional text-based AI guidance.

This feedback suggests an opportunity to develop the application beyond basic optics visualization into an interactive laboratory assistant.

#### 3.5 Accessibility Discussion
After answering the evaluation questions, the participant raised an additional concern: whether people with myopia would be able to use AR glasses comfortably and effectively.

This led to a discussion about possible display technologies, including retinal projection, and how future AR systems might accommodate different visual needs.

Following the discussion, the participant became more optimistic about the possibility of addressing this issue.

However, the conversation did not establish that retinal projection would eliminate the need for vision correction. The concern remains an important accessibility consideration for future development.

### 4. Critical Reflection

#### 4.1 A Promising Concept Requires Meaningful Functionality
The participant's initial enthusiasm, followed by disappointment with the limited prototype, revealed a gap between the proposed concept and the implemented experience.

This suggests that an attractive AR concept alone is not sufficient. The prototype must also demonstrate interactions that users consider meaningful.

Future iterations should therefore focus on practical experimental tasks rather than simply adding more visual information.

#### 4.2 Visualization Must Be Connected to Understanding
Although the virtual rays were helpful, the participant struggled to interpret the formulas.

This highlights the difference between presenting information and supporting understanding.

A future version could connect each formula to corresponding elements in the optical scene, highlight relevant distances, and explain changes interactively.

#### 4.3 Interface Readability and Accessibility Matter
The obstructed text revealed a basic interface design problem that should be addressed before adding more functions.

The discussion about myopia also demonstrated that accessibility needs to be considered from the early design stages.

Future AR implementations should investigate compatibility with prescription glasses, adjustable optical correction, and different display technologies, rather than assuming that one display solution will work for everyone.

#### 4.4 Limitations of the Evaluation
The current prototype simulates AR using a predefined laboratory scene and virtual overlays. It does not implement real-time tracking or spatial registration.

Therefore, this evaluation cannot establish how the system would perform with physical laboratory equipment or how comfortable it would be to use through real AR glasses.

Furthermore, the evaluation involved only one participant. The findings provide useful qualitative design feedback but cannot be generalized to a larger user population.

### 5. Proposed Improvements
Based on the evaluation, I identified four priorities for future development:

1. **Improve interface clarity:** Reorganize information panels, remove text obstruction, and provide contextual explanations for optical formulas.

2. **Expand practical interactions:** Add optical alignment guidance, adjustable components, and more complex experimental scenarios.

3. **Consider accessibility:** Investigate how users with different visual needs could comfortably use the system, including compatibility with vision correction.

4. **Develop a real AR implementation:** Introduce camera-based tracking and spatial registration to connect virtual optical information with physical equipment.

### 6. Conclusion
This user evaluation revealed both the potential and the limitations of the AR Optical Learning Assistant.

The participant understood the central concept, appreciated direct parameter adjustment, and found the virtual ray visualization somewhat helpful.

However, the limited functionality did not fully meet the participant's initial expectations. The evaluation also identified problems with formula comprehension, interface readability, and accessibility considerations.

An important insight was that the participant valued the possibility of using AR for practical optical alignment, rather than only for visualizing theoretical concepts.

The evaluation therefore suggests that future development should prioritize meaningful laboratory interactions, clearer visual explanations, and inclusive design.

Although the findings are limited to one participant and a simulated AR prototype, they provide concrete directions for improving the next iteration.

---

[Back to HCI coursework]({{< relref "/courses/hci" >}})
