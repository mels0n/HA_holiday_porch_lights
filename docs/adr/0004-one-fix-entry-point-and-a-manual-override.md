# ADR-0004: One Fix entry point, and a manual override that holds until 23:59

Status: accepted (2026-11-18)

## Context

Several things can leave the lights in the wrong state: a power blip, a strand re-calibration that wipes the effect without telling Home Assistant, a stuck colour loop, a calendar change, a manual change from a dashboard. Each had its own repair, and the repairs started to differ. I also wanted to pick a theme by hand for a party without editing anything, and without having to remember to undo it.

## Decision

Everything that means "make the lights right for now" goes through one script, `outdoor_lights_fix`. It stops the bulb loop, runs the resolve script again, and then either applies the theme or turns the group off. It applies if it is between sunset minus 30 minutes and 23:59, if it was called with `turn_on: true`, or if a manual pick is set.

The manual pick is a select, `outdoor_theme_override`. Anything other than Auto wins over every calendar and the live match. The nightly automation sets it back to Auto at 23:59, so a pick lasts the evening. A change to the select, or to a paint setting while the select is not Auto, runs Fix. Pressing Run on the nightly automation reaches its default branch and also runs Fix.

## Alternatives considered

- **A separate repair per symptom.** Easy to write, and impossible to know which one to press.
- **A manual pick that holds until cleared.** A theme left on by accident would run for days. A reset at the time the lights turn off is predictable.
- **A pick that lasts only until the next calendar event.** The calendar has no events on most nights, so there is no natural end point.
- **A timer for the pick.** One more helper, and it still needs an answer for what happens at 23:59.

## Consequences

A pick made after 23:59 keeps the lights on until the next 23:59, or until Auto or All off is chosen. Resetting the select at 23:59 fires its own trigger, which runs Fix, which turns the lights off a second time. That is harmless. The cost of a single entry point is that Fix does more work than a narrow repair would, and I accept that because it is always safe to press.
