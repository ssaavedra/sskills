# Set up sskills

In this page you install the plugin, pick which models sskills uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

sskills is a fork of pstack, the Claude Code port of the Cursor plugin by Lauren Tan ([github.com/cursor/plugins](https://github.com/cursor/plugins)), maintained by Santiago Saavedra. In Claude Code, add the marketplace that holds it and install the plugin:

```text
/plugin marketplace add <path-or-repo>
/plugin install sskills@<marketplace>
```

Claude Code confirms the plugin is installed. Plugin skills are namespaced, so `/sskills:santislop-mode` always works, and the bare `/santislop-mode` usually works when it's unambiguous. This guide uses the bare names.

## Pick your models

Run:

```text
/setup-sskills
```

[`/setup-sskills`](../../skills/setup-sskills/SKILL.md) detects the models you have access to, asks for a reasoning budget, shows you each role (code delegates, judgment, the review panels), and asks what you want. Answer the questions. It writes `~/.claude/rules/sskills-models.md`, a small rules file Claude Code loads every session and every sskills skill reads.

The defaults run at `xhigh` reasoning, the same as the `large` budget. `unlimited` lifts each model to its highest tier, up to `max`. Opus goes to `max`. Sonnet tops out at `xhigh`, so it stays there. `medium` and `small` lower the reasoning and spend fewer tokens.

You only override what you care about. A role with no line in the file keeps the skill's default. To restore a default, delete that role's line. A rerun of `/setup-sskills` keeps any role whose model differs from the default. When a default changes, a rules file written before the change still pins the old default, so delete those role lines, or delete the file, then run `/setup-sskills` again.

You might be wondering what happens if you use the default model. Set a role to `inherit-parent` or `auto` and sskills omits the subagent `model` and `effort` fields, so the subagent inherits your parent chat model. Both values mean the same thing, and neither is a model slug. For a panel role the value is a list, and one subagent runs per entry, so the list length sets the panel size. Setup also configures `swarm workers`, the default model for every `/swarm` worker unless a race names a model for each arm.

## Accept the verification offer, or don't

At the end of setup, `/setup-sskills` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.claude/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers it in depth.

If you're new to sskills, say yes. An agent that can check its own work keeps going until the check passes. An agent that can't hands every result back to you to check by hand. Of everything in this guide, the verification skill pays off the most.

After setup, start a new chat. The model rule applies to new sessions.

## Keep the cost in check

sskills spends extra tokens on subagents and review panels. That's the price of the rigor. To spend fewer:

- Rerun `/setup-sskills` and pick a smaller reasoning budget or cheaper models. A strong model in the main chat with cheaper, faster models in the code roles is a good split.
- Set a role to `auto` or `inherit-parent` so it runs on the chat's own model.
- Shorten a panel list. Each entry runs one subagent.
- Save `/santislop-mode` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/santislop-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/santislop-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. Claude Code has no modes, so invoke `/santislop-mode` at the start of each task. Say "new task" when the subject changes so it matches a fresh playbook.

Next: [Route work through `/santislop-mode`](./02-santislop-mode.md).
