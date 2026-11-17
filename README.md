# Final Project

This repo uses [AI-DLC](https://github.com/awslabs/aidlc-workflows) with Claude Code. The workflow's skills, agents and hooks are committed under `.claude/`, and the shared project method is under `aidlc/spaces/`.

## Collaborator setup

1. Install the AI-DLC CLI once per machine (the hooks run through it):
   ```
   curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh
   ```
   Make sure `~/.local/bin` is on your PATH.
2. Clone this repo and open Claude Code in it. Approve the project hooks when prompted.
3. Run `/aidlc --doctor` to check the setup.

Project notes and rules that every AI tool reads live in `aidlc/spaces/default/memory/project.md`. Personal overrides go in `.claude/settings.local.json` (gitignored; copy the `.example`).
