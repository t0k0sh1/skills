---
name: devils-advocate
description: Get an adversarial second opinion on a finished plan or design before human review.
license: MIT
metadata:
  tags: "Review, Planning, Quality, Workflow"
  category: "workflow"
---

# devils-advocate

Identify the plan from the text supplied with the skill invocation. If it
names a file, read it in full; otherwise use the latest plan in the
conversation. If no plan is available, ask which one to review.

In Claude Code, use the named `devils-advocate` agent when available.
When a subagent tool is available, delegate the review to an independent
agent. Supply the full plan, working directory, relevant paths, settled
constraints, and the full review instructions from `references/review.md` next to this skill. Resolve this path
relative to this SKILL.md file in the installed plugin, not the project
working directory. Do not supply your own opinion. Relay its ranked findings
and verdict without softening or resolving them. Do not edit the plan or code.

If delegation is unavailable, disclose that this is a same-agent review
and read and apply `references/review.md` yourself. Do not claim independence.

