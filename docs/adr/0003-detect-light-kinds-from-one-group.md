# ADR-0003: Detect light kinds from capabilities, through one group

Status: accepted (2026-11-18)

## Context

The porch has a LIFX multizone strand, two colour bulbs and a white-only bulb on the back porch. The strand needs effects and zone paints, the colour bulbs need a colour loop, and the white-only light can only change temperature and brightness. The first version named two bulbs in a scene. Adding the strand meant editing every script that listed lights, and a new bulb meant the same again.

## Decision

Keep one light group, `light.outdoor_lights`, as the only list of lights. The scripts read the group's members at run time and sort each available member by what it can do:

- **Strand:** its `effect_list` contains `effect_move`.
- **Colour bulb:** it supports the `hs` colour mode and is not a strand.
- **White-only:** anything else.

Unavailable members are skipped. Adding a light means adding it to the group.

## Alternatives considered

- **One list per kind, in the script.** Three lists to keep in step with the hardware, and a light that changes firmware or mode ends up in the wrong one.
- **Naming lights in each script.** What the first version did. It drifted every time a light was added.
- **Sorting by integration or entity name.** A name is a convention that the next light will not follow. Capabilities are what the code actually needs to know.

## Consequences

The group is the contract: a light that is not in it is never touched by the nightly run. A light that is mid-reconnect has no capabilities to read, so it is skipped until it is available, and the reconnect automation applies the theme to it when it returns. Because the sorting runs on every apply, the cost is a small template evaluated each time.
