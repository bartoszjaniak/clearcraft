# Clearcraft

## 1. Goal and rationale
Clearcraft is a portable Agent Skills skill with installation guides for OpenCode, Claude Code and Codex CLI. It makes final task summaries easy to understand without removing important facts. It teaches one or two relevant technical terms when useful. It follows the language of the conversation, with English documentation and Polish examples.

## 2. Problem
Technical summaries often emphasize implementation details instead of outcomes. Over-shortening can hide failed checks and risks. Clearcraft edits the assistant's own closing summary, not the user's writing or the requested deliverable.

## 3. Adopted solution
- Lead with the user-visible outcome, or the necessary next action together with the current state. Never imply completion without evidence.
- Put needed technical details in a separate short paragraph or list. No mandatory headings or empty sections.
- Explain one or two new, relevant technical terms beside their first use. Keep accurate names of standards and technologies. Explain zero terms when none are needed. Simplify other jargon without deleting required names or facts.
- Select terms that help the reader understand the change, its verification, or a decision. Do not insert jargon just to teach it.
- Do not repeat definitions visible earlier in the same conversation unless asked. Lost context is a real limitation; do not claim persistent memory or store vocabulary on disk.
- At most one brief everyday analogy, only when helpful. Give a literal definition first, state the relevant similarity and any important limit. Omit an analogy that needs lengthy caveats or distorts the concept. The definition must stand alone.
- Preserve required facts, numbers, failed checks, warnings, uncertainty, deployment state, commands, paths and links. Never invent outcomes, causes, benefits or verification. Distinguish a change being made from it being verified.
- Edit silently before sending. Do not ask for a draft or add a report about the editing itself. No repeated closing recap.

## 4. Alternatives considered
Reusing a draft-editing skill was rejected: this skill works on the assistant's own closing response. A plugin, automatic output filter, installer and extra model calls were rejected as unnecessary. Use a native skill plus a short global instruction.

## 5. Architectural patterns
Separate activation, reusable editing rules and evaluation cases. Markdown-only; no dependencies, background process, persistent state or network calls during ordinary use. Follow the Agent Skills open specification for required `name` and `description` frontmatter, naming and length constraints, matching directory name and relative supporting-file references. Host-specific extensions are not required and must not be presented as portable requirements.

## 6. Architecture
Public repository: `https://github.com/bartoszjaniak/clearcraft`, MIT license.

Files:
- `skills/clearcraft/SKILL.md`: portable skill with ID/name `clearcraft`, trigger, workflow, pre-send check and sources.
- `skills/clearcraft/eval.md`: concrete evaluation inputs and expected properties, read only during evaluation.
- `README.md`: detailed English documentation with Polish before/after examples; purpose, boundaries, separate installation/activation guides for OpenCode, Claude Code and Codex CLI, deactivation, update/removal, compatibility, QA and sources.
- `LICENSE`: standard MIT, copyright attributed to `bartoszjaniak`, no private contact information.
- `docs/specs/clearcraft.md` and `.html`: public specification and standalone preview.
- `docs/plan.md`: implementation plan.
- `docs/evaluation.md`: observed model samples, structural and command checks, with host-specific limits of evidence.
- `.gitignore`: exclude secrets, local artifacts and editor files.

Install a copy of `skills/clearcraft/` in `~/.config/opencode/skills/clearcraft/`. Use copying rather than relying on unverified symlink discovery. Stop if the destination already exists; never overwrite unrelated work. Append a small activation section to global `~/.config/opencode/AGENTS.md`, preserving existing contents; do not modify OpenCode JSON or existing skills. Activation asks the main assistant to load `clearcraft` before a final task summary, unless already loaded. It does not apply to internal subagent reports, progress messages, ordinary questions, or requested artifacts such as documents and code. Disable this editing behavior with an explicit conversation or project instruction; independent reporting and safety requirements remain. Remove only the added activation section to disable globally.

