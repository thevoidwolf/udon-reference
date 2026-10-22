# First Script

A minimal complete UdonSharp script and a tour through what each piece does.

## Create the script

In Unity:

1. Right-click in the Project view → **Create → U# Script**.
2. In the save dialog, name it `Greeter.cs`. UdonSharp creates two assets side by side: **[Source]** `Editor/Editors/UdonSharpBehaviourEditor.cs`
   - `Greeter.cs` — your source file (generated from the default or your *Script template override*).
   - `Greeter.asset` — the paired Udon Program Asset.
3. Open `Greeter.cs` in your IDE.

The class name must match the file name (`Greeter` in `Greeter.cs`) or the compiler errors with "UdonSharpBehaviour classes must have the same name as their containing .cs file". **[Source]** Scripts can live under `Assets/` or, from **SDK 3.10.5+**, inside a package under `Packages/`. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5/>

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
2. Drag `Greeter.cs` onto that GameObject (or **Add Component → Greeter**). The `Greeter` component is the U# *proxy*; UdonSharp creates and links the hidden backing `UdonBehaviour` that points at `Greeter.asset`. See [Proxy ↔ Backing](../editor-api/proxy-backing.md).
3. Build & Test the world: `VRChat SDK → Show Control Panel → Builder` tab → **Build & Test**. Set *Number of Clients* to 2+ to test networking with several local clients; set it to 0 to turn the button into *Build & Reload* for already-open clients. **[Docs]** <https://creators.vrchat.com/worlds/udon/using-build-test> Walk through the trigger; the log line appears in the VRChat client log.

Build & Test notes:
- **Any account rank** can Build & Test (the trust-rank requirement was removed; Visitors allowed) — **SDK 3.9.0+**. Uploading still requires a VRChat account that meets the upload requirements. **[Docs]** <https://creators.vrchat.com/releases/release-3-9-0/>
- **Mobile/standalone**: Build & Test on Quest (**SDK 3.7.5+**) and Android devices over ADB, and on iOS via the VRChat Test Protocol (**SDK 3.8.1+**). **[Docs]** <https://creators.vrchat.com/platforms/android/build-test-mobile/>, <https://creators.vrchat.com/platforms/iOS/build-test-mobile>
- UdonSharp recompiles all scripts when you switch build platform (**SDK 3.9.0+**). **[Docs]**

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
Declares this behaviour does **not** participate in networking. No `[UdonSynced]` fields allowed, and `SendCustomNetworkEvent` will not work on it. **[Docs]** <https://udonsharp.docs.vrchat.com/udonsharp> See [BehaviourSyncMode](../attributes/behaviour-sync-mode.md).

```csharp
public class Greeter : UdonSharpBehaviour
```
All U# scripts derive from `UdonSharp.UdonSharpBehaviour`. Never from `MonoBehaviour` directly — the compiler will reject that.

```csharp
[Tooltip("...")]
public string greeting = "Hello, ";
```
A regular Unity inspector field. Public fields on a `UdonSharpBehaviour` are serialized just like on a `MonoBehaviour`. You can drag values onto them in the inspector. Field initializers are evaluated **at compile time**, not at runtime — so `greeting = "Hello, "` becomes baked into the program asset; put initialization that depends on other scene objects in `Start`. **[Docs]** <https://udonsharp.docs.vrchat.com/>

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
`Debug.Log` is supported. In-game log output goes to the VRChat client's log — on PC `C:\Users\<you>\AppData\LocalLow\VRChat\VRChat\output_log_<timestamp>.txt`, one file per launch. **[Docs]** <https://creators.vrchat.com/worlds/udon/debugging-udon-projects/> It does not go to the Unity console by default, but:

- UdonSharp's log watcher (*Listen for client exceptions*, on by default in **Project Settings → Udon Sharp**) reads the client log during Build & Test and forwards Udon exceptions — matched to script and line — to the editor console. **[Docs]** The *Output log watch mode* setting can also forward ordinary messages, optionally filtered by prefix. **[Source]** `Editor/Editors/UdonSharpSettings.cs` The watcher was updated for VRChat's new log format in **SDK 3.7.6**. **[Docs]**
- **`VRChat SDK → Udon Sharp → Parse Logs from File`** (**SDK 3.10.2+**) runs the same parsing over a saved log file — handy for logs from another machine or a Quest. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-2/>

## Run it in Play mode

UdonSharp executes inside Unity's Play mode — you do **not** need to upload to test. **ClientSim**, which ships inside the Worlds SDK, spawns a simulated local player when you press Play: walk it through the trigger and watch the Unity console. **[Docs]** <https://creators.vrchat.com/worlds/clientsim/>

ClientSim only simulates the *local* player — no remote players, and networking/serialization results are approximations. **[Docs]** For behaviour that must match VRChat (player API details, networking), use **Build & Test** instead. See [ClientSim known gaps](../editor-api/README.md#clientsim-known-gaps).

## What comes next

That's the smallest meaningful U# script. From here:

- Networked toggle → [Networking](../networking/README.md).
- Make the script respond to interaction (E-key / trigger) → [Interaction events](../udonsharp-behaviour/interaction-events.md).
- Read player position, voice settings, etc. → [Players](../players/README.md).
- Spawn objects → [GameObject Lifecycle](../gameobject-lifecycle/README.md).
