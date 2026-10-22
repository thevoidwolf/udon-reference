# VRCShader

Static class `VRC.SDKBase.VRCShader`. Pushes values into **global** shader uniforms — values that all shaders rendering in the instance can read. Source: <https://creators.vrchat.com/worlds/udon/vrc-graphics/>. **[Docs]**

## Methods

### `PropertyToID`

```csharp
int PropertyToID(string propertyName);
```

Returns a stable int ID for a shader property name. **Call this once at `Start()` and cache the ID** — the ID never changes, even between different materials and shaders. **[Docs]**

```csharp
int _scoreId;
void Start() { _scoreId = VRCShader.PropertyToID("_UdonScore"); }
void Update() { VRCShader.SetGlobalFloat(_scoreId, currentScore); }
```

### Critical naming rule

**The property name must start with `_Udon` or be the literal string `_AudioTexture`.** Other names return an ID but `SetGlobal*` will silently fail to apply. **[Docs]**

This is VRChat's way of preventing world authors from clobbering common shader uniforms (`_MainTex`, `_Color`, etc.) that avatars and other systems depend on.

### Setters

All variants take the cached int ID returned from `PropertyToID`:

```csharp
void SetGlobalFloat(int id, float value);
void SetGlobalFloatArray(int id, float[] values);
void SetGlobalInteger(int id, int value);      // true integer since SDK 3.9.0 (see below)
void SetGlobalColor(int id, Color value);
void SetGlobalVector(int id, Vector4 value);
void SetGlobalVectorArray(int id, Vector4[] values);
void SetGlobalMatrix(int id, Matrix4x4 value);
void SetGlobalMatrixArray(int id, Matrix4x4[] values);
void SetGlobalTexture(int id, Texture value);
```

**[Docs]**

### `SetGlobalInteger` fix (SDK 3.9.0+)

Before SDK 3.9.0, `SetGlobalInteger` stored the value as a `float` (a Unity bug workaround), so large values and bit patterns were mangled. Since 3.9.0 it calls the correct integer variant of the backing `UnityEngine.Shader` function, **preserving bit patterns** — you can pack flags/IDs into an `int`/`uint` shader global. **[Docs]** <https://creators.vrchat.com/releases/release-3-9-0/>

Note: the VRCGraphics docs page still carries the old "still sets the value as `float` for now" sentence; the 3.9.0 release note supersedes it. If you must support worlds built on older SDKs, use `SetGlobalFloat` instead.

## Globals leak to avatars

Per VRC docs: *"values become available in all shaders in the world (including ones on avatars!)"*. Two consequences:

1. **Namespace your names** — `_UdonMyGame_Score` rather than `_UdonScore` so multiple worlds (or multiple worlds an avatar visits) don't collide.
2. **Don't ship secrets through globals** — anything you push here is readable by any shader in the world.

## Typical patterns

### Drive a wall material from gameplay state

```csharp
int _glowId;

void Start() {
    _glowId = VRCShader.PropertyToID("_UdonAlarmGlow");
}

public void TriggerAlarm() {
    VRCShader.SetGlobalFloat(_glowId, 1f);
    SendCustomEventDelayedSeconds(nameof(FadeAlarm), 3f);
}

public void FadeAlarm() {
    VRCShader.SetGlobalFloat(_glowId, 0f);
}
```

Shaders on any material in the world that read `_UdonAlarmGlow` will respond.

### Push an array of "active player positions"

```csharp
int _playerPositionsId;
Vector4[] _positions = new Vector4[80];   // VRChat instance cap

void Start() {
    _playerPositionsId = VRCShader.PropertyToID("_UdonPlayerPositions");
}

void LateUpdate() {
    int n = VRCPlayerApi.GetPlayerCount();
    var players = new VRCPlayerApi[n];
    VRCPlayerApi.GetPlayers(players);
    for (int i = 0; i < n; i++) {
        Vector3 p = players[i].GetPosition();
        _positions[i] = new Vector4(p.x, p.y, p.z, 1f);
    }
    VRCShader.SetGlobalVectorArray(_playerPositionsId, _positions);
}
```

Shaders can then use `_UdonPlayerPositions[i]` for proximity effects.

## VRChat shader globals

VRChat reserves the `_VRChat*` prefix as a protected namespace — new (including undocumented) variables may appear at any time. Do **not** use that prefix for your own values. **[Docs]** <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrchat-shader-globals/>

