# close-out-retro

A Claude Code skill. Run it once at the end of a task. It does three things:

1. Routes learnings from the session to either Claude's memory or the project's own docs (`CLAUDE.md`, etc.)
2. Checks bugs/gaps found along the way but never fixed against the issue tracker, and proposes filing the confirmed ones
3. Proposes new/improved skills or agents, but only when there's an actual signal for it

Japanese version: [README.ja.md](README.ja.md)

## Background

Sessions tend to surface small useful things — a correction from the user, a workaround for a bug, an unfixed issue found along the way, a procedure repeated by hand a few times — that get lost once the session ends. close-out-retro is meant to catch that at the natural end of a task.

It's not a full audit and not an automatic hook. You call it when a piece of work is done.

## What it does

### 1. Route learnings

For each thing worth keeping: if it's about how to work with this user/project, it goes to memory. If it's about the code or system itself, it goes into the project's docs (`CLAUDE.md`, `docs/`, etc.).

It also avoids leaving a stale memory entry once the same thing is written into a doc.

### 2. Sweep for unfiled issues

Checks anything found as a side effect of the session — a bug, dead code, an edge case — against the project's tracker, and proposes filing it only if it's confirmed and new.

It won't write up a guess as a confirmed bug. If it's unsure ("this might be a real bug, might be a flaky test"), it says so and leaves the decision to you.

### 3. Propose skills/agents, only on real signal

Only proposes a candidate (never creates one) when:

- the same multi-step procedure was done by hand 2+ times
- the user said something like "this is tedious every time"
- an existing skill should have matched but didn't fire

It doesn't ask "should we make a skill?" every time.

Full spec: [SKILL.md](SKILL.md).

## Install

Global:

```bash
mkdir -p ~/.claude/skills/close-out-retro
curl -o ~/.claude/skills/close-out-retro/SKILL.md \
  https://raw.githubusercontent.com/ryo823/close-out-retro/main/SKILL.md
```

Project-scoped:

```bash
mkdir -p .claude/skills/close-out-retro
curl -o .claude/skills/close-out-retro/SKILL.md \
  https://raw.githubusercontent.com/ryo823/close-out-retro/main/SKILL.md
```

## Usage

Call `/close-out-retro`, or say something like "let's close this out" or "anything worth saving?" at the end of a task.

### Example (fictional project)

A fictional `acme-widgets` API, after adding rate limiting.

```
> /close-out-retro

## 1. Learnings
- [memory] User prefers rate-limit config in `config/limits.yml`, not env vars —
  ops rotates limits without a redeploy.
- [docs -> CLAUDE.md] Worker restart order matters: patch config before
  `worker restart`, not after, or the old limit survives one cycle.

## 2. Unfiled backlog
- `test/quota_test.py::test_burst` failed once, unrelated to this change.
  Looks like a CI timing flake, not confirmed. Not filing, your call.

## 3. Skill/agent signal
- "check quota -> patch config -> restart worker -> verify -> tag release" was
  done by hand 3 times this session. Proposing a project-scoped skill,
  `quota-rollout`, since it depends on this repo's tooling.
```

Fictional example, no real project data used.

## Related work

A few things do something similar:

- [netresearch/retro-skill](https://github.com/netresearch/retro-skill): a `/retro` command routing learnings to memory/project-rules/skill-PR destinations. Close to part 1. No issue-tracker sweep.
- [melodykoh/learning-loop-skill](https://github.com/melodykoh/learning-loop-skill): scan/wrap-up pair routing to `CLAUDE.md`/`MEMORY.md`/a "Judgment Ledger". Also close to part 1.
- [A Qiita post by echolimitless](https://qiita.com/echolimitless/items/949070036dba433c69a9) (Japanese): a three-tier memory architecture extracting patterns from session transcripts and promoting reinforced ones into `CLAUDE.md` or new skills. Overlaps heavily with part 1.

Part 1 isn't new. What I couldn't find elsewhere was part 2 — sweeping the issue tracker for things that surfaced but were never filed. close-out-retro combines that with a propose-only version of part 3.

## License

[MIT](LICENSE)
