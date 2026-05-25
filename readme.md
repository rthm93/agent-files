# Agent Files

This repository is a local Codex plugin named `agent-files`. It packages a staged
development workflow for Codex along with the agent profile files that workflow
expects.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin manifest.
- `skills/orchestrated-dev-workflow/`: skill for running a requirements,
  requirements review, architecture, plan approval, coding, and testing workflow.
- `agents/`: five Codex agent TOML profiles used by the workflow:
  `requirements_gatherer`, `requirements_reviewer`, `architect`, `coder`, and
  `tester`.
- `docs/language-conventions/`: supporting coding convention notes.

The `orchestrated-dev-workflow` skill expects all five agent TOML profiles to be
available in `agents/`.

## Local Install

1. Make sure this repo is on the machine where Codex runs.
2. Validate the plugin manifest:

   ```bash
   python3 /home/lantz/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py /mnt/d/Git/agent-files
   ```

3. In Codex, open the plugin installer or `/plugins`.
4. Choose the option to install a local plugin from a folder.
5. Use this checkout as the plugin source path:

   ```text
   /mnt/d/Git/agent-files
   ```

6. Enable the installed plugin named `agent-files`.

The plugin manifest lives at `.codex-plugin/plugin.json` and declares the
bundled skills with `skills: "./skills/"`.

## Local Validation

Validate the plugin manifest from this checkout with:

```bash
python3 /home/lantz/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py /mnt/d/Git/agent-files
```