These are set by the client and readable in any world or avatar shader (no Udon needed): **[Docs]**

| Variable | Type | Contents |
|---|---|---|
| `_VRChatCameraMode` | `float` | `0` normal, `1` VR handheld camera, `2` desktop handheld camera, `3` screenshot. |
| `_VRChatCameraMask` | `uint` | `cullingMask` of the active camera; valid when `_VRChatCameraMode != 0`. |
| `_VRChatMirrorMode` | `float` | `0` not in mirror, `1` mirror viewed in VR, `2` mirror viewed on desktop. |
| `_VRChatFaceMirrorMode` | `float` | `1` while rendering the face mirror. |
| `_VRChatMirrorCameraPos` | `float3` | World-space mirror camera position (eye-centred); `(0,0,0)` outside mirrors. |
| `_VRChatScreenCameraPos` / `_VRChatPhotoCameraPos` | `float3` | Screen / handheld camera world position; `(0,0,0)` when inactive. |
| `_VRChatScreenCameraRot` / `_VRChatPhotoCameraRot` | `float4` | Same, as a quaternion; `(0,0,0,0)` when inactive. |

### Time globals (SDK 3.10.2+)

Four `uint` globals expose time to shaders (clocks, animation sync). **[Docs]** <https://creators.vrchat.com/releases/release-3-10-2/>

| Variable | Contents |
|---|---|
| `_VRChatTimeUTCUnixSeconds` | Lower 32 bits of UTC Unix time in seconds (treat as unsigned). |
| `_VRChatTimeNetworkMs` | Synced network time in ms — same value as `Networking.GetServerTimeInMilliseconds()`. Can wrap; use only for offsets/sync. |
| `_VRChatTimeEncoded1` | Bit-packed UTC hour/minute, shared second, local hour/minute. |
| `_VRChatTimeEncoded2` | Bit-packed millisecond, timezone-offset sign bit and offset (seconds). |

"Local" time is always the **observer's** time (respecting their preferred-timezone setting), not the avatar wearer's. Decode with the SDK's helper header:

```c
#include "Packages/com.vrchat.base/ShaderLibrary/VRCTime.cginc"

uint VRC_GetUTCUnixTimeInSeconds();
uint VRC_GetNetworkTimeInMilliseconds();
void VRC_GetUTCTime(out uint hours, out uint minutes, out uint seconds, out uint milliseconds);
void VRC_GetLocalTime(out uint hours, out uint minutes, out uint seconds, out uint milliseconds);
int  VRC_GetTimezoneOffsetSeconds();
```

**[Docs]** — see the official page for exact bit ranges.

## AudioLink — the canonical `_AudioTexture` consumer

**AudioLink** by llealloo and collaborators is the canonical community use of the `_AudioTexture` exception. It analyzes in-world audio using Unity's `GetOutputData`, runs DSP on the GPU, and publishes results into a 128×64 RGBA `CustomRenderTexture` exposed globally as `_AudioTexture`. **[Community]** <https://github.com/llealloo/audiolink>

Texture layout:
- 4-band amplitude (Bass / Low-Mids / High-Mids / Treble) at `ALPASS_AUDIOLINK (0,0)`.
- 128×2 DFT spanning 10 octaves at `(0,4)`.
- 128×16 waveform region at `(0,6)`.
- ColorChord lighting rows.
- 127 pixels of history cascade rightward each frame for smooth temporal interpolation.

Avatar shaders and world shaders can read this texture identically — that's the whole point of `_AudioTexture` being globally allowlisted.

**Toon Standard support (SDK 3.10.5+):** VRChat's built-in `Toon Standard` shader can react to AudioLink (Pulse / Scroll / Bar effects, including on mobile) — it only works in worlds that include the AudioLink community prefab, with a "Fallback Mode" for worlds without it. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5/>

**Quest caveat:** AudioLink requires synchronous GPU readback on Quest (no async readback there), which the docs explicitly recommend against. **[Community]** <https://github.com/llealloo/audiolink>

## See also

- [VRCGraphics](vrc-graphics.md)
- [Camera & Quality Settings](camera-and-quality-settings.md) — Udon-side counterpart of the camera globals.
- [Materials](materials.md) — `Material.SetFloat/SetColor` for per-material uniforms (not global).
- [References → Community Resources](../references/community-resources.md) — AudioLink, LTCGI, shader projects.
