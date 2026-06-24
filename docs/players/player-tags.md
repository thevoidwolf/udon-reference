# Player Tags

A per-player string-to-string store local to each player. Useful for marking players in custom states (`"team": "red"`, `"in_minigame": "true"`) without using the networking pipeline.

Source: <https://creators.vrchat.com/worlds/udon/players/player-tags/>. **[Docs]**

## API

```csharp
player.SetPlayerTag(string tagName, string tagValue);
string val = player.GetPlayerTag(string tagName);   // "" if not set
player.ClearPlayerTags();                            // wipe all tags for this player
```

Plus the static helper:

```csharp
VRCPlayerApi[] withTag = VRCPlayerApi.GetPlayersWithTag(string tagName, string tagValue);
```

## Semantics

- Tags are **local-only**. Setting a tag on a player only affects what the *local* client thinks. They're not networked.
- Calling `SetPlayerTag` on a remote player from your client only affects your view.
- Empty-string value (`""`) effectively removes the tag.

That makes them useful for one-sided "I'm tracking this" state, not for shared game state. For shared state, use `[UdonSynced]` fields or a per-player object.

## Patterns

### Local tag for tracking who's playing your minigame

```csharp
public void StartGameFor(VRCPlayerApi p) {
    p.SetPlayerTag("minigame", "active");
}

public void EndGameFor(VRCPlayerApi p) {
    p.SetPlayerTag("minigame", "");   // clear
}

void Update() {
    var players = VRCPlayerApi.GetPlayersWithTag("minigame", "active");
    foreach (var p in players) {
        // do per-player game-loop work for the local view
    }
}
```

### Filter raycast targets

Use tags as a "category" for in-game queries without taking on the cost of `GameObject.tag` (which is per-GameObject and must be in your project's tag list).

## Caveats

- Tags persist for the player only while they are in your world's instance — they reset on world unload.
- No documented limit on tag count or string length. **[Unverified]**
- Tag values are strings; encode numeric/Boolean state yourself.

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [UdonSharpBehaviour → player events](../udonsharp-behaviour/player-events.md) — `OnPlayerJoined` is a good place to initialize tags.
