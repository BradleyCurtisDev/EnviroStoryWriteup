# 2309516 - Environmental Storytelling

![Image of factory floor](FactoryFloor.png)

## Exploring (Research & Inquiry)


### Concept & Inspiration
"Where did your ideas come from?"

*   **Mood Boards**:
    *   [Insert description of your PureRef board or initial inspiration]
    *   ![Screenshot of PureRef Mood Board]
*   **Genre Analysis**:
    *   Investigated "Environmental Storytelling" in games like *Gone Home*, *What Remains of Edith Finch*, or *Control*.
    *   Analyzed how they use lighting and object placement to narrate without words.
*   **Mechanic Research**:
    *   Researched scanning mechanics in other games (e.g., *Death Stranding*, *No Man's Sky*).
    *   Reference link: [Scanning Ability in 10 Minutes](https://www.reddit.com/r/unrealengine/comments/1m9dzu9/scanning_ability_in_10_minutes/)
*   **Initial Sketches**:
    *   [Briefly describe your paper prototypes or flowcharts]
    *   ![Image of initial sketches or paper  prototype]



## Making (Production & Iteration)


*   **Material Iteration**:
    *   Refined the `M_Scan` material multiple times to get the right "pulse" speed and glow intensity.
    *   ![Screenshot of an early version of the scan effect]

The core mechanics of the environment were built to support the narrative of discovery and interaction with the abandoned facility. The technical implementation focused on a modular "Scanner" system and a series of interconnected puzzle elements.

### 1. The Scanner Mechanic ("Seeing the Invisible")

The primary tool for environmental storytelling is the Scanner, implemented in `BP_Scanner`. This blueprint manages both the visual effect and the gameplay interaction.

*   **Logic**: The scanning process is driven by a custom event `DoScan`, which plays the `ExpandScan` Timeline. This timeline controls the growth of the scan radius over time.
    ![BP_Scanner Event Graph showing DoScan and Timeline node]
*   **Visuals**: The timeline updates the actor's scale (`SetActorScale3D`) to simulate the expanding wave. Simultaneously, it drives a Scalar Parameter `Scan_Opacity` on the material `M_Scan`. This material uses a Depth Fade and Emissive Color (`Scan_Glow`) to create the holographic, "x-ray" aesthetic that reveals hidden geometry.
    ![Material Graph M_Scan showing Depth Fade and Opacity parameters]
*   **Interaction**: To detect objects, the scanner uses a `Sphere` component. On `OnComponentBeginOverlap`, it checks if the overlapping actor implements the `BPI_Scannable` interface. If it does, the `OnScanned` function is called on that actor, allowing for distinct behaviors for different object types without hard references.

### 2. Interactive Objects (The Lever)

Interactive elements like the Lever (`BP_Lever`) heavily rely on the interface system to decouple them from the player or scanner logic.

*   **Interface Implementation**: `BP_Lever` implements the `BPI_Scannable` interface.
*   **Feedback**: When the `OnScanned` event is triggered by the scanner, the lever executes its specific feedback logic. In this case, it calls `SetOverlayMaterial` on its Static Mesh, applying `M_HighlightItem`. This visual cue immediately informs the player that the object is significant and interactive.
    ![BP_Lever Event Graph showing OnScanned event and SetOverlayMaterial]

### 3. The Puzzle Loop (Generator & Crane)

![Image of early puzzle](EarlyCrane.png)

![CranePuzzleFinal](CranePuzzleFinal.png)


*   **The Generator (`BP_Generator`)**:
    *   **Puzzle Logic**: The actor functions as a "counter" logic gate using a Trigger Volume. It casts overlapping actors to `BP_Box` and increments an integer `ItemsOnPad`.
    *   **Visual Feedback**: To communicate progress without UI, the blueprint toggles the visibility of static mesh components (`Lever1`, `Lever2`, `Lever3`) as the count increases. This diegetic feedback confirms successful placement to the player.
    *   **Consistency**: Like `BP_Lever`, the Generator implements `BPI_Scannable`. Its `OnScanned` event applies the same `M_HighlightItem` overlay, establishing a consistent visual language for interactable objects.
    *   **Trigger**: When `ItemsOnPad >= TargetItems`, it executes the `StartRotation` event on the referenced `CraneActor`.
*   **The Crane (`BP_Crane`)**:
    *   The Crane serves as the reward state. Upon receiving the `StartRotation` call, it sets a boolean `bShouldMove`.
    *   **Interpolation**: In `ReceiveTick`, it uses `RInterpTo` (Rotation Interpolation) to smoothly rotate the arm towards `TargetRotation` at a defined speed. This avoids jarring "snapping" movement, preserving immersion.
    ![BP_Crane Event Graph showing ReceiveTick and RInterpTo]

## Connecting (Collaboration)

### Teamwork & Workflow
"How did you work with the team?"

*   **Communication**:
    *   Describe the use of Discord/Slack for daily communication and "pivot-point" discussions.
    *   ![Screenshot of a key Discord discussion or decision]
*   **Agile Methodology**:
    *   Evidence of Sprint Reviews or Trello/Jira board usage.
    *   ![Screenshot of Trello/Jira board showing task progression]
*   **File Management**:
    *   Adherence to naming conventions (e.g., `BP_Name`, `M_Name`) to ensure seamless integration.
    *   Mention any merge conflict resolutions or version control practices.

## Situating & Synthesizing (Reflection - Context & Conclusion)

### Reflection & Justification
"What is the final result?" and "Why did you make these choices?"

*   **Final Result**:
    *   [Summarize the final look and feel of the project]
    *   ![Before and After comparison screenshots of a key area]
*   **Narrative Justification**:
    *   Explain *why* the scanner was the right choice for this story (e.g., "revealing the past").
    *   Explain how the visual style supports the mood (e.g., "cold, abandoned atmosphere").
*   **Critical Evaluation**:
    *   **Successes**: What went well? (e.g., The modularity of the scanner system).
    *   **Challenges**: What was difficult? (e.g., Optimizing the transparency shaders).
    *   **Future Improvements**: What would you add if you had more time? (e.g., More interactive object types).

- https://www.reddit.com/r/unrealengine/comments/1m9dzu9/scanning_ability_in_10_minutes/