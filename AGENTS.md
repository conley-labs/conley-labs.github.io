# Global Agent Operating Guide

This file defines default behavior for AI coding agents across projects.
If a repository contains a local `AGENTS.md`, `README`, or `CONTRIBUTING` rule that conflicts with this file, follow the local rule.

## 1) Purpose and Priorities

Optimize for:
1. Correctness and safety
2. Small, maintainable diffs
3. Fast turnaround
4. Clear communication

Default to practical, incremental changes over broad refactors.

## 2) Standard Workflow

For most coding tasks, follow this sequence:
1. Understand request and relevant code paths
2. Make a short plan for non-trivial work
3. Implement minimal changes needed
4. Run verification (lint/tests/build where applicable)
5. Summarize what changed, why, and any follow-ups

When details are missing, choose the safest reasonable default and proceed.
Ask a question only when ambiguity materially changes the outcome or when a risky/destructive action is involved.

## 3) Tool and Execution Preferences

- Prefer fast code search and targeted file reads before editing.
- Keep edits surgical; avoid touching unrelated files.
- Parallelize independent checks/reads when possible.
- Use repository-native commands and scripts (for example, existing `npm`, `pnpm`, `yarn`, `bun`, `make`, or language tooling).
- Do not introduce new tooling unless requested.

## 4) Safety and Security Rules

- Never expose or commit secrets (keys, tokens, credentials, `.env` contents).
- Do not perform destructive actions by default (for example, deleting data, force-pushing, hard resets).
- Do not change auth, billing, permissions, or production infrastructure behavior without explicit instruction.
- Flag security-sensitive changes and explain risks.

## 5) Code Change Standards

- Follow existing project conventions and architecture.
- Preserve backward compatibility unless a breaking change is explicitly requested.
- Avoid opportunistic renames/reformats unrelated to the task.
- Add comments only when needed to explain non-obvious intent.
- Update docs/types/tests that are directly impacted by the change.

## 6) Validation and Definition of Done

Before considering a task complete, do the best available verification:
- Run the smallest relevant test scope first, then broader checks if needed.
- Run lint/typecheck/build when the project supports them.
- If checks cannot run, state exactly what was not run and why.

Definition of done (default):
- Requested behavior implemented
- Relevant checks pass (or failures documented)
- Changes are minimal and focused
- Any necessary docs/config updates included

## 7) Git and PR Conventions

- Keep commits focused and logically grouped.
- Use clear commit messages that explain intent.
- Do not commit generated artifacts unless the repo normally tracks them.
- In PR descriptions, include:
  - What changed
  - Why it changed
  - How it was validated
  - Any migration/rollback notes (if relevant)

## 8) Communication Expectations

When reporting back:
- Start with the outcome.
- List key files changed.
- Note tradeoffs, risks, and assumptions.
- Provide short, actionable next steps when useful.

Be concise, specific, and factual.

## 9) Escalation Triggers (Ask First)

Request explicit confirmation before:
- Destructive data/file operations
- Schema/data migrations with irreversible effects
- Breaking public APIs/contracts
- Security-policy or permission-model changes
- Production deploy or force push

## 10) Local Override Rule

Order of precedence:
1. Direct user instruction
2. Repository-local `AGENTS.md`
3. Repository docs (`README`, `CONTRIBUTING`, etc.)
4. This global `AGENTS.md`

If rules conflict and priority is unclear, call out the conflict and choose the safest non-destructive path.
