# Clearcraft

A portable [Agent Skills](https://agentskills.io/specification) skill that rewrites the assistant's **final task summary** so a reader sees the outcome first, still keeps every important fact, and learns up to two useful technical terms along the way. Installation guides cover **OpenCode**, **Claude Code** and **Codex CLI**.

- **What it is:** a Markdown-only skill (`skills/clearcraft/SKILL.md`) plus a short global instruction. No code, no dependencies, no background process, no network calls.
- **What it does:** edits the closing summary the assistant is about to send. Outcome first, technical details separate, zero to two new terms explained.
- **What it never does:** edit files, code, documents or quotations; touch progress messages, internal subagent reports, ordinary questions or requested artifacts; invent outcomes; hide failed checks or risks.
- **License:** [MIT](LICENSE). Repository: `https://github.com/bartoszjaniak/clearcraft`.

One skill, one rule set, one activation block. The three hosts differ only in where the skill is installed and which instruction file is edited — the rules themselves are never duplicated.

> **Status:** this is documentation for a reviewed skill. There is no application build and no test suite. Behavioral evidence is collected as model samples; see [Evaluation](#evaluation).

---

## Why

Technical summaries often lead with implementation details instead of what the reader wanted. Shortening them to sound clean can hide failed checks and risks.

Clearcraft keeps the facts and changes the order. The reader gets the result first, the detail after, and a plain explanation of one or two terms that actually help.

---

## Behavior

When applied to a final task summary, Clearcraft:

1. **Leads with the outcome** — or, if the task is unfinished, the next action plus the current state. It never implies completion without evidence.
2. **Separates needed detail** into a short paragraph or list. No mandatory headings, no empty sections.
3. **Explains zero to two new terms** beside their first use. Terms are only explained when they help the reader understand the change, its verification or a decision. Accurate names of standards and technologies are kept; other jargon is simplified without deleting required names or facts.
4. **Preserves required facts** — numbers, failed checks, warnings, uncertainty, deployment state, commands, paths and links. A change being *made* is distinguished from being *verified*. Nothing is invented.
5. **Adds at most one short everyday analogy**, only when helpful. A literal definition comes first and must stand alone; the analogy states the relevant similarity and any important limit.
6. **Edits silently** before sending. It does not ask for a draft and does not add a report about the editing itself.

### Exclusions

Clearcraft does **not** apply to:

- progress messages and status updates mid-task;
- internal subagent reports;
- ordinary questions and answers;
- requested artifacts such as documents, code and quotations — those are delivered as asked.

Applying the skill never edits files, code, documents or quotations. It edits only the assistant's own closing commentary.

### Educational terms: default cap of two

- Zero is the normal number. Simple changes usually need no definition.
- The **default** cap is two. If four terms compete, the two that most help understanding are chosen and explained; the remaining unnecessary jargon is simplified (required names and facts are kept, not deleted).
- If the user explicitly asks for a deeper or fuller explanation, that request takes precedence and the cap does not apply.
- A term already defined earlier in the visible conversation is **not** redefined unless the user asks.
- Definitions are placed beside first use, not collected in a glossary.

### Analogies and their limits

An analogy is optional and capped at one. The rule is:

- state the literal definition **first**, and make sure it stands alone;
- state the relevant similarity explicitly;
- state the important limit of the comparison.

An analogy that needs long caveats, or that distorts the concept, is dropped. If comparisons are forbidden by a higher-priority instruction, only the literal definition is used.

---

## Examples

The examples below show a raw draft and the edited closing summary. Polish is used for several examples to match real conversations; the skill always follows the language of the conversation.

### No jargon needed (success)

**Raw draft:**

> Changed the submit button label from "Send" to "Create account" in `src/app/checkout.html`. Build passed. No tests cover this file.

**Edited:**

> The button now reads **Create account**.
>
> Build passed. No tests cover `src/app/checkout.html`.

Zero definitions — nothing here needs one. No invented lesson about HTML or UX.

### One or two terms explained (success, Polish)

**Raw draft:**

> Dodano ponawianie żądań z wykładniczym opóźnieniem w `src/http/client.ts`. Testy jednostkowe przechodzą (`npm test -- client.spec.ts`). Błędy 429 i 503 są teraz ponawiane automatycznie. Nie testowano na produkcji.

**Edited:**

> Żądania kończące się błędem **429 lub 503** są teraz ponawiane automatycznie w `src/http/client.ts`.
>
> **Ponawianie** to kolejna próba wysłania tego samego żądania po błędzie. **Wykładnicze opóźnienie** oznacza, że każde kolejne opóźnienie jest mnożone (np. ×2), więc przerwy między próbami wydłużają się, a nie są stałe. Testy jednostkowe przechodzą (`npm test -- client.spec.ts`); nie testowano na produkcji.

Two terms explained, not four. The retry is limited to 429/503 (not any failed request), and the status codes, command and path are kept exactly.

### Partial failure, honest state (Polish)

**Raw draft:**

> Przeniesiono tabele `users` i `orders`. Migracja tabeli `logs` **nie powiodła się** — brakuje uprawnień. Wiersze w `logs` pozostają w starej bazie. Test migracji: 1 błąd, 2 zaliczone.

**Edited:**

> Częściowy sukces: przeniesiono dwie z trzech tabel.
>
> `users` i `orders` są w nowej bazie. Migracja tabeli `logs` **nie powiodła się** — brakuje uprawnień, więc jej wiersze pozostają w starej bazie. Test migracji: 1 błąd, 2 zaliczone.

No claim of full success. The failure, the cause, the unmigrated rows and the test counts are all preserved.

### Unrun tests (Polish)

**Raw draft:**

> Dodano funkcję `parseDuration` w `src/time/parse.ts` dla wejść typu `"1h30m"`. Testy są w `src/time/parse.spec.ts`; komenda: `npm test -- parse.spec.ts`. Nie uruchamiałem testów.

**Edited:**

> Dodano funkcję `parseDuration` w `src/time/parse.ts`, która czyta czasy typu `"1h30m"`.
>
> Testy są w `src/time/parse.spec.ts`; aby je uruchomić: `npm test -- parse.spec.ts`. **Nie uruchomiono testów** — nie wiadomo jeszcze, czy przechodzą.

Tests are stated as *not run*. Command and both paths (`src/time/parse.ts` and `src/time/parse.spec.ts`) are unchanged.

### A term already defined earlier

**Earlier turn (context):**

> Bufor — obszar pamięci, w którym dane czekają, aż program je przetworzy.

**Raw draft:**

> Zwiększono bufor odczytu do 64 KB w `src/io/reader.ts`. Odczyt dużych plików jest teraz szybszy. Nie mierzono dokładnego przyspieszenia.

**Edited:**

> Duże pliki czytają się teraz szybciej.
>
> Bufor odczytu zwiększono do 64 KB w `src/io/reader.ts`. Dokładnego przyspieszenia nie mierzono.

"Bufor" is not redefined — it was already explained in this conversation.

---

## Structure

```
clearcraft/
├── README.md                     # this file
├── LICENSE                       # MIT, copyright bartoszjaniak
├── .gitignore                    # secrets, OS and editor artifacts
├── skills/
│   └── clearcraft/
│       ├── SKILL.md              # the portable skill (trigger, workflow, pre-send check, sources)
│       └── eval.md               # evaluation fixtures and rubric (read only during evaluation)
└── docs/
    ├── plan.md                   # implementation plan
    ├── evaluation.md             # evaluation evidence
    └── specs/
        ├── clearcraft.md         # public specification
        └── clearcraft.html       # standalone specification preview
```

`skills/clearcraft/` is the portable unit and the only thing installed. Its frontmatter uses just `name` and `description` — the fields every Agent Skills host understands. The `name` (`clearcraft`), its length and character set, and the matching directory name already satisfy the shared standard, so `SKILL.md` needs no host-specific extension.

---

## Compatibility: hosts, locations and invocation

Each host finds the skill in its own directory and edits its own instruction file. Pick **one** host per install, update or removal.

| Host | Personal skill directory | Project skill directory | Explicit invocation | Personal instruction file |
| --- | --- | --- | --- | --- |
| OpenCode | `~/.config/opencode/skills/clearcraft/` | `.opencode/skills/clearcraft/` | `@clearcraft` | `~/.config/opencode/AGENTS.md` |
| Claude Code | `~/.claude/skills/clearcraft/` | `.claude/skills/clearcraft/` | `/clearcraft` | `~/.claude/CLAUDE.md` |
| Codex CLI | `~/.agents/skills/clearcraft/` | `.agents/skills/clearcraft/` | `$clearcraft` or the `/skills` selector | `${CODEX_HOME:-$HOME/.codex}/AGENTS.md`, or a nonempty `AGENTS.override.md` in the same home |

**OpenCode compatibility sources.** OpenCode also scans `~/.claude/skills` and `~/.agents/skills`, so a skill installed for another host may still be discovered there. With duplicate `name`s, the native `~/.config/opencode/skills` tree takes precedence over those compatibility sources; a project `.opencode/skills/` takes precedence over global native skills; and an explicitly configured skill source can override discovery. Precedence is not universal across hosts — keep your copies consistent and inspect the path that was actually loaded rather than assuming.

**Codex instruction file.** In the effective Codex home, a **nonempty** `AGENTS.override.md` takes precedence over `AGENTS.md`. Resolve the file before editing instructions:

```bash
CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
if [ -s "$CODEX_HOME/AGENTS.override.md" ]; then
  echo "$CODEX_HOME/AGENTS.override.md"
else
  echo "$CODEX_HOME/AGENTS.md"
fi
```

The skills documentation places the personal skill root at `$HOME/.agents/skills`; the instructions documentation scopes `CODEX_HOME` to the Codex home. Do not assume that changing `CODEX_HOME` moves the documented personal skill root — this is read from the sources, not runtime-tested here.

**Copy versus symlink.** The guides install a **copy** of `skills/clearcraft/`. Copying is our chosen policy, not a format requirement: Claude Code and Codex document symlink support, while OpenCode's symlink discovery was not checked. A copy is what OpenCode is expected to find natively, and it keeps all three hosts working with one procedure.

---

## Installation

Requirements: one supported host installed, `git` available, and Bash on Linux or macOS. The commands use `$HOME` and are written for Bash. Nothing is piped from the network into a shell. These are documentation-only guides — installing for Claude Code or Codex is not performed here and does not change your local configuration.

### 1. Get the files

Clone, or copy an existing checkout. Both work.

```bash
# Option A: clone
git clone "https://github.com/bartoszjaniak/clearcraft" "$HOME/clearcraft"

# Option B: if you already have a checkout, just note its path
```

### 2. Install the skill (copy, never overwrite)

Set `AGENT` to the host you are installing for. Pick it with one line before the block — the block reads the variable in the current shell, so `export` is optional:

```bash
AGENT=claude    # or: AGENT=codex; leave unset to use the default opencode
```

Then run the block. It stops on a symlink or any existing destination — resolve that by hand so nothing unrelated is destroyed. A missing source or a failed copy also stops instead of printing success.

```bash
# Pick the host: opencode (default), claude or codex.
AGENT="${AGENT:-opencode}"
case "$AGENT" in
  opencode) DEST="$HOME/.config/opencode/skills/clearcraft" ;;
  claude)   DEST="$HOME/.claude/skills/clearcraft" ;;
  codex)    DEST="$HOME/.agents/skills/clearcraft" ;;
  *) echo "STOP: AGENT must be one of: opencode, claude, codex."; exit 1 ;;
esac

SRC="$HOME/clearcraft/skills/clearcraft"

if [ -L "$DEST" ]; then
  echo "STOP: $DEST is a symlink. Inspect it and remove it yourself if you are sure."
  exit 1
fi

if [ -e "$DEST" ]; then
  echo "STOP: $DEST already exists. Inspect it and remove it yourself if you are sure."
  exit 1
fi

if [ ! -d "$SRC" ]; then
  echo "STOP: source $SRC not found. Check the clone path from step 1."
  exit 1
fi

mkdir -p "$(dirname "$DEST")"
if ! cp -R "$SRC" "$DEST"; then
  echo "STOP: copy failed. A partial copy may remain at $DEST — inspect and remove it before retrying."
  exit 1
fi
echo "Installed $AGENT skill to $DEST"
```

For a **project-scoped** install, supplying `DEST` from outside is not enough: the `case` assigns `DEST` itself, so your value is overwritten. In the copied block, edit the `DEST=` line of the **one** host case you selected and replace it with the absolute project root path from the table:

```bash
# inside the opencode) branch, for example:
DEST="/absolute/path/to/repo/.opencode/skills/clearcraft"
```

The rest of the block is unchanged. Step 3 then appends the activation block to that host's **project** instructions — OpenCode `AGENTS.md` in the project, Claude Code `CLAUDE.md`, Codex `AGENTS.md` (honoring a nonempty project override if one exists). Keep **one** scope per install: a project skill directory with a project instruction file, or a personal skill directory with the personal instruction file. Mixing them creates two installs that are easy to update or remove by mistake. The `case` selector resolves only the three personal destinations — it has **no project mode**, and this guide makes no claim that it picks a project path for you.

### 3. Activate globally

Append a short, host-neutral activation section to your host's instruction file, preserving every existing line. Do this by hand: it keeps the ordered markers visible and cannot silently delete your other instructions.

1. Open your host's instruction file in an editor — `~/.config/opencode/AGENTS.md` (OpenCode), `~/.claude/CLAUDE.md` (Claude Code), or the effective Codex instruction file printed by the resolution snippet above. Create it if it does not exist.
2. Count the markers `<!-- clearcraft:start -->` and `<!-- clearcraft:end -->` and act on exactly one case:
   - **Zero markers** — no `start` and no `end`: append the block below verbatim at the end of the file, then save.
   - **Exactly one correctly ordered pair** — one `start` followed by one `end`: activation is already done. Stop and change nothing.
   - **Anything else** — only one of the two markers present, duplicated markers, or `end` before `start`: **STOP**. Repair the markers by hand until you have either zero markers or one correctly ordered pair, then re-check. Append nothing.
3. Save the file.

```markdown
<!-- clearcraft:start -->
Before sending a final user-facing task summary, load the `clearcraft` skill
unless it is already loaded. Apply it to that closing summary only — not to
internal subagent reports, progress messages, ordinary questions or requested
artifacts such as documents and code.
Respect an explicit opt-out: if the user says not to apply clearcraft — in the
session or in the project instructions — do not apply it.
If the skill is not available, do not pretend it loaded; say so plainly.
<!-- clearcraft:end -->
```

The same block is used for every host: the markers are unique, the wording names no tool, and it only asks the assistant to load the skill before a final summary. The pair is ordered: `<!-- clearcraft:start -->` first, `<!-- clearcraft:end -->` last, exactly once. An explicit opt-out is respected, and an unavailable skill must be reported honestly.

---

## Activation: manual and automatic

- **Manual invocation** depends on the host: `@clearcraft` (OpenCode), `/clearcraft` (Claude Code), or `$clearcraft` / the `/skills` selector (Codex CLI).
- **Automatic use** is driven by the activation instruction. It is a *request to the assistant*, not a guaranteed runtime hook. Each host discovers the skill natively when the files are in place, but whether an assistant loads and applies it in a given session is up to the model and the instructions in effect.

**No behavioral guarantee.** Clearcraft cannot promise that every model, in every session, will apply the editing style. It can only make the instruction available and easy to load.

### Native discovery vs instruction-based activation

Two different mechanisms, worth keeping apart:

- **Native discovery** — the host lists the skill from its personal skill directory. This is a property of the files being in the right place.
- **Instruction-based activation** — the global instruction block asks the assistant to load the skill before a final task summary. This is a behavioral instruction that depends on the model.

To verify discovery, **start a fresh session** and check the available skill list. If the current session's skill list is stale, it will not reflect a just-installed skill; a fresh session will. Do not claim discovery from reading the Markdown alone.

### Memory limits

The skill's "do not redefine a term already explained" rule works only within the **visible conversation context**. There is no persistent memory and no vocabulary stored on disk. If a definition is no longer visible, the skill cannot know the term was already explained — and it will not claim to.

---

## Opting out

Disable the editing behavior without uninstalling:

- **This session:** say so in the conversation, e.g. *"Do not apply clearcraft in this session."*
- **This project:** add a line to the project's instructions, e.g. *"Nie stosuj clearcraft w tym projekcie."*

An opt-out removes only the receiving skill's contribution. Independent obligations remain: safety requirements, and truthful reporting of tests, warnings and unperformed checks still apply.

---

## Updating

An update refreshes the checkout, shows what differs, asks for explicit confirmation, moves the installed skill to a unique backup **outside every skill discovery root**, and only then copies the fresh version in. Nothing is deleted before you confirm.

> **The block below is personal-scope only.** Its `case` resolves the three personal skill directories, so after a project-scoped install it targets the wrong place — or finds nothing at all. For a project install, edit the selected case's `DEST=` to the absolute project path exactly as in installation step 2, and inspect the host's **project** instruction file (and a nonempty Codex project override) to confirm the activation block there is still what you expect. Never run it unchanged against a machine where only a project install exists: check the resolved path before confirming, so a personal install is not updated by mistake.

```bash
# Pick the host you installed for: opencode (default), claude or codex.
AGENT="${AGENT:-opencode}"
case "$AGENT" in
  opencode) DEST="$HOME/.config/opencode/skills/clearcraft" ;;
  claude)   DEST="$HOME/.claude/skills/clearcraft" ;;
  codex)    DEST="$HOME/.agents/skills/clearcraft" ;;
  *) echo "STOP: AGENT must be one of: opencode, claude, codex."; exit 1 ;;
esac

CLONE="$HOME/clearcraft"
SRC="$CLONE/skills/clearcraft"
BACKUP="$HOME/.local/state/clearcraft-backups/$AGENT-clearcraft-$(date +%Y%m%d-%H%M%S)"

if [ ! -d "$DEST" ]; then
  echo "STOP: not installed yet. Run the installation steps first."
  exit 1
fi

if [ -L "$DEST" ]; then
  echo "STOP: $DEST is a symlink; this update expects a copied install."
  exit 1
fi

# 1. Refresh the checkout first, so the comparison is meaningful.
#    A plain `diff` cannot tell your local edits from upstream changes. Even
#    after a fast-forward, a difference in $DEST can still come from upstream,
#    so the diff alone does not establish provenance: compare against a known
#    baseline, or review by hand, before treating a difference as your own.
if ! git -C "$CLONE" pull --ff-only; then
  echo "STOP: could not fast-forward $CLONE. Resolve it yourself, then retry."
  exit 1
fi

if [ ! -d "$SRC" ]; then
  echo "STOP: source $SRC not found."
  exit 1
fi

# 2. Inspect what differs before touching anything.
diff -rq "$DEST" "$SRC" || true
echo "Differences listed above. Edits you made on purpose must be merged by hand."

# 3. Nothing is deleted until you confirm.
printf 'Replace %s with a fresh copy? [y/N] ' "$DEST"
read -r answer
case "$answer" in
  y|Y) ;;
  *) echo "Aborted. Nothing changed."; exit 0 ;;
esac

# 4. Move the old install to a unique backup (no rm -rf), then copy fresh.
#    A seconds-resolution timestamp can still collide; refuse an existing
#    backup so mv cannot nest $DEST inside an existing directory.
if [ -e "$BACKUP" ] || [ -L "$BACKUP" ]; then
  echo "STOP: backup $BACKUP already exists. Move it aside, then retry."
  exit 1
fi
mkdir -p "$(dirname "$BACKUP")"
if ! mv "$DEST" "$BACKUP"; then
  echo "STOP: could not move the old install to $BACKUP; nothing replaced."
  exit 1
fi
if ! cp -R "$SRC" "$DEST"; then
  echo "STOP: copy failed. Restore manually from $BACKUP."
  exit 1
fi
echo "Updated $DEST (previous copy kept at $BACKUP)"
```

`diff -rq` lists what differs, but it cannot separate your local edits from upstream changes — and a fast-forward does not fix that: a difference can still be an upstream change. Determine provenance against a known baseline, or review by hand, before merging. If the differences are your own edits, merge them by hand — do not overwrite local changes without deciding. The activation block in the instruction file is left unchanged by an update. Because the backup lives under `~/.local/state/clearcraft-backups/`, outside every discovery root, a backup copy is never picked up as an installed skill.

---

## Removal

Removing Clearcraft never deletes the whole instructions file and never touches unrelated instructions. Neither step deletes anything outright: the skill directory is moved to a backup, and the activation block is removed by hand.

> **The block below is personal-scope only**, like the update block: its `case` resolves the three personal skill directories. For a project-scoped install, edit the selected case's `SKILL=` to the absolute project path the same way as in installation step 2, and remove the marked section from the host's **project** instruction file (and a nonempty Codex project override if one exists) rather than the personal one — so a personal install is never moved by mistake.

**1. Move the installed skill to a backup (no delete).**

```bash
# Pick the host you installed for: opencode (default), claude or codex.
AGENT="${AGENT:-opencode}"
case "$AGENT" in
  opencode) SKILL="$HOME/.config/opencode/skills/clearcraft" ;;
  claude)   SKILL="$HOME/.claude/skills/clearcraft" ;;
  codex)    SKILL="$HOME/.agents/skills/clearcraft" ;;
  *) echo "STOP: AGENT must be one of: opencode, claude, codex."; exit 1 ;;
esac

QUARANTINE="$HOME/.local/state/clearcraft-backups/$AGENT-clearcraft-removed-$(date +%Y%m%d-%H%M%S)"

if [ -L "$SKILL" ]; then
  echo "STOP: $SKILL is a symlink. Inspect it and remove it yourself if you are sure."
  exit 1
fi

if [ -d "$SKILL" ]; then
  # A seconds-resolution timestamp can still collide; refuse an existing
  # quarantine so mv cannot nest $SKILL inside an existing directory.
  if [ -e "$QUARANTINE" ] || [ -L "$QUARANTINE" ]; then
    echo "STOP: quarantine $QUARANTINE already exists. Move it aside, then retry."
    exit 1
  fi
  mkdir -p "$(dirname "$QUARANTINE")"
  if ! mv "$SKILL" "$QUARANTINE"; then
    echo "STOP: could not move $SKILL to $QUARANTINE; nothing changed."
    exit 1
  fi
  echo "Skill moved to $QUARANTINE (not deleted). Delete it yourself later if you want."
else
  echo "No installed skill directory found."
fi
```

**2. Remove the activation block by hand.**

Do **not** use a `sed` line range such as `/<!-- clearcraft:start -->/,/<!-- clearcraft:end -->/d`. If the closing marker is missing, the range runs to the end of the file and deletes all of your remaining instructions. Edit the file manually instead:

1. Open your host's instruction file in an editor: `~/.config/opencode/AGENTS.md` (OpenCode), `~/.claude/CLAUDE.md` (Claude Code), or the effective Codex instruction file printed by the resolution snippet above.
2. Find the two markers: `<!-- clearcraft:start -->` and `<!-- clearcraft:end -->`. Confirm there is exactly **one** correctly paired start/end pair.
3. Delete only the lines from the start marker to the end marker, inclusive. Change nothing else.
4. If a marker is missing or duplicated, fix the markers by hand and delete nothing else.

For Codex, inspect **both** possible instruction files — `AGENTS.md` and a nonempty `AGENTS.override.md` in the effective home — and remove Clearcraft's marked section wherever it was actually added. An override may have appeared after installation, so a stale block can sit in either file; remove only the marked section and never delete a whole file.

If you never activated globally, step 2 is a no-op. The file keeps all your other instructions.

---

## Evaluation

There is **no application build and no unit-test suite**. Evaluation is deliberately modest and honest:

- **Structural inspection** — the files are well-formed: valid frontmatter, `name` equal to `clearcraft`, and matching activation markers. This does not prove discovery or behavior.
- **Model samples** — each evaluation input run through a model with the skill loaded, recording the observed summary. This is the only behavioral evidence. One sample is not proof of reliability.
- **Skill discovery** — observation of an actual loaded skill, never claimed from file inspection.

Evidence differs by host. The table records what is actually confirmed and what is only grounded in official documentation:

| Host | What is confirmed | What is not claimed |
| --- | --- | --- |
| OpenCode | Actual current-session load confirmed: the skill is listed and the activation instruction is present. | Fresh-session discovery is untested. |
| Claude Code | Official documentation for the skill directory, `/clearcraft` invocation and `CLAUDE.md`. README commands were shell-checked in a disposable sandbox with a mocked `git`: 12 install scenarios across the 3 hosts, 7 Bash blocks passing `bash -n`, and `SKILL.md` frontmatter parsed with the system PyYAML. The official `skills-ref` validator was not available and was not run, so no validator pass is claimed. Details in [`docs/evaluation.md`](docs/evaluation.md). | No runtime loading test. Claude Code was not installed or run on this machine; the guide is documentation only. |
| Codex CLI | Official documentation for the skill directory, `$clearcraft` / `/skills` invocation and the instruction-file override precedence. The same sandbox shell checks cover the README commands alongside the other hosts. | No runtime loading test. Codex was not installed or run on this machine; the guide is documentation only. |

Only checks that were actually run are reported here, and counts stay with the evidence in [`docs/evaluation.md`](docs/evaluation.md) rather than being restated as a total. Evaluation fixtures and the shared rubric live in [`skills/clearcraft/eval.md`](skills/clearcraft/eval.md). Collected evidence, observed outcomes and limitations live in [`docs/evaluation.md`](docs/evaluation.md). An initial generated pass produced model samples; one case (S2, the multi-term input) failed on its first attempt — it dropped two provided facts (jitter and the configurable per-attempt timeout) and defined exponential backoff loosely — and the report keeps that failed attempt alongside the corrected rerun rather than hiding it. The full public specification is in [`docs/specs/clearcraft.md`](docs/specs/clearcraft.md) (preview: [`docs/specs/clearcraft.html`](docs/specs/clearcraft.html)).

---

## Sources

- [Prosty język — gov.pl](https://www.gov.pl/web/cyfryzacja/prosty-jezyk) — plain Polish: outcome first, explain technical terms.
- [Teaching analogies — CSUN](https://www.csun.edu/~vceed002/books/sourcebook/chapters/10-analogies/teaching-analogies.html) — map relevant similarities and state analogy limits.
- [WCAG supplemental — implicit explained](https://www.w3.org/WAI/WCAG2/supplemental/patterns/o3p12-implicit-explained/) — explain nonliteral meanings; supplemental guidance, not a standalone WCAG conformance requirement.
- [Agent Skills specification](https://agentskills.io/specification) — portable skill format and constraints.
- [OpenCode skills](https://opencode.ai/v2/docs/skills) — native skill structure and discovery.
- [OpenCode instructions](https://opencode.ai/v2/docs/instructions) — global activation guidance.
- [Claude Code skills](https://code.claude.com/docs/en/skills) — personal skill discovery and `/skill-name` invocation.
- [Claude Code memory](https://code.claude.com/docs/en/memory) — personal `~/.claude/CLAUDE.md` guidance.
- [Codex build skills](https://developers.openai.com/codex/build-skills) — personal/project skill locations and explicit invocation.
- [Codex AGENTS.md](https://developers.openai.com/codex/agent-configuration/agents-md) — effective Codex home and instruction override precedence.

---

## License

[MIT](LICENSE) © 2026 bartoszjaniak.
