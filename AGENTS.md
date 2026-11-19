# AGENTS.md

Contributor guide for this repository.

## What it is

A how-to for holiday-themed outdoor lights in Home Assistant, built around LIFX. The deliverables are configuration files a reader copies into their own instance. There is no build step and no application code.

## Layout

- `automations/`: one automation per file, in the format the automation editor's YAML mode accepts
- `scripts/`: one script per file, in the script editor's YAML format. `outdoor_resolve_theme.yaml` holds the rules table, `outdoor_apply_theme.yaml` holds the theme table and drives the lights
- `packages/`: Home Assistant packages (all helpers, the light group and the available-lights sensor)
- `dashboards/`: a sections view to paste into a dashboard
- `docs/adr/`: architecture decision records

## Checks

- YAML must parse: `python -c "import yaml,sys; [yaml.safe_load(open(f)) for f in sys.argv[1:]]" automations/*.yaml scripts/*.yaml packages/*.yaml dashboards/*.yaml`
- Test changes to the resolve template in Developer tools > Template against a real instance, with a hand-built events list for dates such as 31 October, 20 December and 4 July, before committing.
- Test changes to the apply script by running it against a real strand and bulbs, for a built-in theme, a custom colour theme and white.

## Conventions

- Keep examples generic: no real entity names, area ids, calendar URLs or IP addresses from a specific house.
- Use `color_temp_kelvin`, never `color_temp` (mireds), in light actions.
- Every holiday is a row in the `rules` table plus an entry in the `themes` table. Do not add per-holiday branches.
- Lights are named only in the `light.outdoor_lights` group. Scripts sort members by capability at run time.
- Prefer native triggers and conditions over templates where one exists.
- Explain non-obvious steps with a comment or `note:` next to the step.