Additional host guides (documentation only; do not change the user's Claude/Codex configuration):

| Host | Personal skill directory | Project skill directory | Explicit invocation | Personal instructions |
| --- | --- | --- | --- | --- |
| OpenCode | `~/.config/opencode/skills/clearcraft/` | `.opencode/skills/clearcraft/` | `@clearcraft` | `~/.config/opencode/AGENTS.md` |
| Claude Code | `~/.claude/skills/clearcraft/` | `.claude/skills/clearcraft/` | `/clearcraft` | `~/.claude/CLAUDE.md` |
| Codex CLI | `~/.agents/skills/clearcraft/` | `.agents/skills/clearcraft/` | `$clearcraft` or `/skills` selector | `${CODEX_HOME:-$HOME/.codex}/AGENTS.md` |

For Codex, a nonempty `AGENTS.override.md` in the effective Codex home takes precedence over `AGENTS.md`; inspect existing instructions and modify the active file manually, never remove the override. During deactivation/removal, inspect both possible files and remove only Clearcraft's marked section wherever it was actually added; a later override change must not leave a stale activation block. The skills documentation lists the personal skill root under `$HOME/.agents/skills`; the instructions documentation scopes `CODEX_HOME` to the Codex home. Do not assume changing `CODEX_HOME` moves that documented skill root; this interpretation is source-grounded, not runtime-tested here. For Claude, use its documented personal `CLAUDE.md`, not OpenCode's global `AGENTS.md`. Include a host-neutral activation block with the same markers and scope, no dependency on OpenCode's tool name. Explain project-scoped alternatives and keep personal installation as the documented default.

Use one selected host's destination per install/update/remove operation. Backups must live outside every skill discovery root, preferably `~/.local/state/clearcraft-backups/`, never inside `.opencode/skills`, `.claude/skills`, `.agents/skills` or their personal equivalents. Existing installation commands must continue working for OpenCode. Do not make three independent rule copies in the repository or add plugins, package manifests, CI, dependencies or an installer. Copying is our chosen installation policy, not a format requirement: Claude Code and Codex document symlink support; OpenCode symlink discovery was not checked. Warn that OpenCode also discovers `~/.claude/skills` and `~/.agents/skills` as compatibility locations. With duplicate IDs, native global `~/.config/opencode/skills` takes precedence over those compatibility sources; project `.opencode/skills` takes precedence over global native skills, and explicit configured skill sources can override discovery. Keep copies consistent and inspect the actual loaded path rather than assuming universal precedence across hosts.

Activation uses unique markers `<!-- clearcraft:start -->` and `<!-- clearcraft:end -->`; do not append a second section if these already exist. For updates, first inspect and back up the existing installed skill outside the repository, then replace only that skill directory with a fresh copy. Do not overwrite local modifications without approval. Keep activation unchanged on update. Never remove the whole global instructions file during uninstall.

Develop on `feature/clearcraft`. Review all public files and staged changes before committing or pushing. Use repository-local GitHub noreply commit identity to avoid disclosing a private email; do not alter global Git settings. Publish only this repository, never global configuration or private local session files. Final user approval is required before commit and public push. Create an empty remote without an auto-generated README or license; publish the approved initial commit on `main`, and retain/push the reviewed feature branch. Both branches may point to the same approved initial commit; no bootstrap commit before user approval is needed.

## 7. Interface
Manual invocation depends on the host: OpenCode `@clearcraft`, Claude Code `/clearcraft`, Codex CLI `$clearcraft` or the `/skills` selector. Automatic use is instruction-driven, not a guaranteed runtime hook. Input is the actual task results and visible conversation; output is only the edited final summary. No user-supplied draft is needed. Applying the skill does not edit files, code, documents or quotations.

Example (fictional scenario, not a verification claim):
> Logowanie ponownie działa po wygaśnięciu sesji. Dodałem **test regresyjny** — automatyczne sprawdzenie, które ma wykryć powrót tego samego błędu.
>
> Sprawdzenie: test przeszedł. Nie sprawdzałem działania na telefonie.

## 8. Data flow
1. Finish the task and collect real results.
2. Load the skill when preparing the closing summary, if not already in context.
3. Lead with the outcome, separate details, and explain up to two useful new terms.
4. Check against evidence: preserve important facts and limitations; do not add claims.
5. Send the edited summary without describing the editing process.

## 9. Non-functional requirements
Use a respectful adult tone, common words, short readable sentences and small information groups. No rigid word count, infantilization or decorative metaphors. Explicit requests for deeper explanation take precedence over default brevity. Respect higher-priority instructions: if comparisons are forbidden, use a literal definition only. Remain compatible with ADHD-style output and concise engineering summaries: preserve required next steps, unperformed checks and risks, without inventing risks. Do not import draft-editing workflows or modify other skills. Explain unavailable skill loading honestly. Start a fresh session to confirm discovery if the current skill list is stale; do not promise behavior in every model or session.

## 10. Testing strategy and cases
There is no application build or unit-test suite. Report structural inspection, model samples and actual OpenCode discovery separately.

For multi-host documentation, validate the shared skill against Agent Skills constraints (required frontmatter, name 1–64 characters, use the interoperable ASCII kebab-case subset for this repository, no consecutive hyphens, directory match, description 1–1024 characters, supporting files resolvable). Parse the current YAML with available PyYAML and assert these constraints without adding a dependency to the skill; `skills-ref validate` is an optional official validator, not an executed check unless actually run. Extract every fenced Bash block and run `bash -n` on each, reporting syntax separately from execution. Run selected-host install/collision/missing-source/symlink scenarios in disposable homes for all three destination layouts. Check that update/removal preserve nonselected host directories and unrelated instructions; validate README's shell selection of Codex instruction paths with `CODEX_HOME` set and unset and the nonempty override case. This verifies shell/file handling, not actual runtime precedence or discovery. Distinguish source-verified format/location claims, shell-only checks and actual host runtime loading. Do not claim Claude/Codex runtime validation or local installation if not performed.

| ID | Input scenario | Required behavior |
| --- | --- | --- |
| S1 | Small change without jargon | Outcome first, needed details separate, zero invented lessons. |
| S2 | Four or more technical terms | Explain one or two relevant terms; retain required facts and exact names. |
| S3 | A term defined in visible conversation | Do not redefine unless asked. |
| S4 | A useful everyday analogy | Literal definition works alone; analogy states the relevant similarity. |
| S5 | A misleading analogy | Correct or remove it; keep the accurate definition. |
| S6 | Failed check, risk and partial completion | Preserve all limitations; do not claim full success. |
| S7 | Progress question or requested document | Do not impose closing-summary formatting on it. |
| S8 | Unrun tests, an exact command and path | State that tests were not run; preserve command and path. |
| S9 | Explicit opt-out | Do not impose this editing style; retain independent obligations. |
| S10 | Task results without a draft | Produce the summary, not a request for a draft or an editing report. |
| S11 | Code, documentation or quoted text | Edit only the assistant's closing commentary, not files or quotations. |
| S12 | Skill unavailable | Do not pretend it loaded; state the limitation. |

Block on invented verification, lost warnings or required facts, changed commands/paths, misleading definitions, analogy-only explanations, asking for a draft, scope leakage, changes to unrelated settings, secrets or private paths in the public repository. Structural checks include valid frontmatter, unique path-derived ID `clearcraft`, matching metadata name and activation markers. Inspect generated HTML as well as staged content for private data. Evaluation must specify inputs, number of actual samples, observed outputs and pass/fail rationale. A sample is not proof of reliability. Do not claim global discovery from Markdown inspection. README installation, update and removal steps must match the actual layout and preserve unrelated configuration. No dependency or CI framework is needed.

## 11. User decisions
1. Apply only to final task summaries, globally by default in OpenCode.
2. Outcome first, technical details separately; retain and explain important technical terminology for learning.
3. Explain one or two new concepts, avoid repeating explanations in the same conversation; use analogies only if helpful.
4. Preserve errors, risks and unperformed checks. Do not edit code, documents or other messages as part of applying the skill.
5. Publish a new public repository with a detailed README. Approved name/account: `bartoszjaniak/clearcraft`; English README with Polish examples; MIT license. Keep the global installation requirement.
6. After reviewing the initial repository, the user requested installation guides for different agents and selected OpenCode, Claude Code and Codex CLI. This expands documentation and format verification; it does not authorize local installation or configuration changes for the two additional tools.

## 12. Open questions and decisions
No open product questions. Model memory is limited to available context; internal agent reports are not user-facing summaries. These are implementation boundaries, not promises of persistent memory or changes to the agreed scope.

Sources:
- https://www.gov.pl/web/cyfryzacja/prosty-jezyk — plain Polish, outcome first, explain technical terms.
- https://www.csun.edu/~vceed002/books/sourcebook/chapters/10-analogies/teaching-analogies.html — map relevant similarities and indicate analogy limits.
- https://www.w3.org/WAI/WCAG2/supplemental/patterns/o3p12-implicit-explained/ — explain nonliteral meanings; supplemental guidance, not a standalone WCAG conformance requirement.
- https://opencode.ai/v2/docs/skills — native skill structure and discovery.
- https://opencode.ai/v2/docs/instructions — global activation guidance.
- https://agentskills.io/specification — portable skill format and constraints.
- https://code.claude.com/docs/en/skills — Claude Code skill discovery and `/skill-name` invocation.
- https://code.claude.com/docs/en/memory — personal `~/.claude/CLAUDE.md` guidance.
- https://developers.openai.com/codex/build-skills — Codex personal/project skill locations and explicit invocation.
- https://developers.openai.com/codex/agent-configuration/agents-md — effective Codex home and instruction override precedence.

## Multi-host documentation revision
The user selected the three hosts after the initial final review, before authorizing the final follow-up commit/push. Keep the previously reviewed behavior unchanged; add source-grounded installation guides, a compatibility/evidence matrix and checks of the portable format. Infrastructure consultation identified override precedence, backup discovery, duplicate IDs, and per-host path separation as the important risks. This revision awaits its targeted review before publication.

## Review round: 1
Supersedes the earlier local `plain-language-summary` proposal. Core behavior previously reviewed by UX and QA. Updated packaging reviewed by UX, QA and infrastructure: UX has no blockers; QA update/activation-marker checks incorporated; infrastructure empty-remote, public-history inspection and noreply guidance incorporated. The suggestion to create a bootstrap commit before review was rejected: an unborn feature branch can receive the approved initial commit and `main` can then point to it. The current revision uses a copied installation to avoid relying on unverified symlink support. Public packaging reflects the user's explicit approval of repository name, account, visibility, license and documentation language.
