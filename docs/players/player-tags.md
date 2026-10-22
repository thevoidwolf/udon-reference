# Player Tags

A per-player string-to-string store local to each player. Useful for marking players in custom states (`"team": "red"`, `"in_minigame": "true"`) without using the networking pipeline.

Source: the "Player Tag System" section of <https://creators.vrchat.com/worlds/udon/players/getting-players#player-tag-system> (the old `/players/player-tags/` URL now 404s). The docs call it "a quick-and-dirty way of assigning strings to players". **[Docs]**

## API

```csharp
player.SetPlayerTag(string tagName, string tagValue);
string val = player.GetPlayerTag(string tagName);   // unset value: not documented ("" or null) [Unverified]
player.ClearPlayerTags();                            // remove all tags you've set on this player
```

**`GetPlayersWithTag` does not work.** The docs mark it "Not currently working" — it returns a `List`, which Udon can't use. Earlier revisions of this page showed it returning `VRCPlayerApi[]`; that was wrong. Iterate players and compare `GetPlayerTag` instead (see pattern below). **[Docs]**

## Semantics

- Tags are **local-only**. Setting a tag on a player only affects what the *local* client thinks. They're not networked. **[Community-known]** (the docs don't state sync behaviour)
- Calling `SetPlayerTag` on a remote player from your client only affects your view.
- Setting an empty-string value (`""`) is the usual way to "clear" a single tag; there is no per-tag remove. **[Community-known]**

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

VRCPlayerApi[] _players = new VRCPlayerApi[16];

void Update() {
    int n = VRCPlayerApi.GetPlayerCount();
    if (_players.Length < n) _players = new VRCPlayerApi[n * 2];
    VRCPlayerApi.GetPlayers(_players);          // non-allocating overload
    for (int i = 0; i < n; i++) {
        var p = _players[i];
        if (!Utilities.IsValid(p) || p.GetPlayerTag("minigame") != "active") continue;
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
