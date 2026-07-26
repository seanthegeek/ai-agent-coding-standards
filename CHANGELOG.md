# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- "Review discipline" section in `templates/AGENTS.md` — five themes distilled from real multi-agent review cycles: review prose as prose, nothing is pre-verified, check claims against what they range over, verify what CI enforces, and end with a fresh-context review
- "Testing" section in `templates/AGENTS.md` — both halves of exclusive-claim tests, regression tests proven against the unfixed code, and confirming end-to-end runs execute the edited code — plus two additions to the "Check claims against what they range over" review theme (state updates ordered after the fallible action they track; equal handling of return-value and exception failure paths), distilled from [domainaware/parsedmarc#863](https://github.com/domainaware/parsedmarc/pull/863). Restructures the "Testing and review" section that landed in [#2](https://github.com/seanthegeek/ai-agent-coding-standards/pull/2), whose fresh-context-review rule is superseded by the fuller "End with a fresh-context review" theme
- `templates/AGENTS.md` — shared conventions for AI agents working on Sean's projects
- `templates/CLAUDE.md` — Claude Code wrapper that includes `AGENTS.md`
- `README.md` — overview and usage instructions
- `LICENSE` — MIT
- `.vscode/settings.json` — cSpell dictionary for project-specific terms
