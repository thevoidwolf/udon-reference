# VRCShader

Static class `VRC.SDKBase.VRCShader`. Pushes values into **global** shader uniforms — values that all shaders rendering in the instance can read. Source: <https://creators.vrchat.com/worlds/udon/vrc-graphics/>. **[Docs]**

## Methods

### `PropertyToID`

```csharp
int PropertyToID(string propertyName);
```

Returns a stable int ID for a shader property name. **Call this once at `Start()` and cache the ID** — name → ID is a hash computation, the ID setters are fast.

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
void SetGlobalInteger(int id, int value);      // stored as float — Unity bug per VRC docs
void SetGlobalColor(int id, Color value);
void SetGlobalVector(int id, Vector4 value);
void SetGlobalVectorArray(int id, Vector4[] values);
void SetGlobalMatrix(int id, Matrix4x4 value);
void SetGlobalMatrixArray(int id, Matrix4x4[] values);
void SetGlobalTexture(int id, Texture value);
```

**[Docs]**

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

## `_VRChat`-prefixed globals are reserved

VRChat reserves `_VRChat*`-prefixed shader globals for its own built-ins (`_VRChatMirrorMode`, `_VRChatCameraMode`, `_VRChatMirrorCameraPos`, `_VRChatTime`). Do **not** use that prefix for your own values. **[Docs]** <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrchat-shader-globals/>

## AudioLink — the canonical `_AudioTexture` consumer

**AudioLink** by llealloo and collaborators is the canonical community use of the `_AudioTexture` exception. It analyzes in-world audio using Unity's `GetOutputData`, runs DSP on the GPU, and publishes results into a 128×64 RGBA `CustomRenderTexture` exposed globally as `_AudioTexture`. **[Community]** <https://github.com/llealloo/audiolink>

Texture layout:
- 4-band amplitude (Bass / Low-Mids / High-Mids / Treble) at `ALPASS_AUDIOLINK (0,0)`.
- 128×2 DFT spanning 10 octaves at `(0,4)`.
- 128×16 waveform region at `(0,6)`.
- ColorChord lighting rows.
- 127 pixels of history cascade rightward each frame for smooth temporal interpolation.

Avatar shaders and world shaders can read this texture identically — that's the whole point of `_AudioTexture` being globally allowlisted.

**Quest caveat:** AudioLink requires synchronous GPU readback on Quest (no async readback there), which the docs explicitly recommend against. **[Community]** <https://github.com/llealloo/audiolink>

## See also

- [VRCGraphics](vrc-graphics.md)
- [Materials](materials.md) — `Material.SetFloat/SetColor` for per-material uniforms (not global).
- [References → Community Resources](../references/community-resources.md) — AudioLink, LTCGI, shader projects.
