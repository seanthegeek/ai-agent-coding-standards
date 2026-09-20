# AGENTS.md

## Project overview

Always start the `AGENTS.md` file with a description and purpose of the project. This gives the agent key context.

## Conventions

These rules apply to anyone — human or agent — making changes to this repo. They are intentionally checked in (rather than living in any one agent's private scratch memory) so that every collaborator picks them up the same way.

- **Wait for explicit commit AND push permission on the default branch — these are separate grants.** Finish the implementation, run the tests, summarize the diff, then **stop and ask**. The author decides when a change is ready to land; auto-committing makes review noisier and harder to reverse. "Commit this" mid-session counts as permission for that one commit, not a standing grant — and crucially, permission to commit is NOT permission to push. Pushing publishes the change to the remote where collaborators / CI / production deploys can pick it up, and is much harder to walk back than a local commit. Wait for an explicit "push it" before `git push`. If the prior commit was itself unauthorized, do NOT push it to "tidy up" — surface the situation and let the author decide whether to keep, amend, or reset.
- **Self-test before every `git commit`:** has the author typed "commit" (or an unambiguous equivalent — "ok to commit", "commit this", "commit and push") in a present-tense imperative since your last commit? If no, **ask**. Conditional phrasings like "if everything works we can push" or "we could commit this" or "if it looks good ..." are NOT authorizations — they are plans you must confirm before acting on. Treat the literal text of the user's last message as the source of truth, not your own interpretation of where the conversation is going.
  - **Self-test before every `git push`:** has the author typed "push" since your last push? Same rule. Permission to commit is NEVER permission to push.
  - **Exception — branches you created in-session** When you have explicitly created a feature branch yourself (e.g. `git checkout -b feat/something`) in that session, commit and push to THAT branch freely without per-step permission. The entire branch is reviewed at PR-open, so the per-commit gate adds review noise without adding safety. The exception is scoped to branches Claude created in the current session; it does NOT extend to `main`, to other long-lived branches, or to branches the author created.
- **Back up the any database before any schema or migration change.** Before running any schema-changing SQL (ALTER TABLE, CREATE/DROP, hand-rolled column rewrites, anything that mutates table shape) against a database, back it up.
- **Project-specific rules belong in AGENTS.md, not in any agent's private memory store.** If you (Claude Code, Cursor, Codex, Aider, anything that has a "save this preference for next time" surface) catch yourself about to write down a rule that's actually about the codebase rather than about working with this particular user, write it here instead. Memory is fine for user-profile facts and tool-use preferences; project rules should be portable across agents.
- **Plain language over jargon.** Comments, docstrings, AGENTS.md, commit messages, PR descriptions, and user-facing docs should describe what the code does in words a non-specialist would understand. Avoid terminology imported from neighboring fields that only loosely applies — e.g., "projection" from relational algebra to describe "the subset of recap_document fields we keep in the local store", or "compaction" / "denormalization" / similar when a plain description works. When a domain term IS the right word (because the code really is implementing that concept, or the reader needs to look it up to understand a library), use it AND a brief in-place gloss the first time it appears. When a term is borrowed loosely, replace it with the literal description. The test is whether a contributor coming into the codebase from a different background would have to stop and search to understand what a term refers to here; when in doubt, prefer the plainer rewrite even if it's a few extra words.
- **Fix underlying bugs, never just patch the data.** A manual SQL update or shell command that corrects ONE row of bad state a database doesn't help other users running the same code, doesn't help future data hitting the same bug, and doesn't survive a fresh checkout. Every observed bug must result in a code change that prevents the bad state from recurring, even when an immediate manual patch is also applied to unblock the operator. The manual patch is the bridge; the code fix is the destination — both happen, never just the bridge.
- **Verify library signatures against the installed version, not memory.** Before calling an unfamiliar function from a third-party library, read the source of the version that is actually installed in the project (the file in `site-packages` or equivalent). Training data and prior conversations are not authoritative — the installed code is.
- **Read official documentation in full before implementing against an unfamiliar API.** Fetch the relevant pages and read them end-to-end, not just the headings. When the docs offer both a quick-reference and a detail page on the same topic, read the detail page — quick-references omit aliases, edge cases, and secondary functions you will need.
- **SDK research order: installed source, then vendor docs, then GitHub issues.** When figuring out how a vendor SDK behaves, the installed SDK's source is the source of truth, vendor documentation is second, GitHub issues are third (for known bugs and undocumented behavior). Third-party blogs, Stack Overflow answers, and AI-generated explainers are not primary evidence — at best they are pointers to one of the three primary sources.
- **Don't catch `Exception` broadly.** Catch only the specific exception types you have a recovery path for. A bare `except Exception:` (or `except:`) hides programming errors that should be loud, makes debugging harder, and disguises broken assumptions as transient failures. Let unexpected exceptions propagate.

## Testing

- **Name this project's slower or network-dependent suites here, and the code that requires them.** The review step in `CLAUDE.md` reads this list to decide what a round has to run, and the fresh-context review prompt carries the same conditions once a project fills it in. A suite that is not named here is one a round skips silently while still reporting a clean review.
- **A test named for an exclusive claim must prove both halves.** "Only", "never", and "exactly once" each assert a negative as well as a positive. If the shared test setup can't observe the negative half, build a fresh setup for it instead of substituting a nearby assertion that always passes.
- **Prove a regression test by running it against the unfixed code.** A test written alongside a bug fix must fail when the fix is reverted; otherwise it guards nothing.
- **When testing end-to-end, confirm you are running the edited code.** Running a package from outside the project directory can silently resolve an older installed copy; check the module's file path (or the paths in a traceback) before trusting the result — an old copy can convincingly reproduce the exact bug you are fixing.

## Review discipline

These rules were distilled from real multi-agent review cycles in which defects survived thorough author-side review — in later cycles, a fresh-context diff review as well. Each one names a pattern that self-review reliably misses. Grouped by theme, with the reviewer's prompt itself at the end.

### Review prose as prose

A review that only verifies functional correctness (tests pass, files import, types check) sails past exactly the defects a text-first reviewer catches.

- **Whole-file regenerated artifacts put every line in the diff — review them as text, too.** Re-exporting a dashboard definition or other generated document rewrites the entire file, so pre-existing user-facing strings are formally part of the change; a semantic before/after comparison deliberately looks through them. Add a text-level pass over titles, labels, and markdown.
- **Proofread the whole hunk and the *rendered* text, not just the `+`/`-` lines.** Typos one line away from an edit are in your context window and fair game, and wrap points interact with markers and punctuation (a comment marker landing before an issue number, a trailing hyphen, a code span split across lines) — reflow rather than argue the raw text is technically correct.
- **Clean inert config inside hunks the diff already rewrites** — stale entries cost nothing to remove and confuse every later reader; "minimize the diff" is the wrong tiebreaker there, and remains the right one for untouched files.
- **Docstrings and comments are prose surface too — beware dual-use terms.** Words that are both colloquial English and load-bearing technical terms near the code in question ("nested", "index", or "keyword" near a search-engine mapping) pattern-match as true for an author who holds both facts.
- **A plain-type docstring is wrong when `None` is a semantic state.** Documenting optional parameters with bare types is fine while `None` merely means "not provided"; when `None` is a meaningful third state (a sentinel selecting "inherit" or "auto"), document the union type and the sentinel's meaning, reading the entry as a naive caller who doesn't share your context.

### Nothing is pre-verified

Code that *feels* already-reviewed — or exempt from review — has zero review coverage. Five disguises:

- **Moved code.** A "pure move" is a claim about behavior preservation, not an exemption from review — read extractions cold, and be *more* suspicious when a hunk gains callers than when it changes logic.
- **Extracted helpers.** A helper promoted out of a call site inherits none of that site's implicit guarantees: it needs its own eager input validation and its own docstring↔behavior check, even when every current caller happens to be safe.
- **Fixes made during review.** Touching one direction of a paired protocol (`__getstate__`↔`__setstate__`, save↔load, encode↔decode) obligates re-deriving the inverse direction, including version-skew inputs (old data into new code) that no current fixture produces. The review isn't done when the fixes are written.
- **Rewritten code, for coverage.** Rewritten lines are new patch lines even when behavior is intentionally identical — error branches carried over from the old code still need tests now.
- **Mid-incident glue.** Firefighting is not an exemption: before writing new shell/infra code mid-incident, check the file for an existing helper that already does it, and give your own inline code the same scrutiny you'd give a subagent's.

### Check claims against what they range over

The defects that author-side reviews miss are rarely inside one artifact — they are relations between two individually-correct places.

- **When fixing one half of a contract, grep for the other half**: write↔read against the type contract, comment↔declaration, a docstring guarantee↔every statement in its scope, a UI string↔the docs naming it.
- **Count enumerations against the code-defined set they enumerate** — derive the set from the code and count both sides; a reader can't tell an intentional subset from an omission.
- **A quantified claim is an enumeration in disguise, and "pre-existing" triage stops applying when the diff extends its set.** A paragraph asserting something about "all the options above" becomes part of the diff the moment the diff adds options — re-derive the claim against the current diff; don't inherit an earlier pass's "pre-existing, out of scope" label.
- **Build verification fixtures containing what the sample corpus lacks** — optional fields, injected errors, over-the-cap sizes — because an absent field makes the wrong key and the right key behave identically.
- **Update tracking state only after the action it tracks has succeeded.** When code clears a counter, marks something done, or advances a cursor around an action that can fail (a file move, a write, a network call), do the update after the action succeeds — then walk each failure branch and ask what the state means if the action fails right there. Reviews reliably verify that cleanup *exists*; they miss *when* it runs.
- **If something can report failure two ways, handle both ways the same.** A function that signals failure by return value in one configuration and by raised exception in another must run the same cleanup and safety logic on both paths. Find every place that raises, not just every place that returns — and remember that what happens to a raised exception depends on every caller it can propagate through.

### Verify what CI enforces, not a plausible subset

- **Run CI's literal commands from the repo root** — read the workflow file. When repo-wide runs are noisy because of untracked local directories, fix the exclusion in config rather than narrowing the command — a narrowed command is a different check that happens to share a name.
- **Cover CI's gates, not just its commands.** Patch coverage corresponds to no replayable workflow command, so command-replay never asks "does a test execute every new line?" — compare coverage's missing-lines report against the diff before opening a PR.
- **An ad hoc check that matches nothing is broken, not green.** Build one-off verification scripts to fail loudly on zero matches — a filter aimed at the wrong path or key silently produces an empty, passing-looking result. Silence is not success.

### Treat outside values and pasted commands as code

- **A value from outside the checkout is untrusted the moment it reaches a line that runs it.** A tag name, a ref, a CI-supplied value, a file name a user or a forge hands in: each one is data, not code, until something in the diff proves otherwise. In a Makefile recipe, read it from the environment as `"$${VAR}"`, quotes included, rather than expanding it as `$(VAR)` into recipe text: make substitutes a `$(VAR)` into the line before the shell ever parses it, so the shell reads the value itself as code. The quotes are part of the remedy — unquoted, `$${VAR}` is still word-split and glob-expanded, so it stops reporting the value verbatim. In a CI workflow's `run:` step, let it reach the script through the environment, as a step-level `env:` mapping or a `GITHUB_*` default variable, and read it there as `"$VAR"`; a `${{ }}` expression is pasted into the script body before the shell sees the line, in exactly the way `$(VAR)` is. Give it a hostile-value test — a tag or file name carrying shell metacharacters, which the tooling must report verbatim and never execute — because this is the defect class where a passing test suite proves nothing.
- **A command someone will paste is code.** Every command in the README and the docs gets typed into a real shell, sometimes as root, on a machine that cannot afford to break, so review it the way you review the code it installs. It must be safe on failure: fail the download rather than piping a half-written file onward, chain steps with `&&` so a failed step stops the next, and leave nothing half-installed behind. Explain each flag that changes what happens on failure once, the first time it appears — those are the flags someone has to understand before running the command, and they are invisible in a rendered page that only shows the happy path.

### End with a fresh-context review, not a self re-read

The author's "cold re-read" is never cold — it confirms the model the author already holds, which is exactly the blindness a fresh reader doesn't share. Before the change is done, run a review pass whose reviewer has seen *only* the final diff — no plan, no conversation history, no memory of writing it (a subagent given only the checkout and the diff, or an external reviewer) — and end it asking "do these hunks agree with *each other*?", not "is each hunk correct?". Triage its findings like any external review: fix what's real, push back with cited reasoning on what isn't. Two limits to design around: a fresh-context reviewer running the same model still shares its priors (convention-compliance can pass for correctness), and some defect classes are only caught by deterministic gates, not by more reading. Running the last round on a different model from the earlier rounds is the cheapest answer to the first limit.

- **The prompt is bare.** The reviewer's prompt is the verbatim text in "The fresh-context review prompt", below, and the working agent adds no change-specific questions to it. A checklist written by the author of the change points the reviewer at what the author already thought of, which is the opposite of what a fresh reader is for. Anything specific the author wants checked goes in the PR description for the human reviewer, or is checked by the author directly.
- **Rounds repeat until a pass finds nothing beyond wording** (a rewrap, a sentence that says the same true thing less well); those are fixed without another round. A finding that changes what runs, what someone would copy and run, or what a sentence claims about the code or a source it cites gets another round. A sentence that is wrong is in that second group however small the edit, because a reader acts on these documents.
- **Scoping is per reviewer, not per round number.** Every round runs in fresh context, so what a reviewer remembers never decides how much of the diff it reads; the same reviewer means the same model reviewing this change again. A later round by the same reviewer is scoped: the header names the files changed since that reviewer's previous round, and that reviewer reads those whole and the rest of the diff only for agreement with them, resting on the author's word that an earlier round by that same reviewer covered the rest, so a fix that adds new surface is reviewed in full without the whole diff being re-read every time. A reviewer that no earlier round used gets no files-changed line and reads the whole diff, however many rounds another model has already run — otherwise the reviewer brought in precisely because it is a different model would be handed only the last wording fix.
- **A change that edits the instruction files edits its own review.** Every reviewer here reads them from the branch under review: the prompt's first instruction is to read `AGENTS.md` from the checkout, and the page linked below notes that Copilot reads its instructions from the head branch, not the base branch. So whenever a diff touches `AGENTS.md`, `CLAUDE.md`, or the other sources that page lists — `.github/copilot-instructions.md`, path-specific `.github/instructions/**/*.instructions.md`, `GEMINI.md`, `REVIEW.md`, and agent skills under `.github/skills` — a clean verdict on those files is worth less than a clean verdict on the rest of the change, and the author reads those edits directly rather than resting on rounds the edits themselves steered. Your own branches edit these files far more often than a fork's. A fork's pull request is the harder case, and the theme above reaches only half of it: its remedies are shell-level, and a fork's instruction file does not reach a line that runs it, it reaches the reviewer's prompt. The prompt also tells the reviewer to run this project's checks, which on a fork's branch means running the fork's build files and tests in the reviewer's environment. So on a pull request from a fork, read its edits to the instruction files before any round runs, and treat every command the prompt would have the reviewer run as the fork's code rather than yours.
- **A Copilot round with zero findings on the final commit, suppressed comments included, is part of "done."** [Copilot code review reads AGENTS.md and CLAUDE.md too](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review#customizing-copilots-reviews-with-custom-instructions), so it is not unprompted; it is required because it did not write the change and sees only the PR, not the session. It runs last because the round that counts is the one on the PR's final commit; a PR opened earlier, as a draft say, does not change that. Its findings take the same threshold as anyone's: a wording one is fixed without another round, a substantive one goes back through the loop and the Copilot round then re-runs on the new final commit. A finding it reports with no inline thread to reply on is answered in a PR-level comment that quotes it.

### The fresh-context review prompt

When you adopt this template, replace the bracketed list of checks in the prompt below with this project's own commands and delete the brackets. Keep its shape: the checks every round runs, then the slower suites named in the Testing section above, then the documentation checks, each of the conditional ones with the condition that triggers it. A flat list of commands drops those conditions and the slower suites stop running. Make that edit at adoption, and again whenever the Testing roster changes, so the roster and the copy of it in the prompt stay in step; never make it per use.

After that, hand the prompt to the reviewer exactly as written, with one substitution and one addition. The substitution is the branch base in the diff command, if it is not `origin/main`. The addition is a one-line header naming the repository path and branch and, from a reviewer's second round on, the files changed since that reviewer's previous round. Nothing else may differ from the text below.

Before handing it over, commit every fix: the diff command compares commits, so an uncommitted fix is missing from the diff the reviewer is handed while still showing up in the whole-file reads the prompt asks for, leaving the two in disagreement. `git status --porcelain` must print nothing. Then bring the base up to date and look at what the review will cover, with `git fetch origin && git log --oneline origin/main..HEAD`, naming the same base here as in the diff command. That list is the review's scope. A commit in it whose work is already upstream — squash-merged or rebased — means the branch wants rebasing first, or the round is spent re-reviewing merged work. A fetch never moves the local `main`, which is why both commands name `origin/main` and not `main`: a base read from a stale local branch is what puts already-merged commits into a review, and once the fetch has succeeded the remote-tracking base cannot be stale. If the fetch fails — offline, a proxy, expired credentials — the `&&` stops before the log and you get no list at all; fix the fetch rather than reviewing against a base that never moved.

```text
You are reviewing the diff `git diff origin/main...HEAD` of this
repository, and you have seen none of the work that produced it. Read
AGENTS.md from the checkout first, then read every changed file whole,
not just the diff hunks. This is a read-only review: run [the linter and
the test suite, the slower suites AGENTS.md's Testing section names when
the diff touches what they cover, and the documentation checks when the
diff touches prose]; and do not change any file.

Your job is to find what is wrong, not to confirm that the change works.
Security comes first, but it is not the whole job: treat every value that
comes from outside the repository as hostile until proven otherwise,
treat every claim in prose or a comment as unverified until you have
checked it against the code or the source it cites, and remember what
this code does in the hands of the people who run it. A review that finds
nothing still has to say what it looked for and could not find; it never
just says the diff is fine.

Ask whether the hunks agree with each other, not only whether each hunk
is correct on its own. If the header names files changed since a
previous review round, read those whole and the rest of the diff only
for agreement with them; an earlier round of yours has covered the rest.

Assume the diff contains at least one place where a value from outside
the repository reaches a line that runs it unescaped, at least one
command someone would copy and run that misbehaves on failure, and at
least one sentence in prose or a comment that the code or a source it
cites contradicts. Find them, or say plainly why you could not.

For each finding, give the file and line, what is wrong, why it matters
to the people who run this, and the concrete fix. Label it "substantive"
(it changes what runs, what someone would copy and run, or what a
sentence claims about the code or a source it cites; a sentence that is
wrong is substantive however small the fix) or "wording" (the text stays
true and only reads better), so the author can tell whether another round
is owed. Say explicitly what checks out clean, and list anything you
could not verify.

End with a verdict: mergeable as is, mergeable after the listed fixes, or
not mergeable, with the fixes in the order to apply them. Do not fix
anything yourself.
```

## Python Code Style

These standards appply to ALL project Python code **including tests**.

- Formatter/linter: **Ruff**
  - All code must be linted and formatted
- Type annotations use `TypedDict` for structured results
- Supports all currently supported Python versions
- Modern type annotations across the entire project
  - Always use the the latest version of pywright for static type checking
- Testing framework: **pytest**
- Every bit of code should have a test
- Build backend: **hatchling**
- Module-level loggers: `logger = logging.getLogger(__name__)` — one logger per module, named for the module
- Project-defined errors subclass `RuntimeError`, not bare `Exception`, so callers can catch project failures specifically without sweeping in unrelated bugs

## Markdown Style

- All markdown must pass VSCode's default markdownlint config
  - VScode projects must be configured with `"markdownlint.config": {"MD024": false}` to allow for proper changelog headings

## GitHub releases

- Releases are made by version tag not branch
- Version tags should be prefixed with `v`, unless prior tags are not
- Release titles must always exclude the `v` prefix
- For Python projects, wheels and srcbuilds should always be attached
  - Use existing build files **if** they match the release version

## Documentation

The project must be well documented. If existing documentation exists, hollow that convention.

For new projects, do **NOT** use a monolithic readme. Instead, use the readme to provide an overview of the project, and leave specific details in friendly, bite-sized markdown-formatted pages in a `docs` directory.
