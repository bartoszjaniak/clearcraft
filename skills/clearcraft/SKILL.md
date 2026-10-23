---
name: clearcraft
description: "Use when writing the final task summary or closing message after finishing work, when the summary is hard to follow or leads with implementation details instead of outcomes. Rewrites only the assistant's own closing summary: outcome first, details separate, up to two useful new terms explained. Does not edit files, documents, code or quotations, and does not apply to progress messages, internal subagent reports, ordinary questions or requested artifacts."
---

# Clearcraft

Rewrite your own closing summary so a reader sees the outcome first and keeps every important fact. You edit what you are about to send — not the user's writing, not the deliverable.

## When it applies

- The final task summary or closing message you send after finishing work.
- Load this skill when preparing that summary, unless it is already in context.
- Not for progress messages, internal subagent reports, ordinary questions, or requested artifacts such as documents, code and quotations.
- Respect an explicit opt-out in the conversation or project. This skill is a default only; it never overrides higher-priority instructions.

## Workflow

1. Gather the real results: what changed, what was verified, what was not, and any risks.
2. Lead with the user-visible outcome. If the task is unfinished, lead with the next action and the current state instead.
3. Split needed technical detail into a short separate paragraph or list. No mandatory headings and no empty sections.
4. Keep exact names of standards and technologies; never rename, translate or abbreviate them. Keep required numbers, commands, paths, links, failed checks, warnings and uncertainty. Never invent outcomes, causes, benefits or verification. Distinguish made from verified.
5. Explain zero to two new terms that help the reader understand the change, its verification or a decision. Put each definition beside its first use. Explain none when none are needed. Simplify other unnecessary jargon without deleting required names or facts. Do not redefine a term already defined in the visible conversation. Do not claim persistent memory and do not store vocabulary on disk; only the visible conversation is available.
6. Add at most one short everyday analogy, only if it helps. Give the literal definition first and make it work without the analogy; then state the relevant similarity and any important limit. Drop an analogy that needs long caveats or distorts the idea.
7. Write in the conversation's language. Keep a respectful adult tone and short, readable sentences.
8. Send the edited summary silently. Do not ask for a draft and do not describe the editing itself.

Explicit user requests override the defaults: deeper explanations, comparisons or examples take precedence over brevity. Higher-priority instructions win — including an explicit opt-out, or a ban on comparisons or analogies. When opt-out is set, do not apply this editing style; keep independent obligations such as accurate reporting.

## Pre-send check

- Outcome or next action is first.
- Every important fact, warning, failure and unrun test is still present; nothing new is claimed.
- Commands, paths, links and exact names of standards and technologies are unchanged; nothing was renamed or abbreviated.
- Explanations are self-contained: 0–2 new terms, each defined beside its first use, no redefinition of a term already in the visible conversation, and any definition works without its analogy (at most one, honest).
- No draft request, no editing report, no repeated closing recap.

Evaluation cases and rubric live in `eval.md`; read that file only during evaluation, not in everyday use.

## Sources

- https://www.gov.pl/web/cyfryzacja/prosty-jezyk — plain Polish: outcome first, explain technical terms.
- https://www.csun.edu/~vceed002/books/sourcebook/chapters/10-analogies/teaching-analogies.html — map relevant similarities and state analogy limits.
- https://www.w3.org/WAI/WCAG2/supplemental/patterns/o3p12-implicit-explained/ — explain nonliteral meanings; supplemental guidance, not a standalone WCAG conformance requirement.
- https://opencode.ai/v2/docs/skills — native skill structure and discovery.
- https://opencode.ai/v2/docs/instructions — global activation guidance.
