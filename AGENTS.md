# AGENTS.md

Contributor guide for this repository.

## What it is

A how-to for holiday-themed porch lights in Home Assistant. The deliverables are configuration files a reader copies into their own instance. There is no build step and no application code.

## Layout

- `automations/`: one automation per file, in the format the automation editor's YAML mode accepts
- `scripts/`: one script per file, in the script editor's YAML format; holds the rules table and selection logic
- `scenes/`: example scenes, in the format of `scenes.yaml`
- `packages/`: Home Assistant packages (the optional theme helper)
- `docs/adr/`: architecture decision records

## Checks

- YAML must parse: `python -c "import yaml,sys; [yaml.safe_load(open(f)) for f in sys.argv[1:]]" automations/*.yaml scripts/*.yaml scenes/*.yaml packages/*.yaml`
- Test changes to the selection template in Developer tools > Template against a real instance, with a hand-built events list for dates such as 31 October, 20 December and 4 July, before committing.

## Conventions

- Keep examples generic: no real entity names, area ids, calendar URLs or IP addresses from a specific house.
- Use `color_temp_kelvin`, never `color_temp` (mireds), in light actions.
- Every holiday is a row in the `rules` table plus a scene. Do not add per-holiday branches.
- Prefer native triggers and conditions over templates where one exists.
- Explain non-obvious steps with a comment or `note:` next to the step.
