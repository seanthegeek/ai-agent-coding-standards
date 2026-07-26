# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- A "Testing and review" section in `templates/AGENTS.md`: fresh-context diff review before every pull request, ordering of state updates around fallible actions, equal handling of return-value and exception failure paths, both halves of exclusive-claim tests, regression tests proven against the unfixed code, and confirming end-to-end runs execute the edited code — distilled from review cycles in [domainaware/parsedmarc](https://github.com/domainaware/parsedmarc) (most recently [#863](https://github.com/domainaware/parsedmarc/pull/863))
- `templates/AGENTS.md` — shared conventions for AI agents working on Sean's projects
- `templates/CLAUDE.md` — Claude Code wrapper that includes `AGENTS.md`
- `README.md` — overview and usage instructions
- `LICENSE` — MIT
- `.vscode/settings.json` — cSpell dictionary for project-specific terms
