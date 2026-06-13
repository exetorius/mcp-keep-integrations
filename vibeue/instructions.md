# VibeUE — MCP Quick Reference

You have direct access to Unreal Engine 5.7 via the VibeUE Python API.

## Python Basics

```python
import unreal  # lowercase — NOT 'Unreal'
```

**DEPRECATED:** `unreal.EditorLevelLibrary` is removed in UE 5.7+. Use `unreal.get_editor_subsystem(unreal.EditorActorSubsystem)`. `get_all_level_actors_of_class` does NOT exist — use `get_all_level_actors()` + `isinstance()`.

## Skills — Load Before Working

Always load the domain skill before executing tasks:
```
manage_skills(action="load", skill_name="blueprints")   # then discover, then execute
manage_skills(action="list")                             # see all available skills
```

Pattern: load skill → read `vibeue_classes` → `discover_python_class(...)` → `execute_python_code(...)`. Never guess method names — confirm via discovery first.

## Assets — Use manage_asset, Not Python

**For find / open / save / duplicate / move / delete — use the `manage_asset` tool, not Python.**

| Goal | Call |
|------|------|
| Find by name | `manage_asset(action="search", search_term="BP_Enemy", asset_type="Blueprint")` |
| Confirm path | `manage_asset(action="find", asset_path="/Game/AI/ST_Cube")` |
| List folder | `manage_asset(action="list", path="/Game/AI")` |
| Open in editor | `manage_asset(action="open", asset_path="...")` |
| Save | `manage_asset(action="save", asset_path="...")` |
| Move/rename | `manage_asset(action="move", source_path="...", destination_path="...")` |
| Delete | `manage_asset(action="delete", asset_path="...")` |

Never guess paths. Never duplicate+delete to simulate a move — that breaks references.

## Critical Rules

**Non-destructive:** Never remove-and-recreate to change a value. Use a direct setter. If no setter exists, stop and report — don't fake it with destructive fallbacks.

**Check before create:** Use `*_exists()` or `manage_asset(action="find", ...)` before creating assets to avoid duplicates.

**Compile after structure changes:** After adding variables, functions, or components: `unreal.BlueprintService.compile_blueprint(path)`.

**Verify graph edits:** A successful tool call is not enough. Re-read with `get_nodes_in_graph()` / `get_connections()` / `compile_blueprint(...).success` before claiming done.

**Always use full paths:** `/Game/Blueprints/BP_Name` not `BP_Name`.

## Loop Prevention — CRITICAL

Self-monitor for repeated failures. Track OUTCOMES, not just arguments.

- Same tool call + same result appearing twice → STOP, do not retry
- Same error across 3 different code variations → STOP, report to user
- Max 3 attempts at the same operation
- Max 2 discovery calls for the same function
- After 2 failed attempts: explain what you tried and ask the user

Never retry blindly. One alternative approach after failure — if that also fails, stop.
