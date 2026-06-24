# Animation Events

Unity's Animation Window lets you add events at specific timestamps during a clip. VRChat allowlists a specific set of method names callable through this system.

Source: <https://creators.vrchat.com/worlds/udon/animation-events/>. **[Docs]**

## How to use

1. Open the Animation Window (Window → Animation → Animation).
2. Select the GameObject playing the clip.
3. Right-click the timeline at the desired time → Add Animation Event.
4. In the Inspector for the event, choose:
   - **Function**: one of the allowlisted method names below.
   - **Object** / **String** / **Float** / **Int**: parameter according to the chosen method.

## Allowlisted methods

### `UdonBehaviour`

| Method | Param | Use |
|---|---|---|
| `SendCustomEvent(string eventName)` | string | The canonical way to call into your U# code from animation. |
| `RunProgram(string entryPoint)` | string | Lower-level. Most worlds use `SendCustomEvent`. |

### `Animator`

| Method | Param |
|---|---|
| `Play(string stateName)` | string |
| `PlayInFixedTime(string stateName)` | string |
| `SetBool(string name, bool value)` | name + value via separate event params (limited; community-known) |
| `SetFloat(string name, float value)` | |
| `SetInteger(string name, int value)` | |
| `SetTrigger(string name)` | string |
| `ResetTrigger(string name)` | string |

### `Animation`

| Method | Param |
|---|---|
| `Rebind()` | none |

### `AudioSource`

Animation events targeting an AudioSource can call methods listed in the [UI Whitelist](ui-whitelist.md) (`Play`, `Pause`, `Stop`, etc.). The animation-event system shares the UI event allowlist for component-method calls.

### `GameObject`

| Method | Param |
|---|---|
| `SetActive(bool)` | bool |

## Anything not on this list won't fire

If you choose a method outside the allowlist, the animation event will simply not call anything. The Unity Animation Window won't warn you; you'll just see nothing happen at runtime.

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

- **Method signatures must be no-arg.** Animation events configured to call your U# method should only use `SendCustomEvent("methodName")`. Don't try to pass parameters through animation events to U# methods; the parameter forms aren't supported on UdonBehaviour.
- **Animation event timing precision** depends on Unity's animation clip sampling. Don't expect frame-accurate timing — events fire at the next animation update tick at or after their timestamp.
- **Imported clips lose events on re-import** unless you've added events to the clip asset itself (vs. via Animator state-machine state events). Be aware of which mechanism you're using.

## See also

- [UI Events](ui-events.md)
- [UI Whitelist](ui-whitelist.md)
