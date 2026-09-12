---
name: close-out-retro
description: When wrapping up a significant piece of work, review the session for genuine learnings and route each one to the right place (Claude's memory vs. the project's own docs), check whether any non-blocking bugs/gaps discovered along the way still need to be filed in the project's issue tracker, and flag any signal that a new or improved custom skill/agent (Claude Code's own, global or project-scoped) would have helped.
when_to_use: The user wants to close out or wrap up a finished piece of work, asks for a retrospective, or asks something like "any learnings to save", "anything to reflect in memory/docs", or "any issues we haven't filed yet".
allowed-tools: Read, Edit, Write, Bash
---

# Close-out retro

A short, two-part closing pass for the end of a task. Not a full audit — just what genuinely surfaced during the work just done.

## 1. Route learnings: memory vs. project docs

Scan the session for things actually worth keeping — corrections the user gave, design decisions with non-obvious rationale, gotchas hit along the way, security/architecture principles that emerged. For each one, decide where it belongs:

- **Claude's memory** (the auto-memory system) — if it's about *how to work with this user or this project in future sessions*: a workflow preference, a tooling quirk, context that isn't derivable from the code itself. Follow the existing memory type rules (user/feedback/project/reference) already active in this environment.
- **The project's own docs** (`CLAUDE.md`, `docs/*.md`, README, etc., committed to the repo) — if it's about *the code or system itself*, and any contributor (human or a future AI session with no memory access) would benefit from knowing it. This includes process rules (e.g. deploy ordering), architecture gotchas, and security principles discovered the hard way.

Some learnings need both — e.g. a project-memory note that a design decision was made, plus the actual rule written into the relevant doc. But **don't let a memory entry become a permanent duplicate of something now written into the docs.** Once something is captured in `CLAUDE.md`/`docs/`, a standalone memory file that just restates it is waste — the memory system's own rules already say not to save what's documented there. If a prior memory entry becomes redundant after a doc update in this pass, trim or remove it rather than leaving both.

Before editing a project doc, read the relevant file first and place the addition precisely (don't duplicate an existing section). Present the proposed doc edits and memory writes to the user rather than assuming approval — doc edits touch the repo and may need committing as a separate explicit step; memory writes are usually fine to just do, per the standing memory rules already in effect. When in doubt about a specific doc edit's placement, ask.

## 2. Check for unfiled backlog issues

Recall anything discovered as a side effect of this session's work — a bug, dead code, a latent edge case — that was **not** already fixed as part of the main task. For each:

1. Check whether it's already tracked (e.g. `gh issue list --search "<keyword>"` or the project's equivalent tracker).
2. If it's genuinely new and non-blocking, propose filing it. Match the existing issues' style (title format, body structure) if a pattern is visible.
3. **Don't file something whose root cause or real-world impact you never actually confirmed.** If you only suspect it might be a bug (e.g. "this Playwright click failure might indicate a real UI bug, or might just be an automation artifact"), say that plainly and let the user decide whether it's worth investigating or filing speculatively — don't write it up as a confirmed defect.

## 3. Suggest new/improved custom skills or agents

Scope: Claude Code's own skill/agent system only — global (`~/.claude/skills/`, `~/.claude/agents/`) and project-scoped (`<repo>/.claude/skills/`, `<repo>/.claude/agents/`). Never touch or mention a project's separate agent-orchestration layer for other AI tools (e.g. this repo's `.github/agents/`, `.github/skills/`) — that has its own dedicated tooling and is out of scope here.

Only raise this if a real signal actually showed up in the session just finished — don't ask "should we create a skill?" as a rote checklist item, and don't write anything (not even "none found") when no signal appeared:

- A similar multi-step procedure was done manually two or more times in this session.
- The user said something like "this is tedious every time" or "next time just do X automatically."
- An applicable skill's description/trigger didn't match so it never got invoked, or no skill existed and a long generic manual procedure was used instead.

When a signal is present:

- Propose only — never create the skill/agent file in this pass. Give a short (1-2 line) proposal per candidate: name, purpose, a draft trigger description. Wait for the user to decide.
- Cover both brand-new candidates and improvements to an existing skill/agent whose description or behavior this session showed to be off (e.g. this very kind of retro).
- Recommend global vs. project scope based on whether the pattern depends on this project's specific knowledge/structure. When it's ambiguous, default to recommending project scope, not global — an overly-generic global skill is noise in unrelated projects later.
- Before proposing, do a lightweight duplicate check: list the four locations above and check names/descriptions for overlap. Don't read candidate files deeply — that's for the actual creation step, not this pass.
- Don't audit or suggest scope migrations for existing skills/agents ("this global one should really be project-scoped") — stick to what this session's work actually surfaced.

Keep this separate from part 1: a simple one-off behavioral preference (something Claude can just judge in the moment) belongs in memory only. A multi-step, multi-tool procedure worth automating belongs here only — if it also warrants a memory note, cross-reference this section in one line rather than duplicating the content.

## Notes

- This is a closing pass, not a fresh codebase audit — scope it to what this session's work actually touched or surfaced.
- Skip any part if the user says it's not needed (e.g. "docs will be handled elsewhere", "no need to check for issues").
