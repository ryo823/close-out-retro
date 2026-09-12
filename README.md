# close-out-retro

A [Claude Code](https://claude.com/claude-code) skill that runs a short closing pass at the end of a significant task: it routes genuine learnings to the right place (persistent memory vs. project docs), sweeps for bugs/gaps that surfaced but were never filed, and — only when a real signal appeared — proposes new or improved skills/agents.

日本語版は [README.ja.md](README.ja.md) を参照してください。

## Why

Claude Code sessions surface a lot of small, valuable signal that's easy to lose once the terminal scrolls away: a correction the user gave, a workaround for a subtle bug, a side-discovered issue nobody filed, a manual procedure repeated three times that should've been a skill. `close-out-retro` is a deliberately small, invocable-on-demand pass that catches that signal at the natural end-of-task boundary, instead of relying on it being remembered later.

It is not a full audit and not a background/automatic hook — you (or a "wrap this up" style trigger phrase) invoke it once a piece of work is actually done.

## What it does

1. **Routes learnings: memory vs. project docs.** For each thing worth keeping, it decides whether it belongs in Claude's cross-session memory (workflow/tooling context specific to *this user or project*) or in the project's own committed docs (`CLAUDE.md`, `docs/*.md`, README — anything a future contributor, human or AI, would benefit from). It explicitly avoids letting a memory entry become a stale duplicate of something a doc update just captured.
2. **Sweeps for unfiled backlog issues.** Anything discovered as a side effect of the session — a bug, dead code, a latent edge case — that wasn't already fixed gets checked against the project's tracker and proposed for filing, *if* it's genuinely new and confirmed. It's explicit about not filing speculative "this might be a bug" guesses as if they were confirmed defects.
3. **Proposes new/improved skills or agents — only on real signal.** If a multi-step procedure was done manually more than once, or the user said something like "this is tedious every time," or an applicable skill existed but didn't trigger, it proposes (never auto-creates) a candidate skill/agent, scoped to global vs. project based on how project-specific the pattern is.

Full behavior spec: [`SKILL.md`](SKILL.md).

## Install

**Global** (available in every project):

```bash
mkdir -p ~/.claude/skills/close-out-retro
curl -o ~/.claude/skills/close-out-retro/SKILL.md \
  https://raw.githubusercontent.com/ryo823/close-out-retro/main/SKILL.md
```

**Project-scoped** (this repo only):

```bash
mkdir -p .claude/skills/close-out-retro
curl -o .claude/skills/close-out-retro/SKILL.md \
  https://raw.githubusercontent.com/ryo823/close-out-retro/main/SKILL.md
```

Claude Code picks up skills from either location automatically — no build step, no dependencies.

## Usage

Invoke it with `/close-out-retro`, or just say something like "close this out," "any learnings to save?", or "anything we haven't filed yet?" at the end of a task — the `when_to_use` trigger in `SKILL.md` matches on intent, not just the exact slash command.

### Example (fictional project)

Say you just finished adding rate-limiting to a fictional `acme-widgets` API, and along the way: the user corrected you twice on where config belongs, you noticed an unrelated flaky test, and you manually repeated a 5-step "check quota, patch config, restart worker, verify, tag release" sequence three times.

```
> /close-out-retro

## 1. Learnings
- [memory] User prefers rate-limit config in `config/limits.yml`, not env vars —
  said this is because ops rotates limits without a redeploy. (workflow preference → memory)
- [docs → CLAUDE.md] Worker restart order matters: config patch must land before
  `worker restart`, not after — restarting first silently keeps the old limit for
  one cycle. Proposed addition to CLAUDE.md's "Rate limiting" section.

## 2. Unfiled backlog
- `test/quota_test.py::test_burst` failed once, unrelated to this change, looked
  like a timing flake on CI — not confirmed as a real bug. Flagging, not filing,
  unless you want it investigated.

## 3. Skill/agent signal
- The "check quota → patch config → restart worker → verify → tag release" sequence
  was done manually 3 times this session. Proposing a project-scoped skill,
  `quota-rollout`, scoped to acme-widgets since it depends on this repo's specific
  worker/release tooling. Want me to draft it?
```

(This is a constructed example for illustration — no real project's data was used.)

## Design notes / guardrails

- **Never claims a suspicion is a confirmed bug.** Part 2 explicitly separates "confirmed, worth filing" from "might be a flake, your call."
- **Never lets memory rot into a duplicate of a doc.** Once something lands in `CLAUDE.md`/`docs/`, the standalone memory note that only restated it should be trimmed.
- **Proposes skills, never creates them silently.** Part 3 only ever surfaces a candidate for you to approve — it never writes a new skill/agent file on its own.
- **Scoped to what this session actually touched** — not a fresh full-codebase audit every time it runs.

## Related work

This isn't the first attempt at giving an AI coding agent a memory of its own sessions — worth knowing before assuming this is unprecedented:

- [netresearch/retro-skill](https://github.com/netresearch/retro-skill) — a `/retro` command that routes session learnings to memory/project-rules/skill-PR destinations, with per-proposal approval. Close to part 1 of this skill, and has a thin version of part 3 (a "new-skill" destination bucket). Doesn't sweep an issue tracker (no part 2).
- [melodykoh/learning-loop-skill](https://github.com/melodykoh/learning-loop-skill) — a scan/wrap-up pair that routes learnings to `CLAUDE.md`/`MEMORY.md`/a "Judgment Ledger." Also close to part 1; no issue-tracker sweep, and skill/agent proposals aren't a first-class part of it.
- [「Claude Codeに『ふりかえり』を教えてみたら、セッションをまたいで成長するAIパートナーに近づいた話」](https://qiita.com/echolimitless/items/949070036dba433c69a9) (Qiita, ja) — a three-tier memory architecture extracting patterns from session transcripts and escalating reinforced ones into `CLAUDE.md` or new skills/hooks. Overlaps significantly with part 1 and a thresholded version of part 3.

As far as I could find, none of the above combine all three parts — in particular, sweeping the project's own issue tracker for things that surfaced but were never filed (part 2) appears to be genuinely uncommon. `close-out-retro` is best read as a synthesis: it doesn't claim the memory-routing idea (part 1) is new, but bundles it with an issue sweep and a disciplined, propose-only skill-gap check.

## License

[MIT](LICENSE)
