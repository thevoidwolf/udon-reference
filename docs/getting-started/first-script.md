# First Script

A minimal complete UdonSharp script and a tour through what each piece does.

## Create the script

In Unity:

1. Right-click in the Project view → **Create → U# script**.
2. Name it `Greeter.cs`. Unity creates two assets:
   - `Greeter.cs` — your source file.
   - `Greeter.asset` — the paired Udon Program Asset.
3. Open `Greeter.cs` in your IDE.

## Replace the contents

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;
using VRC.Udon;

[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class Greeter : UdonSharpBehaviour
{
    [Tooltip("Shown when a player walks into the trigger.")]
    public string greeting = "Hello, ";

    public override void OnPlayerTriggerEnter(VRCPlayerApi player)
    {
        if (player == null || !player.IsValid()) return;
        Debug.Log(greeting + player.displayName);
    }
}
```

## Attach it

1. In the Hierarchy, create a `GameObject` with a `Collider` set to **Is Trigger**.
2. Drag `Greeter.cs` onto that GameObject. Unity adds an `UdonBehaviour` and links it to the program asset; the C# class shows up as the "Udon" component in the inspector.
3. Build & Test the world (`VRChat SDK → Show Control Panel → Builder → Build & Test`). Walk through the trigger; the log line appears in the VRChat client logs.

## What every line does

```csharp
using UdonSharp;            // For UdonSharpBehaviour, attributes
using UnityEngine;          // Standard Unity API (Tooltip, Debug, etc.)
using VRC.SDKBase;          // VRCPlayerApi
using VRC.Udon;             // UdonBehaviour, used in advanced cross-behaviour code
```

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
```
Declares this behaviour does **not** participate in networking. No `[UdonSynced]` fields allowed, no `SendCustomNetworkEvent`. See [BehaviourSyncMode](../attributes/behaviour-sync-mode.md).

```csharp
public class Greeter : UdonSharpBehaviour
```
All U# scripts derive from `UdonSharp.UdonSharpBehaviour`. Never from `MonoBehaviour` directly — the compiler will reject that.

```csharp
[Tooltip("...")]
public string greeting = "Hello, ";
```
A regular Unity inspector field. Public fields on a `UdonSharpBehaviour` are serialized just like on a `MonoBehaviour`. You can drag values onto them in the inspector. Field initializers are evaluated **at compile time**, not at runtime — so `greeting = "Hello, "` becomes baked into the program asset.

```csharp
public override void OnPlayerTriggerEnter(VRCPlayerApi player)
```
Overrides a virtual lifecycle event declared on `UdonSharpBehaviour`. The VRChat runtime calls it when a player's capsule collider enters a trigger on this GameObject. The list of available overrides is in [UdonSharpBehaviour lifecycle events](../udonsharp-behaviour/lifecycle-events.md).

```csharp
if (player == null || !player.IsValid()) return;
```
Defensive check. `VRCPlayerApi` references can become invalid if the player leaves between when the reference was captured and when you use it. `IsValid()` always returns the current truth.

```csharp
Debug.Log(greeting + player.displayName);
```
`Debug.Log` is supported. In-game log output appears in the VRChat client's `output_log.txt` (Windows) or `Player.log` (other platforms), not in the Unity Editor console once you're in-VRChat.

## Run it in Play mode

UdonSharp executes inside Unity's Play mode — you do **not** need to upload to test. Press Play, walk a character with a Collider through the trigger, and watch the Unity console.

If you want the in-VRChat behaviour to match Play mode (player API, networking), use **Build & Test** instead.

## What comes next

That's the smallest meaningful U# script. From here:

- Networked toggle → [Networking](../networking/README.md).
- Make the script respond to interaction (E-key / trigger) → [Interaction](../udonsharp-behaviour/api-methods.md#interaction).
- Read player position, voice settings, etc. → [Players](../players/README.md).
- Spawn objects → [GameObject Lifecycle](../gameobject-lifecycle/README.md).
