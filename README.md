# Clearcraft

An [OpenCode](https://opencode.ai) skill that rewrites the assistant's **final task summary** so a reader sees the outcome first, still keeps every important fact, and learns up to two useful technical terms along the way.

- **What it is:** a Markdown-only skill plus a short global instruction. No code, no dependencies, no background process, no network calls.
- **What it does:** edits the closing summary the assistant is about to send. Outcome first, technical details separate, zero to two new terms explained.
- **What it never does:** edit files, code, documents or quotations; touch progress messages, internal subagent reports, ordinary questions or requested artifacts; invent outcomes; hide failed checks or risks.
- **License:** [MIT](LICENSE). Repository: `https://github.com/bartoszjaniak/clearcraft`.

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

The installed unit is a **copy** of `skills/clearcraft/` placed in `~/.config/opencode/skills/clearcraft/`. Copying is used on purpose instead of a symlink: symlink discovery is not verified for every setup, and a copy is what OpenCode is expected to find natively.

---

## Installation

Requirements: OpenCode installed, `git` available, and Bash on Linux or macOS. The commands use `$HOME` and are written for Bash on Linux or macOS. Nothing is piped from the network into a shell.

### 1. Get the files

Clone, or copy an existing checkout. Both work.

```bash
# Option A: clone
git clone "https://github.com/bartoszjaniak/clearcraft" "$HOME/clearcraft"

# Option B: if you already have a checkout, just note its path
```

### 2. Install the skill (copy, never overwrite)

Stop if the destination already exists — a file, directory or symlink. That is a collision you must resolve manually, so nothing unrelated is destroyed. The source is checked too, and a failed copy stops instead of printing success.

```bash
DEST="$HOME/.config/opencode/skills/clearcraft"
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

mkdir -p "$HOME/.config/opencode/skills"
if ! cp -R "$SRC" "$DEST"; then
  echo "STOP: copy failed. A partial copy may remain at $DEST — inspect and remove it before retrying."
  exit 1
fi
echo "Installed to $DEST"
```

### 3. Activate globally

Append a short activation section to the global instructions file, preserving every existing line. Do this by hand: it keeps the ordered markers visible and cannot silently delete your other instructions.

1. Open `$HOME/.config/opencode/AGENTS.md` in an editor. Create it if it does not exist.
2. Confirm it holds **exactly one** ordered pair of markers — `start` before `end`. If any marker is missing, duplicated or out of order, fix that by hand first and append nothing.
3. If a valid ordered pair is already present, stop: activation is done.
4. Otherwise append the block below verbatim at the end of the file, then save.

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

The pair is ordered: `<!-- clearcraft:start -->` first, `<!-- clearcraft:end -->` last, exactly once. The block itself states that an explicit opt-out is respected and that an unavailable skill must be reported honestly.

---

## Activation: manual and automatic

- **Manual invocation:** type `@clearcraft` in a conversation to invoke the skill directly.
- **Automatic use:** driven by the activation instruction. It is a *request to the assistant*, not a guaranteed runtime hook. OpenCode discovers the skill natively, but whether an assistant loads and applies it in a given session is up to the model and the instructions in effect.

**No behavioral guarantee.** Clearcraft cannot promise that every model, in every session, will apply the editing style. It can only make the instruction available and easy to load.

### Native discovery vs instruction-based activation

Two different mechanisms, worth keeping apart:

- **Native discovery** — OpenCode lists the skill from `~/.config/opencode/skills/clearcraft/`. This is a property of the files being in the right place.
- **Instruction-based activation** — the global `AGENTS.md` block asks the assistant to load the skill before a final task summary. This is a behavioral instruction that depends on the model.

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

An update refreshes the checkout, shows what differs, asks for explicit confirmation, moves the installed skill to a unique backup **outside the repository**, and only then copies the fresh version in. Nothing is deleted before you confirm.

```bash
DEST="$HOME/.config/opencode/skills/clearcraft"
CLONE="$HOME/clearcraft"
SRC="$CLONE/skills/clearcraft"
BACKUP="$HOME/.config/opencode/skills-backup/clearcraft-$(date +%Y%m%d-%H%M%S)"

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

`diff -rq` lists what differs, but it cannot separate your local edits from upstream changes — and a fast-forward does not fix that: a difference can still be an upstream change. Determine provenance against a known baseline, or review by hand, before merging. If the differences are your own edits, merge them by hand — do not overwrite local changes without deciding. The activation block in `AGENTS.md` is left unchanged by an update.

---

## Removal

Removing Clearcraft never deletes the whole instructions file and never touches unrelated instructions. Neither step deletes anything outright: the skill directory is moved to a backup, and the activation block is removed by hand.

**1. Move the installed skill to a backup (no delete).**

```bash
SKILL="$HOME/.config/opencode/skills/clearcraft"
QUARANTINE="$HOME/.config/opencode/skills-backup/clearcraft-removed-$(date +%Y%m%d-%H%M%S)"

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

1. Open `$HOME/.config/opencode/AGENTS.md` in an editor.
2. Find the two markers: `<!-- clearcraft:start -->` and `<!-- clearcraft:end -->`. Confirm there is exactly **one** correctly paired start/end pair.
3. Delete only the lines from the start marker to the end marker, inclusive. Change nothing else.
4. If a marker is missing or duplicated, fix the markers by hand and delete nothing else.

If you never activated globally, step 2 is a no-op. The file keeps all your other instructions.

---

## Evaluation

There is **no application build and no unit-test suite**. Evaluation is deliberately modest and honest:

- **Structural inspection** — the files are well-formed: valid frontmatter, ID/name `clearcraft`, matching activation markers. This does not prove discovery or behavior.
- **Model samples** — each evaluation input run through a model with the skill loaded, recording the observed summary. This is the only behavioral evidence. One sample is not proof of reliability.
- **Actual OpenCode discovery** — checking a fresh session lists the skill. Not claimed from file inspection.

Evaluation fixtures and the shared rubric live in [`skills/clearcraft/eval.md`](skills/clearcraft/eval.md). Collected evidence, observed outcomes and limitations live in [`docs/evaluation.md`](docs/evaluation.md). An initial generated pass produced model samples; one case (S2, the multi-term input) failed on its first attempt — it dropped two provided facts (jitter and the configurable per-attempt timeout) and defined exponential backoff loosely — and the report keeps that failed attempt alongside the corrected rerun rather than hiding it. The full public specification is in [`docs/specs/clearcraft.md`](docs/specs/clearcraft.md) (preview: [`docs/specs/clearcraft.html`](docs/specs/clearcraft.html)).

---

## Sources

- [Prosty język — gov.pl](https://www.gov.pl/web/cyfryzacja/prosty-jezyk) — plain Polish: outcome first, explain technical terms.
- [Teaching analogies — CSUN](https://www.csun.edu/~vceed002/books/sourcebook/chapters/10-analogies/teaching-analogies.html) — map relevant similarities and state analogy limits.
- [WCAG supplemental — implicit explained](https://www.w3.org/WAI/WCAG2/supplemental/patterns/o3p12-implicit-explained/) — explain nonliteral meanings; supplemental guidance, not a standalone WCAG conformance requirement.
- [OpenCode skills](https://opencode.ai/v2/docs/skills) — native skill structure and discovery.
- [OpenCode instructions](https://opencode.ai/v2/docs/instructions) — global activation guidance.

---

## License

[MIT](LICENSE) © 2026 bartoszjaniak.
