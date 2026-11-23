# References

External resources and pointers into the UdonSharp source. Last full link/status check: **2026-10-22** (Worlds SDK 3.10.5). Official links and the source map re-checked **2026-11-23** (Worlds SDK 3.10.6, Unity 2022.3.22f1). Community project statuses were not re-checked in the November pass.

## Pages

| Page | Topic |
|---|---|
| [Source Map](source-map.md) | Where to read what in the UdonSharp source — bundled in the Worlds SDK (`com.vrchat.worlds`) since 3.4.0; the GitHub repo is frozen at U# 1.1.9. |
| [Official Docs](official-docs.md) | The VRChat-published pages this reference cites, plus release notes and the current Unity version page. |
| [Community Resources](community-resources.md) | Discord servers, GitHub repos, blog posts, decompiled bits. |
| [SDK 3.11 beta preview](sdk-3-11-beta.md) | What the Unity 6 SDK beta (3.11.0-beta.1, Nov 2026) changes for Udon: new APIs, Unity 6 renames, compiler fixes. To be folded into the topic pages when 3.11.0 ships. |

## What to consult when

| Question | Where |
|---|---|
| "What's the exact signature of method X?" | UdonSharp source in `Packages/com.vrchat.worlds/Integrations/UdonSharp/` ([Source Map](source-map.md)) or the SDK DLLs. |
| "When did API X ship / what changed in SDK Y?" | [Release notes](https://creators.vrchat.com/releases/) (linked from [Official Docs](official-docs.md)). For betas: <https://vrc-beta-docs.netlify.app/releases/>. |
| "Is extern X exposed to Udon in SDK version Y?" | Class Exposure Tree in the editor, or the signature strings in that version's `VRC.Udon.Wrapper.dll` ([Source Map](source-map.md#where-the-code-lives-as-of-sdk-3106-nov-2026)). |
| "Is this officially documented?" | [Official Docs](official-docs.md). |
| "Has anyone implemented this pattern?" | [Community Resources](community-resources.md) — check the Status column; several older projects are archived or superseded by built-in SDK features. |
| "Is this still supported in the current client?" | Official docs first, then community Discord. |

VRChat does not publish a complete API surface for the Udon VM or the SDK runtime. This reference, the source, and the community fill the gap.
