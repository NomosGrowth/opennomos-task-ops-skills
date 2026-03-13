# OpenNomos Task Ops Skills

Reusable Codex skill files for operating OpenNomos project tasks.

## Included Skills

- `opennomos-task-ops`

## Structure

```text
skills/
  manifest.json
  opennomos-task-ops/
    SKILL.md
    agents/
      openai.yaml
    references/
      templates.md
```

## Install

```bash
git clone https://github.com/<your-org>/opennomos-task-ops-skills.git
mkdir -p "$CODEX_HOME/skills"
cp -R opennomos-task-ops-skills/skills/* "$CODEX_HOME/skills/"
```

## Validate (optional)

```bash
python3 "$CODEX_HOME/skills/.system/skill-creator/scripts/quick_validate.py" \
  "$CODEX_HOME/skills/opennomos-task-ops"
```
