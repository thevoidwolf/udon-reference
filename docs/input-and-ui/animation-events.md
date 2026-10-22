# Animation Events

Unity's Animation Window lets you add events at specific timestamps during a clip. VRChat allowlists a specific set of method names callable through this system.

Source: <https://creators.vrchat.com/worlds/udon/animation-events/>. **[Docs]** Unity reference: [Use Animation Events](https://docs.unity3d.com/2022.3/Documentation/Manual/script-AnimationWindowEvent.html).

## How to use

1. Open the Animation Window (Window → Animation → Animation).
2. Select the GameObject playing the clip.
3. Right-click the timeline at the desired time → Add Animation Event.
4. In the Inspector for the event, choose:
   - **Function**: one of the allowlisted method names below.
   - **Object** / **String** / **Float** / **Int**: parameter according to the chosen method.

## Allowlisted methods

The official allowlist is a flat list of **function names** (13 entries): **[Docs]**

`RunProgram`, `SendCustomEvent`, `Play`, `Pause`, `Stop`, `PlayInFixedTime`, `Rebind`, `SetBool`, `SetFloat`, `SetInteger`, `SetTrigger`, `ResetTrigger`, `SetActive`

Unity dispatches an animation event by name to the components on the animated GameObject, so in practice these map to:

| Name(s) | Likely target | Param |
|---|---|---|
| `SendCustomEvent`, `RunProgram` | `UdonBehaviour` | string (event name) — the canonical way into U# code. |
| `Play`, `PlayInFixedTime`, `Rebind`, `SetBool`, `SetFloat`, `SetInteger`, `SetTrigger`, `ResetTrigger` | `Animator` | string (state / parameter name) |
| `Play`, `Pause`, `Stop` | `AudioSource`, `ParticleSystem` | none |
| `SetActive` | `GameObject` | — |

The name → component mapping is inferred from Unity's animation-event behaviour, not stated by VRChat. **[Unverified]** Unity animation events pass **at most one** parameter (float, int, string, or object), so two-argument methods such as `SetBool(string, bool)` / `SetFloat(string, float)` can't receive both a name and a value this way — use `SendCustomEvent` into U# and set the Animator parameter from code. **[Unverified]** (standard Unity limitation)

Animation events have a **separate, much shorter** allowlist than [UI events](ui-whitelist.md): e.g. `PlayOneShot`, `PlayDelayed`, `UnPause` and property setters are UI-event-only.

## Anything not on this list won't fire

If you choose a method outside the allowlist, it "will not be called when your world runs in VRChat" **[Docs]** — the event simply does nothing. The Unity Animation Window won't warn you; you'll just see nothing happen at runtime.

## Pattern: animation-driven sound and logic

```csharp
public AudioSource hitSfx;

// Called from an animation event at the "impact" frame:
public void OnImpact() {
    hitSfx.PlayOneShot(hitSfx.clip);
    DoDamage();
}
```

Set up the animation event with Function = `SendCustomEvent`, String = `OnImpact`.

## Caveats

- **Method signatures must be no-arg.** Animation events configured to call your U# method should only use `SendCustomEvent` with String = `methodName`. Your U# method name itself is not on the allowlist, so pointing the event's Function directly at it does nothing in-client.
- **Animation event timing precision** depends on Unity's animation clip sampling. Don't expect frame-accurate timing — events fire at the next animation update tick at or after their timestamp.
- **Imported clips lose events on re-import** unless you've added events to the clip asset itself (vs. via Animator state-machine state events). Be aware of which mechanism you're using.

## See also

- [UI Events](ui-events.md)
- [UI Whitelist](ui-whitelist.md)
