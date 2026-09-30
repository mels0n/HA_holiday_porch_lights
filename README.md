# Home Assistant Holiday Porch Lights

## What this is

A how-to for holiday-themed porch lights in Home Assistant. Every evening at dusk the porch lights come on in the colours of the nearest holiday, or in your team's colours on a game day, or plain warm white on an ordinary night. They switch off at 23:59.

This is a cleaned-up, generic version of the setup running on my own porch, not a copy of my live configuration. Entity names are generic and anything specific to my house has been removed. On my porch the game-day look is for STL CITY SC; use whichever team, or none, suits you.

**How it works**

- **Precedence.** A holiday beats a game day, and a game day beats the default. A holiday lead-up (Christmas starts 25 days out, Halloween 15) therefore keeps its look even on a match night.
- **One rules table.** `scripts/porch_lights_with_holiday.yaml` holds a table of `[summary substring, scene, lead days, display name]` rows. It is the only place a holiday is defined. Adding one is a new row and a new scene.
- **Two calendars.** The holiday calendar is the core Holiday integration. The game-day calendar is any calendar of your team's schedule (an ICS feed added with the Remote Calendar integration works). Any event today counts as game day, home or away.
- **Scenes are the looks.** Each look is an ordinary scene, so you can recolour it in the scene editor without touching the script.
- **Survives a missing calendar.** Both calendar lookups continue on error. If one is briefly unavailable the script falls through to the next branch and the porch still lights.
- **Theme name (optional).** The script records the display name of the chosen look in `input_text.porch_theme`, so a dashboard or a public badge can show what the porch is doing tonight.

| File | Purpose |
|---|---|
| `automations/nightly_porch_lights.yaml` | The schedule: run the script 30 minutes before sunset, turn the lights off at 23:59, and re-apply after a restart during the evening |
| `scripts/porch_lights_with_holiday.yaml` | The rules table, the calendar lookups, and the look selection |
| `scenes/porch_scenes.yaml` | Example scenes for eight looks (seven holidays and a game day) |
| `packages/porch_theme.yaml` | Optional `input_text.porch_theme` helper |

**Entities to replace**

| In this repository | Replace with |
|---|---|
| `calendar.holidays` | Your Holiday integration calendar (for example `calendar.united_states`) |
| `calendar.team_schedule` | Your team's schedule calendar, or delete the game-day pieces |
| `light.porch_left`, `light.porch_right` | Your two porch bulbs (any colour-capable lights) |
| `scene.porch_*` | The scenes you create from `scenes/porch_scenes.yaml` |
| `input_text.porch_theme` | Created by `packages/porch_theme.yaml` (optional) |

The default look uses `light.turn_on` with `color_temp_kelvin: 2700`, so it works with any colour-temperature light, not one brand.

## Install

**Requirements:** Home Assistant 2024.10 or later, the core Holiday integration, and two colour-capable lights. The Sun integration is on by default.

1. **Holiday calendar.** Settings > Devices & services > Add integration > Holiday, pick your country and region. Note the calendar entity it creates and use it in place of `calendar.holidays`. If a holiday you want (Halloween, Valentine's Day, St. Patrick's Day) is missing from the calendar, check the integration's options for extra holiday categories.
2. **Team calendar (optional).** Add your team's schedule as a calendar, for example with the Remote Calendar integration and the ICS link the league publishes. Use its entity in place of `calendar.team_schedule`. Keep the same entity id from year to year by editing the URL in place: deleting and re-adding the calendar can give it a new id, and the script would need updating.
3. **Scenes.** Copy the entries in `scenes/porch_scenes.yaml` into your `scenes.yaml` (or recreate them in the scene editor), swapping in your two light entities. Reload scenes (Developer tools > YAML).
4. **Theme helper (optional).** Enable packages if you have not already, by adding this to `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
   Copy `packages/porch_theme.yaml` to `/config/packages/` and restart. Or create a Text helper named `porch_theme` in the UI (Settings > Devices & services > Helpers) with a maximum length of 100. If you skip it, delete the `input_text.set_value` step at the end of the script.
5. **Script.** Settings > Automations & scenes > Scripts > Add script > three-dot menu > Edit in YAML. Paste `scripts/porch_lights_with_holiday.yaml`, adjust the entities, save. Its entity id becomes `script.porch_lights_with_holiday`, which the automation calls.
6. **Automation.** Settings > Automations & scenes > Create automation > three-dot menu > Edit in YAML. Paste `automations/nightly_porch_lights.yaml`, save.
7. **Try it.** Run the script from the UI. Check Settings > Automations & scenes > Scripts > this script > Traces to see which branch it took.

**Adding a holiday.** Look up the exact event summary the calendar uses (Developer tools > Actions > `calendar.get_events`), add a row to `rules` with a distinctive substring of it, and make the scene. Put rows with a long lead-up above rows with a short one.

**The gotchas.** Three things went wrong when I moved to the core Holiday calendar, and each has a quiet failure mode.

1. **Core calendar entities have no `events` attribute.** `state_attr('calendar.holidays', 'events')` returns nothing, so a find-and-replace of an old calendar entity id fails silently and every night falls through to the default. The events have to be fetched with the `calendar.get_events` action into a `response_variable`.
2. **`duration:` counts from now.** With `duration:` the window starts at the current moment and returns an empty list whenever the next holiday is farther out than the window. The script uses an explicit `start_date_time` of today at 00:00 and an `end_date_time` 25 days later (the longest lead, for Christmas). Anchoring to the start of today also makes the day-of check reliable whatever time the script runs.
3. **Holiday summaries are messy, so match by substring.** Martin Luther King Jr. Day arrives as one combined string (`Birthday of Martin Luther King, Jr.; Martin Luther King Jr. Day`, matched with `King`). Presidents Day is `Washington's Birthday`. Independence Day appears twice, including `Independence Day (observed)`. The calendar also carries many holidays with no scene (Christmas Eve, New Year's Eve, Good Friday, Groundhog Day, state holidays) that the substring list ignores by design. Never compare with `==`.

One more shape to know: `calendar.get_events` returns `start` as a plain ISO string (`"2026-09-07"` for an all-day event), not a dict. The script reads the first ten characters and does date-only maths, which avoids the naive-versus-aware datetime error you get from comparing an all-day event with `now()`.

**Tuning.** Lead days, colours and display names live in `rules` and the scenes. The start (30 minutes before sunset) and end (23:59) times live in the automation. To end the evening at a different time, change the `time` trigger (and the matching `before:` in the restart branch).

## Publish live values (optional)

The script writes the theme name (for example `Halloween`, `Game day`, or `Default white`) to `input_text.porch_theme`, and it keeps that value until the next evening. That makes it publishable as a live badge, the same way the circadian-lights how-to publishes its sensors: see "Publish live values" in [HA_circadian_lights](https://github.com/mels0n/HA_circadian_lights) for the small read-only proxy. Add `input_text.porch_theme` to the proxy's `$entities` allow-list, then point a shields.io [dynamic JSON badge](https://shields.io/badges/dynamic-json-badge) at it with a query like `$['input_text.porch_theme']`.

The proxy only ever reads the entity ids on its allow-list and only returns their `state`, so a caller cannot use it to read anything else.

## Secrets

Nothing in this repository needs credentials. If you set up the optional proxy, it needs a Home Assistant long-lived access token, covered in the circadian-lights how-to linked above. Never commit a token, and keep calendar ICS links out of public repositories too: some leagues publish per-user links.

## License

MIT. See [LICENSE](LICENSE).
