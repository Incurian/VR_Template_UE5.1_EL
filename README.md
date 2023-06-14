# VR_Template_UE5.1_EL

For the Epic Launcher version of Unreal Engine 5.1.  If you have a source built version of UE5.1, try the other template - it comes with a little extra stuff, and is required for Dedicated Servers - https://github.com/Incurian/VR_Template_UE5.1_SB

=Introduction
*
Note to self: remove keys before publishing
Do not remove this note, you'll forget next time
*

Get help from our discord server any time: https://discord.gg/ANt8bbh3rA

VR is a hard problem.  There are too many ways to move and too few ways to stop the player from moving; any solutions need to be computationally cheap and not make the player sick.  VR gives you effortless presence and intuitive interactions, but it makes breaks from reality all the more jarring.  Making the world too much like reality can backfire because the player's ability to interact with it is significantly handicapped compared to reality; perfect simulations that don't feel right are counter-productive.  It needs to be as realistic as can be experienced properly, but not one iota more.

The solutions are made more difficult by my decision to use blueprint scripting exclusively, but I think it's worthwhile because blueprints are surprisingly powerful considering how easy they are to use.  And I don't want to learn C++.

As you look through this project you'll have questions about the choices I made.  Here are the answers in advance:

a) I don't know a lot of computer science and my programming skill is minimal.
b) I didn't think of it.
c) I tried it your way but it didn't work exactly the way I wanted.
d) I was trying to work around a bug which may or may not exist anymore but it's too late to go back.
e) I don't know a lot of math.
f) I didn't know I could do that.
g) It's too computationally expensive the other way.
h) There are a lot of edge cases and I want player movement to feel as intuitive as the hardware allows.
i) I definitely sometimes overfit to an edge case and then just leave it because I'm lazy.
j) I am a visual thinker.
k) I am just dumb sometimes.
l) I forget why.
m) I'm not sure but if I change it the compiler threatens to destroy the universe.
n) I copied this piece verbatim from a forum post from 2016 that I can't find anymore.
o) I was optimizing for a different system.
p) I did what?
q) I couldn't think of a better way using blueprints.

Please report bugs or opportunities to improve the blueprints.  I'm begging you.

This will be more of a quick-start guide than a detailed manual for each function.  I will try to make the comments within the blueprints explanatory, and make it easy to switch common options.  The idea is that you will know that a certain function exists and where, if not necessarily how it works.  If you have any questions please ask.

=Framework

Ensure these are selected either in ProjectSettings->Maps&Modes, or in WorldSettings->GameModeOverride
Note that there are custom collision channels, and they are required.  About half of all bugs are from improper collision settings.

refs:
https://docs.unrealengine.com/5.1/en-US/gameplay-framework-quick-reference-in-unreal-engine/
https://docs.unrealengine.com/5.1/en-US/game-mode-and-game-state-in-unreal-engine/

-Game Mode: GMode_22
Accepts new connections from controllers, spawns a pawn, orders controller to possess pawn
Spawns the "router" which helps certain blueprints find and talk to each other

-Player Controller: PC_VR_22
Sets variables in Pawn and PC so they can reference each other
Checks whether the VR setup has already been completed this instance
Forces VR Setup if necessary, saves setup information locally and in the Game Instance, then prompts the Pawn to continue further initialization stuff

-Game Instance: GInstance_22
Holds the variable for Pawn VR setup once completed so it doesn't need to be repeated.



=Pawn
Pawn_VR_22

SCENES
There are many scene components within the Pawn blueprint.  These are the most important.

-RootBox
This represents the origin of your VR playspace, in realspace.  If it moves, everything about your pawn moves with it, including tracked components.
For 360 locomotion, this is the component that moves.  It also attaches itself to the ground so that if the ground moves, you follow it.
Transforms applying to the "actor" are the same as applying them to the RootBox.

