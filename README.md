# Unity Third Person Controller

A third-person player controller for Unity with walking, jumping, double jump, rolling and gliding. Movement is driven by a small finite state machine so each behavior is isolated and easy to swap or extend.

Built on Unity's Input System and camera-relative movement, so the controls always feel right regardless of where the camera is looking.

## Features

- Camera-relative movement using Unity's Input System
- Finite state machine for movement (Walk, Ball Run, Glide), each state owns its own speed, acceleration and gravity settings
- Jump and double jump with ground detection
- Roll state with its own movement profile
- Glide state with reduced gravity for slow descents
- UnityEvents on state enter and exit for hooking up animations, sounds or VFX without touching the controller code

## Requirements

- Unity 2022.3 LTS or newer
- Input System package (com.unity.inputsystem)
- A third-person camera in the scene (Cinemachine or any camera you drive yourself)

## Getting Started

1. Clone the repo and open the project in Unity
2. Open the demo scene to see the controller in action
3. To use it in your own project, copy the `Scripts/Player` folder and the input actions asset into your project

The player prefab needs the following components on the same GameObject:

- `PlayerInputManager` reads the Input System actions and exposes them to the rest of the controller
- `PlayerMoveFSM` runs the movement state machine
- `PlayerJump` handles jump and double jump
- `PlayerRotation` rotates the character based on camera and input

Assign your main camera on `PlayerMoveFSM` so movement is transformed into world space correctly.

## Adding a new movement state

States inherit from `PlState`. Create a subclass, implement the movement behavior, and add it to the FSM.

```csharp
public class PlSprintState : PlState
{
    public float SprintSpeed = 12f;
    public float Acceleration = 40f;

    public override void OnEnter() { /* setup */ }
    public override void OnTick()
    {
        // read input, apply movement using SprintSpeed and Acceleration
    }
    public override void OnExit()  { /* cleanup */ }
}
```

Wire the transition in `PlayerMoveFSM` (for example, enter sprint while walking and the sprint input is held, exit when the input is released).

## How it works

`PlayerInputManager` reads Input System actions once per frame and stores the raw values (movement vector, jump pressed, glide held, etc). The other components read from it instead of talking to the Input System directly, which keeps things testable and easy to rebind.

`PlayerMoveFSM` picks the active state based on input and context (grounded, in air, glide button held). Each `PlState` subclass defines its own movement, so switching between walking and gliding is just switching the active state. States expose UnityEvents for enter and exit so you can hook animation triggers or sound effects from the inspector.

`PlayerJump` runs alongside the FSM. It listens for jump input, checks if the player is grounded or has a double jump available, and applies the impulse.

Movement input is always transformed relative to the camera, so pressing forward moves the character away from the camera regardless of where either is facing.

## Project structure

```
Assets/
  Scripts/
    Player/
      Input/         PlayerInputManager and input actions
      Movement/      PlayerMoveFSM, PlState, PlWalkState, PlBallRunState, PlGlideState
      Jump/          PlayerJump
      Rotation/      PlayerRotation
  Scenes/
    Demo.unity       Demo scene with the character and camera set up
```

## Notes

- The FSM is intentionally simple. If you need something more complex (hierarchical states, visual editor, etc.), a dedicated FSM tool would fit better
- Gravity is handled per state rather than globally, which means gliding and falling behave differently even though both are "in the air"
- The controller assumes a `CharacterController` or `Rigidbody` on the player. Check the demo prefab to see which one is used

## License

MIT. See `LICENSE` for details.

## Contact

Marc Sans Cormenzana &middot; [LinkedIn](https://www.linkedin.com/in/marc-sans-cormzn) &middot; marcsans1212@gmail.com
