# Clearcraft implementation plan

All tasks use `docs/specs/clearcraft.md`. Work on `feature/clearcraft`; no commit or public push until final user approval.

### 1. Skill instructions → @balin
- **Description:** Create the portable skill and its pre-send check.
- **Files:** `skills/clearcraft/SKILL.md`.
- **Acceptance:** Correct ID/frontmatter, final-summary-only scope, outcome first, educational 0–2 definitions, optional accurate analogy, facts preserved, sources included.
- **Dependencies:** Approved behavior and packaging.

### 2. Evaluation cases → @samwise
- **Description:** Write concrete inputs, expected properties and an evaluation rubric for S1–S12.
- **Files:** `skills/clearcraft/eval.md`.
- **Acceptance:** Cases are reproducible; contain no fabricated execution claims; distinguish inspection, model samples and discovery.
- **Dependencies:** Specification; independent of #1.

### 3. Detailed README → @balin
- **Description:** Write English documentation with Polish before/after examples and safe manual installation, activation, update and removal instructions.
- **Files:** `README.md`.
- **Acceptance:** Commands match layout; no destructive overwrite, no private paths; fixed activation markers; truthful limitations and sources; links to evaluation, license and spec.
- **Dependencies:** #1 and #2.

### 4. MIT license → @balin
- **Description:** Add the standard MIT license attributed to the public GitHub identity.
- **Files:** `LICENSE`.
- **Acceptance:** Standard MIT text, year 2026, copyright `bartoszjaniak`, no private email.
- **Dependencies:** None.

### 5. Git exclusions → @balin
- **Description:** Exclude common secret files and local/editor artifacts.
- **Files:** `.gitignore`.
- **Acceptance:** Minimal exclusions, no needed source files hidden.
- **Dependencies:** None.

### 6. Model evaluation → @samwise
- **Description:** Apply the actual skill to the evaluation inputs and record observed outputs and verdicts.
- **Files:** `docs/evaluation.md`.
- **Acceptance:** Actual sample count, evidence and limitations; no claim of application build or global discovery from inspection.
- **Dependencies:** #1, #2.

### 7. Global installation → @balin
- **Description:** Copy only the skill directory to the global OpenCode skills directory and create or append the uniquely marked activation instruction.
- **Files:** User-local `~/.config/opencode/skills/clearcraft/` and `~/.config/opencode/AGENTS.md` (never committed).
- **Acceptance:** No existing files overwritten; unrelated instructions preserved; no duplicate activation; installation matches canonical skill. Stop and report collisions.
- **Dependencies:** #1, #2, #3 and verification of their content.

### 8. Review and publish → lead
- **Description:** Inspect changes and public-data safety, run structural checks, obtain Frodo and Sauron read-only gates, then obtain final user approval. Commit and create an empty public remote, push `main` and `feature/clearcraft`, verify visibility and default branch.
- **Files:** Reviewed repository only; no global configuration.
- **Acceptance:** Clean staged scope, no private paths/secrets, gates addressed, approval obtained, verified public URL.
- **Dependencies:** #1–#7, generated HTML preview from Bilbo.
