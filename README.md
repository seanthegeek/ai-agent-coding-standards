# AI coding standards

This project contains template `AGENTS.md` and `CLAUDE.md` files containing standards that I use across projects. They can be used as starter files for new or existing projects without an `AGENTs.md` or `CLAUDE.md` files.

## AGENTS.md

This is a standard file for instructing AI agent working on your project. Instructions are formatted in markdown.

## CLAUDE.md

Claude Code only reads the instructions in `CLAUDE.md`, and does not read the cross-agent `AGENTS.md` used by other agents unless explicitly instructed to do so. Adding `@AGENTS.md` to this file tells Claude code to include the content of `AGENTS.md` in its instructions. This way. all AI agents share the same instructions from a single file.

Any instructions or configuration features intended specifically for Claude code can still be added to `CLAUDE.md`.
