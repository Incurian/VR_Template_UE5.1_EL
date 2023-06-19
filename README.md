# VR_Template_UE5.1_EL

# Table of Contents

- [VR_Template_UE5.1_EL](#vr_template_ue51_el)
  - [Introduction](#introduction)
  - [Framework](#framework)
  - [Game Mode: GMode_22](#game-mode-gmode_22)
  - [Player Controller: PC_VR_22](#player-controller-pc_vr_22)
  - [Game Instance: GInstance_22](#game-instance-ginstance_22)
  - [Pawn: Pawn_VR_22](#pawn-pawn_vr_22)
    - [Scenes](#scenes)
    - [Events](#events)
    - [Incomplete list of potential Pawn behavior](#incomplete-list-of-potential-pawn-behavior)
  - [Buttons](#buttons)
    - [Button Component](#button-component)
    - [Routing Singleton](#routing-singleton)
    - [Remote Target Helper](#remote-target-helper)
    - [Any Remote Target Actor](#any-remote-target-actor)
  - [Items](#items)
    - [Item Coordinator](#item-coordinator)
    - [Item Subcomponent [logic] (SCL)](#item-subcomponent-logic-scl)
    - [SCL States](#scl-states)
    - [SCL Relations](#scl-relations)
    - [SCL Functions](#scl-functions)
    - [Other SCL Configuration](#other-scl-configuration)
    - [HAND PLACEMENT AID](#hand-placement-aid)
    - [MANIPULATION CONSTRAINT](#manipulation-constraint)
    - [ATTACHMENT REFERENCE](#attachment-reference)


For the Epic Launcher version of Unreal Engine 5.1.

If you have a source built version of UE5.1, try the other template - it comes with a little extra stuff, and is required for Dedicated Servers - [VR_Template_UE5.1_SB](https://github.com/Incurian/VR_Template_UE5.1_SB)

## Introduction

Note to self: remove keys before publishing Do not remove this note, you'll forget next time

Get help from our discord server any time: [Discord Server](https://discord.gg/ANt8bbh3rA)

VR is a hard problem. There are too many ways to move and too few ways to stop the player from moving; any solutions need to be computationally cheap and not make the player sick. VR gives you effortless presence and intuitive interactions, but it makes breaks from reality all the more jarring. Making the world too much like reality can backfire because the player's ability to interact with it is significantly handicapped compared to reality; perfect simulations that don't feel right are counter-productive. It needs to be as realistic as can be experienced properly, but not one iota more.

The solutions are made more difficult by my decision to use blueprint scripting exclusively, but I think it's worthwhile because blueprints are surprisingly powerful considering how easy they are to use. And I don't want to learn C++.

As you look through this project you'll have questions about the choices I made. Here are the answers in advance:

<ol type="a">
  <li>I don't know a lot of computer science and my programming skill is minimal.</li>
  <li>I didn't think of it.</li>
  <li>I tried it your way but it didn't work exactly the way I wanted.</li>
  <li>I was trying to work around a bug which may or may not exist anymore but it's too late to go back.</li>
  <li>I don't know a lot of math.</li>
  <li>I didn't know I could do that.</li>
  <li>It's too computationally expensive the other way.</li>
  <li>There are a lot of edge cases and I want player movement to feel as intuitive as the hardware allows.</li>
  <li>I definitely sometimes overfit to an edge case and then just leave it because I'm lazy.</li>
  <li>I am a visual thinker.</li>
  <li>I am just dumb sometimes.</li>
  <li>I forget why.</li>
  <li>I'm not sure but if I change it the compiler threatens to destroy the universe.</li>
  <li>I copied this piece verbatim from a forum post from 2016 that I can't find anymore.</li>
  <li>I was optimizing for a different system.</li>
  <li>I did what?!</li>
  <li>I couldn't think of a better way using blueprints.</li>
</ol>


Please report bugs or opportunities to improve the blueprints. I'm begging you.

This will be more of a quick-start guide than a detailed manual for each function. I will try to make the comments within the blueprints explanatory, and make it easy to switch common options. The idea is that you will know that a certain function exists and where, if not necessarily how it works. If you have any questions please ask.

## Framework

Ensure these are selected either in `ProjectSettings->Maps&Modes`, or in `WorldSettings->GameModeOverride`. Note that there are custom collision channels, and they are required. About half of all bugs are from improper collision settings.

References:
- [Gameplay Framework Quick Reference in Unreal Engine](https://docs.unrealengine.com/5.1/en-US/gameplay-framework-quick-reference-in-unreal-engine/)
- [Game Mode and Game State in Unreal Engine](https://docs.unrealengine.com/5.1/en-US/game-mode-and-game-state-in-unreal-engine/)

## Game Mode: GMode_22
- Accepts new connections from controllers, spawns a pawn, orders controller to possess pawn
- Spawns the "router" which helps certain blueprints find and talk to each other

## Player Controller: PC_VR_22
- Sets variables in Pawn and PC so they can reference each other
- Checks whether the VR setup has already been completed this instance
- Forces VR Setup if necessary, saves setup information locally and in the Game Instance, then prompts the Pawn to continue further initialization stuff

## Game Instance: GInstance_22
- Holds the variable for Pawn VR setup once completed so it doesn't need to be repeated.

## Pawn: Pawn_VR_22
### SCENES
- RootBox: Represents the origin of your VR playspace in realspace. It moves everything about the pawn with it, including tracked components. For 360 locomotion, this component moves and attaches itself to the ground.
- VROO Box: The VR Origin Offset, the Pawn's origin in virtual space. Separates the VR Origin from the RootBox to allow recentering the pawn without invoking XR API functions.
- Camera: Tracked input that automatically updates its transform to match HMD movement in realspace.
- Motion Controllers: MC_Left and MC_Right, tracked components representing left and right hands. Each has a "ghost" hand and a "solid" hand for visualization and interaction purposes.

### EVENTS
- StartUp: Initializes variables, tracks initial data, and sets scene components to starting positions after the pawn is possessed.
- VRMenu: Handles VR-specific setup functions and currently houses a VR menu for setup control.
- EventGraph: Contains Tick event and handles various actions based on different phases (Pre, During, Post) and input processing.

### Incomplete list of potential Pawn behavior:
- Reset Orientation and Position without XR API call
- Float in zero g, moving with thrusters or by pushing off from walls with hands
- Switching from floating to walking when coming into contact with a floor
- Climbing walls, vaulting over walls
- 360 Locomotion over various terrain
- Edge detection to prevent walking off cliffs
- Collision detection to prevent walking through objects
- Lean over a counter or obstacle without collision interference
- Seated pawn with constraints
- Jumping, walking along with a moving floor, smooth hand animations, hand interactions, etc.

## Buttons
### BUTTON COMPONENT
- Represents a button that can be pressed, shows its state, and triggers a function.
- Requires setting the "Target Input" struct with function name, target name, and target method.

### ROUTING SINGLETON
- Connects buttons with their target functions in actors, assigns IDs, and manages function status.

### REMOTE TARGET HELPER
- A scene component that enables triggering functions on remote actors.
- Requires parent actor input on start and communicates with the router.

### Any Remote Target Actor
- Actor with remote target functionality needs to implement the Interface_Remote Functions interface.
- Receives messages from the router to trigger remote functions.

## Items
### ITEM COORDINATOR
- Required for every item actor, facilitates communication between subcomponents and manages state changes.

### ITEM SUBCOMPONENT [logic] (SCL)
- Logic scene components that control specific subcomponents of an item.
- Each SCL has states, relations, functions, and constraints to define functionality.

## SCL States
- Resting: item is left alone
- Null: not used
- Attached: the item is attached to another item actor (think magazine in a rifle)
- Held: the item is held by a pawn
- Manipulated: a piece of the item is being moved by a pawn, but cannot necessarily be picked up and moved around (think moving handles or a foregrip that stay connected to their parent while moved)

Only states you intend to use should be added to the struct. Within each state, there are a variety of relations, which act as permission classes.

## SCL Relations
- Owner: the pawn currently using the primary component (but NOT the component in question), OR a parent SCL the component is attached to
- User: the actual hand currently using this component
- Offhand: the other hand of the pawn currently using this component
- Public: not otherwise currently related

So we have told the component "If you are in STATE, and RELATION attempts to use this component..." The next level of the struct is the actual function we want the component to be capable of when it is in that state and approached by something trying to use it. There should only be one function per state:relation pair. They generally result in a state change, and may also trigger a new SCL to take over.

## SCL Functions
- Pickup: the item is held by the requesting hand
- Drop: the item is dropped by the holding hand because the grip button has been released
- Drop Sticky: the item has been dropped by the holding hand because the grip button has been released and then pressed again (sticky items do not require the player to hold the grip button to continue holding them)
- Manipulate: the component now may be moved within constraints while staying attached to its parent by the requesting hand
- Release: the component is no longer being manipulated because the player released the grip button
- Release Sticky: [empty string] and then pressed the grip button again
- Activate Non Diagetic: For triggering an event without actually touching the item
- Trigger Actions: Not yet implemented, but would call a non-pickup/manipulate type function like pulling a trigger
- Other Actions: [empty]
- Dry Grip: Not used during setup, but is sent from the hand to facilitate sticky things somehow

Additionally, there are optional constraints for each function:
- Block: do not allow this function to activate
- Prox Only: the requesting hand must be within close proximity to allow the function
- Other Checks: not yet implemented, but is a freeform checklist you can create for complicated permissions
- New Logic: optional, but it's the name of the SCL that should take over if this function is allowed

Note: you may want to switch to a new logic when a function/state change requires additional changes (e.g., a different highlighter box), more below.

Note: Practically any time an SCL does something, it fires an event on the parent actor (if it has the Item Interaction Interface) to let it know. You can use this to have the item do more complicated things not already contained within the SCLs.

## Other SCL Configuration
- Primary: already discussed.
- Attachment Info: should be fairly self-explanatory, see the examples.
- Is foregrip/reargrip: on a rifle this causes each hand to control a different aspect of the transform

There are three more helper scene components that can optionally be attached (they should find each other automatically) to SCLs.

### HAND PLACEMENT AID
This allows you to easily tell the SCL where the holding/manipulating hand (one each for left and right) should be relative to the component by placing them in the viewport.

### MANIPULATION CONSTRAINT
A box you can place and scale within the viewport to easily tell a manipulatable component where it may move relative to its parent.

### ATTACHMENT REFERENCE
Any scene component attached to an SCL with attachment enabled, created by adding "attm" or "attf" tags (for male and female respectively) to the scene component. Allows you to use the viewport to fine-tune exactly where items attach to one another.

