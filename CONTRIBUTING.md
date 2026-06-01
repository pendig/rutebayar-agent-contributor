# Contributing

Thanks for helping improve the Rute Bayar Agent & Contributor skill.

## Development Flow

1. Create a small branch.
2. Update `SKILL.md` for core workflow changes.
3. Put detailed or conditional guidance in `references/`.
4. Keep examples concise and credential-free.
5. Validate before opening a PR.

## Validate

Use the Skill Creator validator when available:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py .
```

If your Python environment does not have `PyYAML`, use a temporary virtual environment:

```bash
python3 -m venv /tmp/rutebayar-skill-validate
/tmp/rutebayar-skill-validate/bin/python -m pip install PyYAML
/tmp/rutebayar-skill-validate/bin/python ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py .
```

## Pull Request Checklist

- No secrets or credentials are included.
- `SKILL.md` frontmatter remains valid.
- References are linked from `SKILL.md`.
- Provider guidance is consistent with Rute Bayar docs and official provider docs.
- README install examples still work.