-VROO Box
The VR Origin Offset is a child of the RootBox and a parent to the tracked components.
While the RootBox is the VR Origin in real space, the VROO acts as the Pawn's origin in virtualspace.  Separating these two ideas allows us to "recenter" the pawn without invoking XR API functions that may work differently on different systems.
This is the scene whose rotation matters for the pawn.  It is offset for minor adjustments or recentering.

-Camera
This is a tracked input; it automatically updates its transform (relative to the VROO) to match HMD movement in realspace.
It is not moved manually; the RootBox or VROO, or other scenes that help track the player's location, are moved around it.

-Motion Controllers
MC_Left and MC_Right
These are tracked components.  At this time, the template uses a separate mesh for left and right hands (the alternative is to invert the scale on one mesh).
There are two sets of hands for each motion controller, a "ghost" hand and a "solid" hand.  The ghost hand always shows the player where his hand is relative to the camera 1:1 in realspace and virtualspace, and is only visible to the owning player.  The solid hand shows where the pawn's hand is as far as the game is concerned.  The solid and the ghost hands are typically in exactly the same spot, but there may be some difference due to collision or item interaction.  The ghost hand does not collide or interact.

There are many other scene components.  They mostly serve as visualization aids, and common named reference points.  The help position the body, determine player height, etc.  I will mention them as they come up.

EVENTS

-StartUp
The BeginPlay lives on a separate event graph with the pawn blueprint called "StartUp."  It waits until the pawn is properly possesed by the player controller before running.
Mostly it initializes variables, takes note of initial tracking data, and moves some scene components to their starting positions.
When complete it toggles "Common Setup Complete" (distinct from VR Setup), though it may wait to make sure tracking is working properly.

-VRMenu
This event graph holds all of the VR-specific setup functions, like figuring out the relative positions of the player and the floor (both in realspace and virtualspace).  This is probably the most complicated part of the blueprint, but probably the most solid; I don't expect many changes to be made here.
There is also currently a VR menu living in this event graph that lets the player control the setup process and override certain settings during gameplay.  I hate the way this is currently implemented and will be moving it to a separate blueprint that allows for better ergonomics for the player and more flexibility for the developer.  Do not get attached to the menu or add too much that depends on it.

-EventGraph
This is where "Tick" lives, but it won't go until the common setup is complete.
--"Pre"
The first thing it does on tick is update the real and virtual floor location for later use.
Control Inputs are copied into variables and processed for later use.  Some of them are used to execute actions immediately but they shouldn't be.
--"During"
Everything to do with Hands then fires, it has its own Event Graph page.  Hands will include item interaction, collision, climbing locomotion, and sometimes physics.
The appropriate "movement sequence" fires, depending on whether the pawn is walking or floating.  These each have their own page.  This mostly includes locomotion stuff, collision, physics, and whether to switch to a different movement regime.  The pawn is moved directly.
--"Post"
Body parts are moved to the appropriate spot.  The effects of any collisions are applied.
Various velocities are recorded for future use.

Everything else is details.  There are a lot of details but you should have an idea of where to find them now.

Incomplete list of potential Pawn behavior:
Reset Orientation and Position without XR API call
Float in zero g, moving with thrusters or by pushing off from walls with hands, or from physics collision, bouncing off walls
Switching from floating to walking when coming into contact with a floor of approximately the same normal as the pawn
Climbing walls, vaulting over walls
360 Locomotion over various terrain, up normal can be snapped in various ways or follow the ground
Edge detection to prevent pawns from walking off cliffs both in 360 and roomscale
Collision detection that prevents pawns from walking through things but doesn't interfere too much with locomotion
The ability to lean over a counter or something without collision detection bouncing you back right away
Pawns bumping their heads on the ceiling will be forced to crouch until they can fit
Jumping by realspace jumping (or a quick bounce on tippy toes), with the jump velocity modified by hand swinging and prior pawn speed and floor velocity
I think the direction the pawn is looking relative to the movement direction also does something
A walking pawn will move along with the floor if it is also moving/rotating
Smooth hand animations that depending on what controller buttons are pressed
Hands can collide, push, climb walls, grab items, interact with held items, push buttons
Pawn can be "seated" with constraints


