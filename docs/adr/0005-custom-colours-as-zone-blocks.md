# ADR-0005: Custom colours as zone blocks, built-in themes through effect_move

Status: accepted (2026-11-18). Supersedes the scene-based looks in ADR-0001.

## Context

A LIFX strand can run a built-in theme as a moving effect, but I also wanted team colours and a "paint my own colours" option with exact hues. Two behaviours of the LIFX integration get in the way:

- `lifx.paint_theme` with a custom palette blends hues across the zones, so red, green and blue become every colour in between.
- `lifx.effect_move` only paints when it is given a built-in theme. With no theme it scrolls the colours already on the zones.

Scenes could not describe either. They hold a fixed state per light, not an effect or a zone pattern.

## Decision

Keep the looks in a `themes` table in the apply script, and give each strand entry one of two modes:

- **move:** call `lifx.effect_move` with a built-in LIFX theme name and a speed.
- **blocks:** paint exact colours with `lifx.set_state` on a list of zones, four zones per colour, repeating along the strand, then call `lifx.effect_move` with no theme to scroll them.

The two paint options are built at run time from helpers. Paint a LIFX theme uses move, or `lifx.paint_theme` when the speed is 0. Paint my colours uses blocks from three hue sliders. Bulbs run a separate loop script. For built-in themes it repaints the bulbs from the strand's LIFX theme every 5 seconds. For custom colours it rotates the colour list every 20 seconds, with each bulb one step apart.

## Alternatives considered

- **Scenes.** They cannot start an effect or paint zones, and they tie each look to a fixed list of lights.
- **`paint_theme` with a custom palette.** It blends, so exact colours are lost.
- **Using only built-in themes.** There is no built-in theme for a team's colours.
- **A slow fade on the bulbs.** The colours in a LIFX theme are random picks, and a slow fade let two matching or pale bulbs last too long. Instant changes every few seconds keep any bad pair brief.

## Consequences

The blocks mode paints zones 0 to 119 and the strand ignores zones it does not have, so a longer strand is covered without a change. Effect speed is seconds for the whole strand, with 60 as the maximum, so a slow block scroll uses 60. LIFX themes carry their own brightness per colour, so theme-driven bulbs and strand zones vary in brightness by design. A two-second button hold on the strand wipes the effect without an event, which is why the Fix button in ADR-0004 exists.
