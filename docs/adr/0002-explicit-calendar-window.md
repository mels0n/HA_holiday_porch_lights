# ADR-0002: Fetch calendar events with an explicit window anchored to today

Status: accepted (2026-09-30)

## Context

Core calendar entities do not expose an `events` attribute, so events must come from the `calendar.get_events` action. That action accepts a `duration:`, which counts forward from the current moment.

## Decision

Pass `start_date_time` as today at 00:00 and `end_date_time` as today at 00:00 plus 25 days (the longest lead in the rules table). For the game-day calendar the window is today at 00:00 to one day later. Both lookups use `continue_on_error: true`, and the selection templates guard with `is defined`.

## Alternatives considered

- **`duration:`.** Returns an empty list whenever the next holiday is farther away than the duration, and the day-of check depends on the clock time the script runs.
- **Reading a calendar entity's state.** Only the next single event is visible, so a holiday sitting behind another one is missed.
- **Per-holiday sensors.** They depended on a third-party integration that can disappear, and did.

## Consequences

The maximum lead is tied to the window. Raising a lead above 25 days means raising the window in the same commit. Starting at 00:00 makes an all-day event today visible at any hour.