=Buttons
There are several blueprints and components associated with buttons, including the pawn.  The reason it's so complicated is that I want to be able to dynamically reassign the target functions from a huge library and changing actors, and I want to cut down the amount of work it takes to set that up.

BUTTON COMPONENT
This is the actual button.  It moves when it's pressed, shows its state, and can trigger a function somewhere.
There are many things set up by default, and there are many options available, but only one thing you actually must set.  The "Target Input" struct will ask for the function name, the target name, and the target method.  The function name is a nickname you've assigned to the function (see below).  The target name is the actor where that function is located. This target actor can be located using a variety of methods.
"OwnActor" is the simplest; the target is whatever actor the button is attached to, you do not need to enter a name with this method.
"UsingPawn" (not currently implemented) will target whatever pawn is pressing the button.
"DirectName" (not currently implemented) will use the actual object name assigned by the engine, the catch is you have to know what that is.
"LookupByFunctionName" (not currently implemented) is useful for when there is only a single unique function by that name in the game and you don't care which actor it happens to reside in.  This will use the router (see below) to facilitate lookup.
"UniqueTargetNick" is a nickname you've assigned to the actor ahead of time, the router (see below) will find it for you.
"ManualSet" lets you skip all that stuff and input the targeting data directly, without bothering with a router.

ROUTING SINGLETON
This is an obnoxious hack and I'm sorry.  This actor is spawned by the game mode on start.  It helps connect buttons with their target functions in actors.  Buttons and actors with target functions call in to the router when the game starts (or when something relevant changes) to let it know they exist and if they are looking for someone in particular (see below).  The router assigns an integer ID to remote object (e.g. button), target object, and remotely callable function.  It keeps track of function status (i.e. whether they are activated and may change status) and some other stuff and passes that information from the target to the remote.  Optionally, it can act as a filter when the remote attempts to activate an unavailable function so the target actor isn't bothered by impossible requests.

REMOTE TARGET HELPER
This is a cute little scene component you can add to any actor you want to be able to trigger functions on by pressing a button or something.  It is not strictly necessary, but it helps by acting as a function library and keeping variable names predictable.  It helps to communicate with the router so you don't need to copy/paste the same logic everywhere you want a remote function.  This could also be accomplished by inheritance or something but I didn't want to do it that way.

It requires input from its parent actor on start.  The function Pre-Setup From Actor needs to be called with the relevant data filled in. This is where the target nickname is established, and the list of potential functions along with their default statuses are initialized.
The helper also contains a function called Quick Status Setter.  If your remote functions have simple status schemas, this function will set or flip them, record the new status in a local variable, and inform the router of the update (which will then inform any subscribed terminals so they can update their displays or whatever).

Any Remote Target Actor
In addition to the helper, an actor with a remote target needs to have the Interface_Remote Functions interface.  This comes with a one-way message received event called Event_Router to Function Standard.  When a button (or something) tells the router it wants to trigger a remote function, this is the message the router sends to the target actor.  You will want to add this event to the actor and use a SwitchOnName node to route the message to the actual target function.  At that point, the target actor will trigger the target function and make any changes to the function status.

Note: the way the "standard" function trigger is conceived, receipt of the message by the target actor is permission to fire.  By the time it receives the command the button has already checked its local copy of the status to ensure it's allowed, and the router (which should be located on the server) checks its copy before forwarding the message to the target actor.  You may want to create a more complicated message chain where the permission to trigger is checked again at the target actor, but then you will also probably want to figure out how to get the failure message back to the terminal/button.  You may also want to bypass the router entirely for the sake of simplicity, this is discussed below.

The Event_Quick Activate is also within the remote functions interface.  It is used in conjunction with manual button setup and bypasses the need for a router completely.  The status of the function is not tracked and it is assumed the button/terminal always has permission to fire the event.


