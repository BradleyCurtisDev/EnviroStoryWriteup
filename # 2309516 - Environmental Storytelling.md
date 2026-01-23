# 2309516 - Environmental Storytelling

![Image of factory floor](FactoryFloor.png)

## Exploring (Research & Inquiry)


### Concept & Inspiration


*   **Mood Boards**:
    *   As a group we put together a Figma board which has mood boards for the factory environment, UI elements such as widgets and fonts within the original game and a storyboard of how we wanted the initial cutscene to play out. 
    *   [Link to Figma board](https://www.figma.com/board/1UYfQkvr74c0H5pE9FhWsr/Environmental-Storytelling?node-id=0-1&t=lBZoQASadgRYAqbQ-1)

    ![Image of Figma board](FigmaBoard.png)

*   **Genre Analysis**:
    *   At the start of the first week we each took time to look at how environments tell stories within existing games to try and emulate a similar feeling. Games that came to mind were *Control* and *What remains of Edith Finch*. 

    ![Control Image](ControlFloatingBodies.webp)
    https://www.hollywoodreporter.com/movies/movie-features/control-game-review-1234007/

    ![Edith Finch Image](WhatRemainsImage.avif)
    https://www.theguardian.com/technology/2017/apr/26/what-remains-of-edith-finch-review-giant-sparrow


    *   In Control the environment is used to build a sense of corporate dread and surrealism through scattered memos and office layouts that feel both mundane and impossible. You might find a simple sticky note that reveals a terrifying supernatural containment failure which makes the cold concrete hallways feel alive and dangerous. On the other hand Edith Finch uses every inch of the family house to tell a more intimate and personal history. Each bedroom is a time capsule tailored to a specific character where the posters on the walls and the clutter on the floor explain who they were without ever needing a line of dialogue. While Control uses the world to explain its complex lore and mechanics Edith Finch uses it to create an emotional connection to a family that is already gone.
*   **Mechanic Research**:
    *   When coming up with the puzzle ideas and storytelling elements I had to look into different ways a character such as a robot could interact with the environment without using common methods such as waypoints. Games like *No Mans Sky*, *Cyberpunk 2077* and *Death Stranding* all use a scanning feature to reveal hidden details about the environment and objects. 

    ![No Mans Sky Scanner](NMSScanner.webp)
    https://nomanssky.fandom.com/wiki/Analysis_Visor

    *   In *No Man's Sky*, the scanner is a multi-tool utility that pulses a local area to highlight nearby resources, mission objectives, and points of interest through a visual HUD.

    ![Cyberpunk Scanner](CyberpunkScan.png)
    https://interfaceingame.com/screenshots/cyberpunk-2077-scan-results/

    *   In *Cyberpunk 2077*, the scanner allows you to identify interactive objects, hackable devices, and lootable containers within your field of vision. It also provides a tactical overlay of NPCs, revealing their vulnerabilities, resistances, and any active bounties or "Wanted" statuses.

    ![Death Stranding Scanner](DeathStrandingScan.jpg)
    https://www.kojimaproductions.jp/en/DSTips

    *   In *Death Stranding*, the Odradek Scanner is a pulse from the players location that analyzes the landscape to identify traversable paths, marking hazards like deep water or steep slopes with color-coded icons (blue, yellow, and red).

    *   I preferred the feel of the scanner in *Death Stranding* because it provided a better sense of spacial awareness to the player especially as we decided on the game having a darker and more somber lighting scheme. I wanted the scan to also be able to highlight specific objects in the environment more closely to how *Cyberpunk 2077* does it with the deep red outlines. Finally I wanted the scan to also flow over objects as it expands outwards from the players location.

## Making (Production & Iteration)


The core mechanics of the environment were built to support the narrative of discovery and interaction with the abandoned facility. The technical implementation focused on a modular "Scanner" system and a series of interconnected puzzle elements.

### 1. The Scanner Mechanic

The primary tool for navagating the environment is the Scanner. The scanner blueprint manages both the visual effect and the gameplay interaction.

*   **Logic**: The scanning process is driven by a custom event `DoScan`, which plays the `ExpandScan` Timeline. This timeline controls the growth of the scan radius over time.

![BP_Scanner](ScannerBlueprint.png)

![BP_Scanner Timeline](ScannerTimeline.png)

*   **Visuals**: The timeline updates the actor's scale (`SetActorScale3D`) to simulate the expanding wave. Simultaneously, it drives a Scalar Parameter `Scan_Opacity` on the material `M_Scan`. This material uses a Depth Fade and Emissive Color (`Scan_Glow`) to create the holographic, "x-ray" aesthetic that reveals hidden geometry.

![M_Scan Material](M_Scan.png)


*   **Interaction**: To detect objects, the scanner uses a `Sphere` component. On `OnComponentBeginOverlap`, it checks if the overlapping actor implements the `BPI_Scannable` interface. If it does, the `OnScanned` function is called on that actor, allowing for distinct behaviors for different object types without hard references.

![Scan In Action](ScanInAction.png)

### 2. Interactive Objects

To bridge the gap between "looking" and "doing", I implemented a physics-based interaction system in `BP_ThirdPersonCharacter`. This allows the player to pick up specific objects such as the Levers and Tablets scattered around the environment.

*   **Trace & Grab Mechanics**:
    *   **Input**: The interaction is triggered by the `Left Mouse Button` 
    *   **Detection**: A `LineTraceByChannel` is fired from the camera's center forward vector. This ensures the player picks up exactly what they are looking at.
    *   **Physics Handle**: If the trace hits a valid physics object, the `PhysicsHandle` component is activated. `GrabComponentAtLocationWithRotation` attaches the object to the character's `HoldLocation` scene component.
    *   **Tick Update**: In the `ReceiveTick` event, the system continuously updates the Physics Handle's target location (`SetTargetLocationAndRotation`) to match the `HoldLocation`. This creates a smooth, laggy "towing" effect where the object physically follows the player's view rather than snapping rigidly.

    ![Third Person Character Interact](BP_ThirdPersonCharacterBlueprint.png)
*   **Interface Implementation**: `BP_Lever` implements the `BPI_Scannable` interface.

*   **Feedback**: When the `OnScanned` event is triggered by the scanner, the lever executes its specific feedback logic. In this case, it calls `SetOverlayMaterial` on its Static Mesh, applying `M_HighlightItem`. This visual cue immediately informs the player that the object is significant and interactive.
    ![BP_Lever Event Graph showing OnScanned event and SetOverlayMaterial]

### 3. The Puzzle Loop (Generator & Crane)

![Image of early puzzle](EarlyCrane.png)

![CranePuzzleFinal](CranePuzzleFinal.png)


*   **The Generator (`BP_Generator`)**:
    *   **Puzzle Logic**: The actor functions as a "counter" using a Trigger Volume. It casts overlapping actors to `BP_Lever` and increments an integer `ItemsOnPad`. Initially the generator started as a Pressure pad in which you put boxes onto but to better fit the theme of the environment I changed it to be levers that are replaces on a control panel which then allow the crane to function.

    ![Generator Blueprint](GeneratorBlueprint.png)

    ![Generator blueprint view](GeneratorBlueprintView.png)


*   **Visual Feedback**: To communicate progress without UI, the blueprint toggles the visibility of static mesh components (`Lever1`, `Lever2`, `Lever3`) as the count increases. This diegetic feedback confirms successful placement to the player.

    ![Generator with levers in](LeversInGenerator.png)

*   **Consistency**: Like `BP_Lever`, the Generator implements `BPI_Scannable`. Its `OnScanned` event applies the same `M_HighlightItem` overlay, establishing a consistent visual language for interactable objects.


*   **The Crane (`BP_Crane`)**:
    *   As soon as the generator has all three levers in place it will call the `StartRotation` event on the crane. Upon receiving the `StartRotation` call, it sets a boolean `bShouldMove`.
    *   **Interpolation**: In `ReceiveTick`, it uses `RInterpTo` (Rotation Interpolation) to smoothly rotate the arm towards `TargetRotation` at a defined speed. This avoids jarring "snapping" movement, preserving immersion.
    ![BP_Crane Event Graph showing ReceiveTick and RInterpTo](CraneBlueprint.png)

### 4. UI 

*   **New Area Popups**: In Stray, when the player enters a new area a popup will appear in which it has the areas name using the games unique font as well as an English translation below. For most of the 4 weeks we did not have a popup like this and the game felt a bit empty but after adding it in, it really helped a sense of discovery and progression.

    ![New Area Popup](StrayAreaPopup.jpg)
    https://interfaceingame.com/screenshots/stray-new-zone/

    ![Ingame Area Popup](FactoryFloor.png)
    *   **Implementation**:
        *   **Trigger (`BP_OfficeUI`)**: I created an actor that functions as a trigger. On `ReceiveActorBeginOverlap`, it checks if the overlapping actor is the player (`BP_ThirdPersonCharacter`). If confirmed, it creates the `WBP_Office` widget, adds it to the viewport, and immediately calls `DestroyActor` on itself. This ensures the popup only triggers once per playthrough.
        *   **Widget Logic (`WBP_Office`)**: The widget handles its own lifecycle to keep the implementation clean.
            *   **Construct**: When created, it enters a `Delay` of 5 seconds, keeping the text fully visible.
            *   **Animation**: After the delay, it plays the `FadeOut` animation, which modulates the `RenderOpacity` from 1 to 0.
            *   **Cleanup**: I used the `OnAnimationFinished` event to call `RemoveFromParent`, ensuring no invisible widgets remain in memory.

            ![Popup Blueprint](AreaPopupGraph.png)

            ![Popup Animation](AreaPopupAnim.png)

*   **Credits Screen (`BP_Credits`)**:
    *   **Auto-Scroll**: I implemented a scrolling mechanic in the `Tick` event. It takes the current `ScrollOffset` of the main ScrollBox and adds a `ScrollSpeed` variable multiplied by `DeltaTime`. This creates a smooth, frame-rate-independent rolling effect typically seen in films.
    *   **Concept Art Slideshow**: To make the credits visually engaging, I added a dynamic background.
        *   **Logic**: On `Construct`, a Timer is set to fire every 3 seconds.
        *   **Randomization**: The `CycleConceptArt` event picks a random texture from an `ImageArray` and updates the background image (`SetBrushFromTexture`). This showcases the team's concept art alongside their names.
    *   **Navigation**: A simple "Back" button allows the player to close the credits via `RemoveFromParent`, returning them to the main menu.

    ![Credits Screen Blueprint](CreditsBlueprint.png)

    ![Credits Screen](Credits.png)


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


### References

- Turbulent_Sun_9378 (2025) Scanning ability in 10 minutes! : ). [Reddit Post] At: https://www.reddit.com/r/unrealengine/comments/1m9dzu9/scanning_ability_in_10_minutes/ (Accessed  23/01/2026).

- New zone - Stray (s.d.) At: https://interfaceingame.com/screenshots/stray-new-zone/ (Accessed  23/01/2026).

- Ramanan, C. (2017) 'What Remains of Edith Finch review: magical ode to the joy of storytelling' In: The Guardian 26/04/2017 At: https://www.theguardian.com/technology/2017/apr/26/what-remains-of-edith-finch-review-giant-sparrow (Accessed  23/01/2026).

- Keeley, P. (2019) ‘Control’: Game Review. At: https://www.hollywoodreporter.com/movies/movie-features/control-game-review-1234007/ (Accessed  23/01/2026).

- Analysis Visor - No Man’s Sky Wiki (s.d.) At: https://nomanssky.fandom.com/wiki/Analysis_Visor (Accessed  23/01/2026).

- DEATH STRANDING Tips: Connecting the Eastern Region | Kojima Productions (s.d.) At: https://www.kojimaproductions.jp/en/DSTips (Accessed  23/01/2026).

- Scan results - Cyberpunk 2077 (s.d.) At: https://interfaceingame.com/screenshots/cyberpunk-2077-scan-results/ (Accessed  23/01/2026).

- 