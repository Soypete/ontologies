# MemPalace Activity Log

Date: 2026-05-15

## Initialization

```bash
mempalace init ~/code/opensource/soypete_tech_ontologies --yes
```

Result:
- Wing: `soypete_tech_ontologies`
- Auto-detected rooms from folder structure:
  - `thesaurus` - files from thesaurus/
  - `education` - files from education/
  - `software` - files from software/
  - `social` - files from social/
  - `general` - files that don't fit other rooms

## Mining

```bash
mempalace mine ~/code/opensource/soypete_tech_ontologies
```

Result:
- Files processed: 2 (README.md, AGENTS.md)
- Drawers filed: 6

## Project Mode Mining

```bash
mempalace mine ~/code/opensource/soypete_tech_ontologies --mode projects
```

Result:
- Files skipped (already filed): 2
- No new drawers added

## Status

```bash
mempalace status
```

Total drawers in system: 318

```
WING: experiments
  ROOM: sessions                 1 drawers

WING: pedro_bots
  ROOM: src                    139 drawers
  ROOM: general                 43 drawers
  ROOM: scripts                 38 drawers
  ROOM: charts                  31 drawers
  ROOM: testing                 23 drawers
  ROOM: k8s                     13 drawers
  ROOM: alembic                 13 drawers
  ROOM: migrations              11 drawers

WING: soypete_tech_ontologies
  ROOM: software                 6 drawers
```

## Query Examples

To query this palace:
```bash
mempalace search "what Go courses are available"
mempalace search "competency questions for software development"
```

Config saved to: `~/code/opensource/soypete_tech_ontologies/mempalace.yaml`