=Items
Things that can be picked up by the pawn and manipulated are simply known as "items."  They are fairly flexible, and composed of several helper scene components.  I expect the most change here as the template becomes multiplayer capable.  I also have a vague intent to simplify it eventually.

ITEM COORDINATOR
Every item actor needs a coordinator.  It helps individual subcomponents communicate with each other and facilitates state changes.  It finds, activates, and tracks each subcomponent.  It also handles physics changes and such when dropped/thrown.

ITEM SUBCOMPONENT [logic] (SCL)
Each subcomponent logic scene should be a child of the component it controls, controlled components may have more than one SCL.  The root component (the main body of the item) should probably be the "primary," and this needs to be checked for each SCL attached to the primary.  This helps to alert the coordinator when a major change in state has occurred.  If there is more than one SCL per component, they each need a uniqe name.  SCL functionality is controlled by a nested struct called Functions by State.

SCL States: 
Resting - item is left alone
Null - not used
Attached - the item is attached to another item actor (think magazine in a rifle)
Held - the item is held by a pawn
Manipulated - a piece of the item is being moved by a pawn, but cannot necessarily be picked up and moved around (think moving handles or a foregrip that stay connected to their parent while moved)

Only states you intend to use should be added to the struct.  Within each state, there are a variety of relations, which act as permission classes.

SCL Relations:
Owner - the pawn currently using the primary component (but NOT the component in question), OR a parent SCL the component is attached to
User - the actual hand currently using this component
Offhand - the other hand of the pawn currently using this component
Public - not otherwise currently related

So we have told the component "If you are in STATE, and RELATION attempts to use this component.." The next level of the struct is the actual function we want the component to be capable of when it is in that state and approached by something trying to use it.  There should only be one function per state:relation pair.  They generally result in a state change, and may also trigger a new SCL to take over.

SCL Functions:
Pickup - the item is held by the requesting hand
Drop - the item is dropped by the holding hand because the grip button has been released
Drop Sticky - the item has been dropped by the holding hand because the grip button has been released and then pressed again (sticky items do not require the player to hold the grip button to continue holding them)
Manipulate - the component now may be moved within constraints while staying attached to its parent by the requesting hand
Release - the component is no longer being manipulated because the player released the grip button
Release Sticky - "" and then pressed the grip button again
Activate Non Diagetic - For triggering an event without actually touching the item
Trigger Actions - Not yet implemented, but would call a non-pickup/manipulate type function like pulling a trigger
Other Actions - 
Dry Grip - Not used during setup, but is sent from the hand to facilitate sticky things somehow

Additionally, there are optional constraints for each function:
Block - do not allow this function to activate
Prox Only - the requesting hand must be within close proximity to allow the function
Other Checks - not yet implemented, but is a freeform checklist you can create for complicated permissions
New Logic - optional, but it's the name of the SCL that should take over if this function is allowed

Note: you may want to switch to a new logic when a function/state change requires additional changes (e.g. a different highlighter box), more below

Note: Practically any time an SCL does something, it fires an event on the parent actor (if it has the Item Interaction Interface) to let it know.  You can use this to have the item do more complicated things not already contained within the SCLs.

Other SCL Configuration:
Primary - already discussed.
Attachment Info - should be fairly self explanatory, see the examples.
Is foregrip/reargrip - on a rifle this causes each hand to control a different aspect of the transform

There are three more helper scene components that can optionally be attached (they should find each other automatically) to SCLs.

HAND PLACEMENT AID
This allows you to easily tell the SCL where the holding/manipulating hand (one each for left and right) should be relative to the component by placing them in the viewport.

MANIPULATION CONSTRAINT
A box you can place and scale within the viewport to easily tell a manipulatable component where it may move relative to its parent.

ATTACHMENT REFERENCE
Any scene component attached to an SCL with attachment enabled, created by adding "attm" or "attf" tags (for male and female respectively) to the scene component.  Allows you to use the viewport to finetune exactly where items attach to one another.

