# AI Development Guide for Pet Crate Simulator

You are working on a s&box game project. Be a careful, practical s&box developer: read the existing code before changing it, use the engine APIs correctly, and verify your work before handing it back.

## First Steps

Before editing code, inspect the local project. Learn the current component patterns, prefab layout, networking style, and naming conventions from the files already in the repo.

Use these references when you are unsure about s&box APIs:

- Official s&box docs index: https://sbox.game/llms.txt
- s&box public source: https://github.com/Facepunch/sbox-public
- Facepunch sandbox project examples: https://github.com/Facepunch/sandbox
- s&box API reference pages when you need exact method names or component behavior.

Do not guess engine APIs when the answer can be checked locally or in official docs. s&box APIs change, and small naming mistakes can waste a lot of time.

## Working Style

- Explain what you are checking and why while you work, especially when reading files, touching prefabs, or changing networking.
- Be proactive. If the user asks for a fix or feature, implement it instead of only proposing a plan.
- Keep changes scoped to the request. Do not refactor unrelated systems unless the refactor is necessary to finish safely.
- Preserve the user's edits. The worktree may be dirty; never reset, revert, or overwrite changes you did not make unless the user explicitly asks.
- Prefer simple, readable component code over clever abstractions.
- If a design choice is ambiguous, follow the patterns already used in this project.

## s&box Project Notes

- Components usually inherit from `Component`; interaction components may inherit from project-specific base classes such as `Interactable`.
- Use `[Property]` for editor-configurable values.
- Use `[Sync]` for state that needs to replicate.
- Use RPCs deliberately:
  - `[Rpc.Host]` for work that must execute on the host.
  - `[Rpc.Owner]` for work that must execute only on the owning client.
  - `[Rpc.Broadcast]` for cosmetic or shared events that all clients should see.
- Avoid hiding gameplay side effects in UI update methods. UI methods like healthbar refreshes should update UI, not award money or mutate gameplay state.
- For local-only player data, check the existing singleton/reference pattern before adding a new one.

## Networking Guidance

Think carefully about who owns the object and who is allowed to mutate state.

- Player money is local-player-owned data. Award money through `PlayerData` using the existing RPC flow instead of directly changing another client's local value.
- If a player interaction should reward the interacting player, get the `PlayerData` from the interacting player's controller/component path and call the appropriate `PlayerData` method/RPC.
- Keep authoritative gameplay state separate from cosmetic animation.
- For smooth network feel, drive important replicated transforms from the owner/host, but broadcast cosmetic events when remote clients should play the same hit, flash, lunge, or bob animation.
- Do not rely on every client being allowed to set another player's local data.

## Prefab Safety

Be extremely careful editing `.prefab` files. They are JSON-like, but s&box relies on metadata fields such as `__type`, `__guid`, `__version`, and component structure.

- Do not round-trip prefab files through generic JSON serializers unless you have verified they preserve all s&box metadata exactly.
- In particular, never strip component `__type` fields. If those are missing, s&box will show missing components and models may stop rendering.
- Prefer small text patches or carefully targeted edits for prefab changes.
- After editing prefabs, validate that:
  - The file still parses.
  - Each component still has the correct `__type`.
  - Model renderers still use `Sandbox.ModelRenderer`.
  - Model paths still point to valid `.vmdl` assets.
  - Child object transforms, especially rotations and scales, are intentional.

## Assets and UI

- When rendering images from `.razor` UI using project assets, use `.vtex` textures. A plain `.png` may work in editor but fail for clients outside the editor.
- The user can create `.vtex` assets in s&box by right-clicking an image in the asset browser and choosing the texture creation option.
- Reference actual project assets where possible. Do not invent asset paths.

## Current Project Knowledge

These patterns are important in this project:

- `InteractGivePlayerCoin` handles destructable-style interact targets that lose health when the player presses Use.
- Money rewards should go through `PlayerData`, not through UI refresh code.
- `PlayerData.AddMoney(...)` applies pet coin multipliers through `PetFramework`.
- `PetFramework.EquipPet(GameObject prefab)` should clone the prefab before equipping it.
- Equipped pets are stored under the player prefab/player object, and the player prefab owns `PetFramework`.
- Pet prefabs live under `Assets/Prefabs/Pets`.
- Pet model source files live under `Assets/3rdparty/kenny.nl`.
- Pet prefabs should have:
  - A root object with `PetComponent`.
  - An `AnimalModel` child.
  - A `Sandbox.ModelRenderer` on the model child.
  - The model child rotation set intentionally, usually identity: `0,0,0,1`.
- Pet coin multipliers are configured on `PetComponent`.
- Pets should orbit the player evenly, move toward their positions instead of teleporting, touch the ground when possible, bob while moving, and lunge/bob toward the destructable the player attacks.

## Animation and Feel

For simple game-feel animation:

- Prefer time-based interpolation and easing over instant jumps.
- Keep the gameplay object stable when possible and animate child visual objects for bounce, squash, lunge, flash, or bob effects.
- Separate the base/rest transform from the visual offset so effects can cleanly return to zero.
- Use configurable `[Property]` values for distances, durations, speeds, colors, and multipliers so the user can tune them in the editor.
- For hit feedback, broadcast cosmetic effects when everyone should see them.
- For pets or followers, a simple "move toward target position" behavior is often enough; do not add pathfinding unless the user asks for it.

## Validation Checklist

Before finishing a coding task:

- Read the final diff.
- Check for accidental unrelated edits.
- Validate prefab/component metadata if any assets were touched.
- Run a build or the closest available validation command.
- If the build cannot run because of local permissions or environment issues, say that clearly and report what validation did run.
- Mention changed files and the important behavior changes in the final response.

When you are ready to begin after reading this file and the project context, say:

`Ok, lets get started with development! Tell me what to do!`
