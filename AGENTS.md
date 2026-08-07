# Grandoria Game Instructions

## Architecture

- Grandoria is a 2D top-down multiplayer RPG.
- GDevelop is the client, and Colyseus is authoritative for real-time gameplay.
- Firebase is the target system only for authentication and persistent data.
- Legacy RTDB systems are temporary migration code. Remove them subsystem by subsystem only after the equivalent Colyseus system is validated, and never reactivate legacy Firebase or local authority accidentally.

## Canonical Project and Validation

- The canonical project is `RPG-2D-project-organized-by-systems-pre-combat-english.json`.
- The initial scene is `Scene_Menu`.
- `RPG-2D-projeto.json.autosave` is not canonical and must never be used as the project source.
- `Saves_Gdevelop/`, `jsonBackups/`, and `git_saves/RPG` are historical material. Do not edit them unless explicitly requested.
- `js/colyseus.js` is the required Colyseus 0.17.10 SDK. Do not edit it manually.
- Keep GDevelop closed while modifying its JSON, and parse every modified GDevelop JSON before completing a task.
- Preserve exact scene, object, variable, animation, resource, and saved-data identifier names unless every reference is verified.

## Working Rules

- Preserve existing functionality and user-owned uncommitted changes.
- Never commit or push without explicit user authorization.
- Never expose or edit secrets.
- Keep code, identifiers, comments, technical documentation, and logs in English. Keep player-facing interface text in Portuguese.
- Prefer generic, reusable implementations for multiple maps, players, and monster types.
- Do not introduce temporary hardcoded entity IDs when a generic implementation is possible.
