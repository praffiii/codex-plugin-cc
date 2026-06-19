# Codex plugin for Claude Code

Use Codex from inside Claude Code for reviews, adversarial reviews, and delegated coding tasks.

This fork keeps the original plugin features, but changes background execution so Codex work stays attached to a visible Claude Code subagent instead of disappearing into a detached worker.

## What You Get

- `/codex:review` for normal read-only Codex review
- `/codex:adversarial-review` for steerable challenge review
- `/codex:rescue` for delegated bug fixing, implementation, and investigation
- `/codex:status`, `/codex:result`, and `/codex:cancel` for tracked Codex jobs
- `/codex:setup` for install/auth checks and review-gate configuration

## Fork Behavior

This fork preserves the original command set and runtime behavior, with one workflow-focused change:

- background review, adversarial review, and rescue runs stay attached to visible Claude Code subagents
- Claude Code still shows the Codex task while it is running
- `/codex:status`, `/codex:result`, and `/codex:cancel` still work
- no invisible detached Codex worker is used for rescue tasks
- no feature from the original plugin is intentionally removed

This is useful if your workflow depends on Claude as the orchestrator/reviewer and Codex as the visible executor or second reviewer.

## Requirements

- ChatGPT subscription, including Free, or an OpenAI API key
- Node.js 18.18 or later
- Codex CLI installed and logged in

## Install

Add this fork as a Claude Code marketplace:

```bash
/plugin marketplace add praffiii/codex-plugin-cc
```

Install the plugin:

```bash
/plugin install codex@openai-codex
```

Reload plugins:

```bash
/reload-plugins
```

Then run setup:

```bash
/codex:setup
```

`/codex:setup` will tell you whether Codex is ready. If Codex is missing and npm is available, it can offer to install Codex for you.

If Codex is not installed yet:

```bash
npm install -g @openai/codex
```

If Codex is installed but not logged in:

```bash
!codex login
```

After install, you should see:

- `/codex:review`
- `/codex:adversarial-review`
- `/codex:rescue`
- `/codex:status`
- `/codex:result`
- `/codex:cancel`
- `/codex:setup`
- `codex:codex-rescue` in `/agents`

This fork also includes visible review forwarding agents for background review runs.

## Usage

### `/codex:review`

Runs a normal read-only Codex review on your current work.

Use it when you want:

- a review of uncommitted changes
- a review of your branch compared to a base branch like `main`

Examples:

```bash
/codex:review
/codex:review --base main
/codex:review --background
/codex:review --wait
```

When run with `--background`, this fork keeps the review attached to a visible Claude Code subagent while preserving tracked job status.

### `/codex:adversarial-review`

Runs a steerable review that challenges the implementation approach, design choices, assumptions, and risk areas.

It uses the same review target selection as `/codex:review`, including `--base <ref>` for branch review.

Use it when you want Codex to pressure-test:

- architecture
- security
- race conditions
- reliability
- data loss
- rollback strategy
- whether the chosen approach is actually the right one

Examples:

```bash
/codex:adversarial-review
/codex:adversarial-review --base main challenge whether this was the right caching and retry design
/codex:adversarial-review --background look for race conditions and question the chosen approach
```

This command is read-only. It does not fix code.

### `/codex:rescue`

Delegates a task to Codex through the `codex:codex-rescue` subagent.

Use it when you want Codex to:

- investigate a bug
- implement a fix
- continue a previous Codex task
- take a cheaper/faster pass with a smaller model
- execute a task after Claude has planned or reviewed it

Examples:

```bash
/codex:rescue investigate why the tests started failing
/codex:rescue fix the failing test with the smallest safe patch
/codex:rescue --resume apply the top fix from the last run
/codex:rescue --fresh investigate this from scratch
/codex:rescue --model gpt-5.4-mini --effort medium investigate the flaky integration test
/codex:rescue --model spark fix the issue quickly
/codex:rescue --sandbox workspace-write fix with normal workspace boundaries
/codex:rescue --background investigate the regression
```

In this fork, `--background` backgrounds the Claude Code subagent, but Codex itself stays attached inside that visible subagent until completion.

Write-capable rescue tasks default to `danger-full-access` so delegated implementation behaves closer to a full-access Codex app session. Review-only requests stay read-only. Pass `--sandbox read-only`, `--sandbox workspace-write`, or `--sandbox danger-full-access` to override the task sandbox for one run.

**Notes:**

- if you do not pass `--model` or `--effort`, Codex chooses its own defaults.
- if you say `spark`, the plugin maps that to `gpt-5.3-codex-spark`
- follow-up rescue requests can continue the latest Codex task in the repo

### `/codex:status`

Shows running and recent Codex jobs for the current repository.

```bash
/codex:status
/codex:status task-abc123
```

Use it to check progress, inspect active jobs, or find the latest completed job.

### `/codex:result`

Shows the final stored Codex output for a finished job.

```bash
/codex:result
/codex:result task-abc123
```

When available, it also includes the Codex session ID so you can resume the run directly in Codex:

```bash
codex resume <session-id>
```

### `/codex:cancel`

Cancels an active Codex job.

```bash
/codex:cancel
/codex:cancel task-abc123
```

### `/codex:setup`

Checks whether Codex is installed and authenticated.

```bash
/codex:setup
```

You can also enable or disable the optional review gate:

```bash
/codex:setup --enable-review-gate
/codex:setup --disable-review-gate
```

When enabled, the review gate uses a Claude Code `Stop` hook to run a targeted Codex review before the session stops. If Codex finds issues, the stop is blocked so Claude can address them first.

## Typical Workflows

### Review before shipping

```bash
/codex:review --background
/codex:status
/codex:result
```

### Challenge a risky implementation

```bash
/codex:adversarial-review --background look for auth, race condition, and rollback risks
```

### Delegate a bug fix

```bash
/codex:rescue --background fix the failing integration test with the smallest safe patch
```

### Claude orchestrator, Codex executor

1. Brainstorm or plan with Claude Code
2. Ask Claude to delegate implementation to Codex
3. Codex runs visibly in a subagent
4. Claude reviews the result
5. User approves the next fix or merge step

## Codex Integration

This plugin uses your local Codex CLI and Codex app server.

That means it uses:

- your existing Codex login
- your existing Codex config
- the same local repository checkout
- your machine-local environment

Configuration is read from:

- `~/.codex/config.toml`
- `.codex/config.toml` in trusted projects

Example project config:

```toml
model = "gpt-5.4-mini"
model_reasoning_effort = "high"
```

## FAQ

### Is this the official OpenAI plugin?

No. This is a personal fork of `openai/codex-plugin-cc`.

The goal of this fork is to preserve the original plugin features while making background Codex work visible and attached inside Claude Code.

### Why fork it?

The original background behavior can detach Codex work into a background worker. In some Claude Code workflows, that makes Codex feel invisible or unreliable because the main UI no longer shows the running Codex task.

This fork keeps the task visible through Claude Code subagents.

### Are any commands removed?

No. The original user-facing command set is preserved.

### Does `/codex:status` still work?

Yes. Background jobs are still tracked by the plugin runtime.

### Does `/codex:cancel` still work?

Yes. Active tracked jobs can still be cancelled.

### Can I still resume work in Codex directly?

Yes. Use `/codex:result` or `/codex:status` to find the Codex session ID, then run:

```bash
codex resume <session-id>
```
