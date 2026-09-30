# ADR-0001: One rules table in the script, scenes for the looks

Status: accepted (2026-09-30)

## Context

The porch needs a different look for a dozen holidays, each with its own lead-up window, plus a game-day look and a default. The first version of this setup had one hand-written branch per holiday. Adding or retiming a holiday meant editing several places, and the branches drifted.

## Decision

Keep every holiday in one list of `[summary substring, scene, lead days, display name]` rows under the script's `variables:`. A single template finds the first row that matches an upcoming event and returns its index. The script then reads the scene and the display name from that row. Each look is an ordinary scene, so colours are edited in the scene editor.

## Alternatives considered

- **One `choose` branch per holiday.** Every branch needs the same template condition anyway, so it removes no templating, and each new holiday touches several lines.
- **Colours inline in the script.** Recolouring then means editing YAML, and the game-day look would duplicate the pattern.
- **A blueprint.** The interesting part is the rules data, and a blueprint input for a list of lists is awkward to edit in the UI.

## Consequences

Two template conditions remain (`rule_index >= 0` and `is_game_day`), because no native condition tests a script variable or an action response. The scene is applied through a templated `scene.turn_on` target; the value comes from a closed list in the same file and is guarded against the no-match case. In return there is one place to change a holiday and the display name for a badge comes for free.
