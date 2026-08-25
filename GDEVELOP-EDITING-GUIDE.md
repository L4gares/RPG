# Grandoria GDevelop Editing Guide

This project keeps game authoring in GDevelop wherever the server can execute an existing generic mechanic safely. The server remains authoritative for access rules, cooldowns, combat, damage, status effects, death, persistence, and synchronization.

## Visual editing in `MAP_1`

The following scene objects are editor-only markers. They are visible in the scene editor and hidden automatically at runtime. Move or resize them directly in GDevelop; do not delete them.

- `Editor_PlayerStatusAnchor`: horizontal center and top position of the local player's buff row.
- `Editor_SkillSlotAnchor`: one marker for each action-bar slot. Keep the entire marker inside `skill_bar`; its position becomes the runtime icon position. Edit its instance variable `slot_size` to change that slot's square icon size without stretching the image.
- `Editor_AttackVfxOrigin`: visual reference point for the character.
- `Editor_AttackVfxAnchor`: four instances with the object variable `direction` set to `up`, `down`, `left`, or `right`. Their offsets from `Editor_AttackVfxOrigin` define the directional skill VFX position.

The Skills window is also laid out directly in the scene editor on `SkillsLayer`. Move and resize these objects without changing code:

- `Skills_WindowPanel`, `Skills_ListPanel`, and `Skills_DetailPanel`
- `Skills_Title`, `Skills_Subtitle`, `Skills_Close`, and `Skills_Footer`
- the four `Skills_RowName`, `Skills_RowState`, and row `UI_SkillIcon` instances
- `Skills_DetailTitle`, `Skills_DetailDescription`, `Skills_DetailMeta`, and the detail `UI_SkillIcon`
- `Skills_EquipButton`, `Skills_KeyButton`, `Skills_PreviousPage`, `Skills_PageText`, and `Skills_NextPage`

The four visible rows are reusable slots, not Warrior-specific skills. Additional class skills are paginated automatically.

Each row `UI_SkillIcon` must keep `ui_role = row`, `row_index` from `0` through `3`, and `icon_size = 48`. The larger detail icon must keep `ui_role = detail`, `row_index = 999`, and `icon_size = 56`. The exporter rejects missing, duplicated, or misplaced authoring objects before they can reach the server.

## Global client configuration

Open **Project Manager > Game settings > Global variables** and edit `gameplay_editor_config`.

- `basic_attack`: unarmed/melee sound resources, volume, and pitch.
- `skills_window`: toggle key, dynamic labels, translations, and text wrapping.
- `skill_hud`: default square icon size, key/cooldown offsets, and cooldown opacity. An individual `Editor_SkillSlotAnchor.slot_size` overrides the default size.
- `player_status_hud`: player buff icon size, spacing, timer offsets, layer, and Z order.
- `monster_status_hud`: selected-monster debuff layout.
- `skill_vfx`: sound distance/volume and VFX cleanup timing.
- `damage_numbers`: colors, duration, rise distance, stacking, and offsets.

Static window text such as `Skills_Title`, `Skills_Subtitle`, and `Skills_Footer` is edited directly on the corresponding text object.

Edit `skill_system_data` to configure the keys that may be assigned to equipped skills and `global_cooldown_ms`. These values are exported to the authoritative server. Movement keys, `F` (Basic Heal), and the Skills-window key remain reserved.

## Editing or creating a skill

Open the global structure `skills_data`. Duplicate an existing skill that uses the closest supported effect template, give it a globally unique ID, then edit its fields.

Common authoring fields:

- `name_display`, `description`, and `ui_order`
- `required_level`, `cooldown_ms`, and `weapon_requirement`
- `enabled`, `available_to_all`, and `loadout_eligible`
- `icon_animation`, `vfx_animation`, `sound_resource`, and `status_icon_override`
- `visual`: VFX anchor, dimensions, directional offset scale, fixed offsets, rotation, Z offset, and fallback duration
- `effects`: authoritative balance values for the selected `effect_template`

Supported authoritative templates:

| `effect_template` | Activation | Required effect fields |
| --- | --- | --- |
| `direct_damage` | `armed_attack` | `damage_multiplier_percent` |
| `damage_stun` | `armed_attack` | `damage_multiplier_percent`, `stun_duration_ms`, `status_id` |
| `multi_hit_bleed` | `armed_attack` | `hit_count` (2-8), `hit_damage_multiplier_percent`, `hit_interval_ms`, `bleed_damage_multiplier_percent`, `bleed_duration_ms`, `bleed_tick_ms`, `status_id` |
| `self_armor_percent` | `instant` | `armor_percent_bonus`, `duration_ms`, `status_id` |
| `self_heal_percent` | `instant` | `heal_percent_max_hp` |

For a skill icon, add an animation to the global sprite `UI_SkillIcon` and use that exact animation name in `icon_animation`. The same animation catalog feeds both the Skills window and the action bar, so the icon only needs to be imported once.

For a skill VFX, add an animation to the global sprite `VFX_Skill` and use that exact animation name in `vfx_animation`. Use `none` when the skill has no VFX.

For a status icon, add an animation to `UI_StatusIcon`, add or edit the status under `status_effects_data`, and reference its ID from the skill effect. A status contains `display_name`, `icon_animation`, and `kind` (`buff` or `debuff`); this catalog is exported to the server together with the skills.

For a sound, import the audio through GDevelop's Resources panel and set `sound_resource` to the exact resource name. Use `none` when the skill has no sound.

## Assigning a skill to a class

Open `classes_data`, select the class, then add the skill ID under its `skills` structure as a Boolean set to `true`. The client window and the authoritative server class catalog both use this relation.

To create another class, duplicate the existing class structure, use a unique class ID, edit its display fields/base attributes, and replace its `skills` entries. No client UI code is required.

## Synchronizing authoritative configuration

After changing classes, skills, items, monsters, quests, slots, or world configuration in GDevelop, regenerate the server catalogs from the server directory:

```powershell
node scripts/export-gdevelop-world.mjs --project "C:\Users\danie\Projects\Grandoria\grandoria-game\RPG-2D-project-Grandoria-Colyseus-authoritative-inventory-equipment.json"
node scripts/export-gdevelop-world.mjs --project "C:\Users\danie\Projects\Grandoria\grandoria-game\RPG-2D-project-Grandoria-Colyseus-authoritative-inventory-equipment.json" --check
npm test
```

The export now generates and validates skills, status effects, allowed skill keys/global cooldown, classes, slots, items, monsters, quests, and the world artifact. It also verifies that referenced icons, VFX frames, sounds, status IDs, and class skills exist in the GDevelop project.

Visual-only changes under `gameplay_editor_config`, `visual`, editor markers, object layouts, animations, or client audio do not change server authority. Running the exporter and tests is still recommended before publishing a new build.

## Current intentional limits

`cast_time_ms`, resource consumption, and attribute scaling remain disabled until their authoritative server mechanics exist. Keep `cast_time_ms` at `0`, `resource_type` at `none`, `resource_cost` at `0`, and scaling at `none`/`0`.

A skill using one of the supported templates can be added and balanced entirely in GDevelop. A genuinely new mechanic (for example, projectile simulation, teleportation, summons, area targeting, mana consumption, or a new crowd-control rule) requires one generic server implementation first; subsequent skills can then reuse that template without per-skill code.
