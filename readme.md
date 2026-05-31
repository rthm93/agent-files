# Agent Files

This repository is a local Codex plugin installed as `agent-files@rthm93`. It packages a staged
development workflow for Codex along with bundled agent profile prompts used by
that workflow.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin manifest.
- `skills/orchestrated-dev-workflow/`: skill for running a requirements,
  requirements review, architecture, plan approval, coding, and testing workflow.
- `agents/`: five bundled Codex agent TOML profile prompts used by the workflow:
  `requirements_gatherer`, `requirements_reviewer`, `architect`, `coder`, and
  `tester`.
- `docs/language-conventions/`: supporting coding convention notes.

The `orchestrated-dev-workflow` skill expects all five TOML profile prompt files
to be available in this plugin's `agents/` directory. These files are not
automatically registered as custom Codex subagent types. The skill uses Codex's
built-in subagent roles and instructs the orchestrator to inject the relevant
TOML profile contents into each spawned subagent prompt.

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

6. Enable the installed plugin named `agent-files@rthm93`.

The plugin manifest lives at `.codex-plugin/plugin.json` and declares the
bundled skills with `skills: "./skills/"`. It does not register the TOML files
under `agents/` as subagent roles.

## Local Validation

Validate the plugin manifest from this checkout with:

```bash
python3 /home/lantz/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py /mnt/d/Git/agent-files
```
