---
name: setup-sskills
description: Configure which models sskills uses per role and at what reasoning budget. Detects your available models and writes a user-level rules file that overrides the skill defaults. Use for /setup-sskills, "configure sskills models", "sskills budget", or changing sskills's model choices.
---

# Setup sskills

Write `~/.claude/rules/sskills-models.md`, a user-level Claude Code rules file (loaded every session) that sets sskills's model per role.

## Steps

### 1. Detect available models

Read the `Agent` tool's schema in this session. Its `model` enum lists the model aliases you can pass to a subagent (currently `sonnet`, `opus`, `haiku`, `fable`), and its `effort` enum lists the reasoning levels (`low`, `medium`, `high`, `xhigh`, `max`). That is the dependable source. A role entry is `<model>-<effort>`, such as `opus-xhigh`, and means Agent `model: opus`, `effort: xhigh`. The detected set is every `<model>-<effort>` pair built from the enums. If the enum is not visible, ask the user which model aliases they have access to. Never write a model alias you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected entries.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.claude/rules/sskills-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer AskUserQuestion over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`, `auto`). `unlimited`, `large`, `medium`, and `small` set the effort token of every real entry, panel entries included, to `max`, `xhigh`, `high`, or `medium`. The effort token is the last token, on the ladder `max` > `xhigh` > `high` > `medium` > `low`, and it maps 1:1 to the Agent `effort` parameter. If the result is not a detected entry, use the same model alias at the highest detected effort at or below the target, else mark the role as needing a choice. `inherit-parent` and `auto` do not change. So `unlimited` turns `opus-xhigh` into `opus-max` and `sonnet-xhigh` into `sonnet-max`. `large` keeps both defaults. `small` turns them into `opus-medium` and `sonnet-medium`.

**(c) Show the roles and confirm.** Show every role with its model, marking any real entry not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected model aliases (each at a chosen effort) plus `inherit-parent` and `auto` (both mean: this role runs on the parent session's model and effort) as the options. Prefer AskUserQuestion over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model family differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real entry written must be in the detected set. `inherit-parent` and `auto` always pass. If a chosen real entry is not available, stop and ask again.

### 5. Write the rule

Write `~/.claude/rules/sskills-models.md` as plain markdown with no frontmatter, a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels santislop-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# sskills model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent session's model (omit Agent `model` and `effort`). Alias entries in a panel list still count toward its fan-out.
# budget: large (xhigh)
feature, refactoring: sonnet-xhigh
bug-fix: sonnet-xhigh
perf-issue: sonnet-xhigh
hillclimb: sonnet-xhigh
judgment and prose: opus-xhigh
hardest tasks: opus-xhigh
how explorer: sonnet-xhigh
how explainer: opus-xhigh
why investigators: sonnet-xhigh
why synthesizer: opus-xhigh
reflect tooling: sonnet-xhigh
reflect judgment, divergent, synthesizer: opus-xhigh
arena runners: opus-xhigh, sonnet-xhigh
arena cross-judge pool: opus-xhigh, sonnet-xhigh
swarm workers: sonnet-xhigh
architect runners: opus-xhigh, sonnet-xhigh
interrogate reviewers: opus-xhigh, sonnet-xhigh
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever sskills is installed: workspace, user, or plugin). On no, move on without pushing.
