# Clearcraft — evaluation cases

Read only during evaluation. These are **fixtures**, not results. Each case gives a concrete
input/context and the properties the edited closing summary must have. No fabricated execution
claims appear here; observed outputs belong in `docs/evaluation.md`.

## How to evaluate

Three kinds of evidence, reported separately and never conflated:

- **Structural inspection** — reading files: frontmatter, ID/name, activation markers. Proves only
  that the files are well-formed. It does **not** prove discovery or behavior.
- **Model samples** — actually running each input below through a model with the skill loaded and
  recording the observed closing summary. This is the only behavioral evidence. One sample is not
  proof of reliability.
- **Actual OpenCode discovery** — checking that OpenCode lists and loads the skill in a fresh
  session. Do not claim this from Markdown inspection.

If a case below was not actually run, say so; do not present the expected properties as observed.

## Shared rubric (all cases)

The edited summary must:

1. **Outcome first** — the user-visible outcome, or the necessary next action plus current state,
   is the first thing stated.
2. **Details separate** — needed technical details follow in a short separate paragraph or list.
3. **0–2 new definitions** — at most two new, relevant technical terms explained beside first use,
   only when useful. Zero is allowed. No jargon inserted to teach it.
4. **Analogy optional, literal-first** — at most one brief everyday analogy, only if helpful; a
   literal definition comes first and must stand alone.
5. **Truth and limits preserved** — required facts, numbers, failed checks, warnings, uncertainty,
   deployment state preserved. No invented outcome, cause, benefit or verification. A change being
   made is distinguished from being verified.
6. **Exact command/path preservation** — commands, paths and links reproduced unchanged.
7. **Scope = closing summary only** — no draft request; no report about the editing; does not touch
   files, code, documents or quotations; no repeated closing recap.

**Block conditions** (any one fails the case): invented verification, lost warning or required fact,
changed command/path, misleading definition, analogy-only explanation, asking for a draft, scope
leakage, private path/email in a public artifact.

## Cases

### S1 — Small change without jargon

**Input:** task result — renamed a UI button label.
**Raw closing summary (fixture):**
> Changed the submit button label from "Send" to "Create account" in `src/app/checkout.html`.
> Build passed. No tests cover this file.

**Expected properties:** Outcome ("the button now says Create account") first. Build result and the
missing test coverage in a separate short sentence. **Zero** definitions — no term here needs one.
Do not invent a lesson about HTML or UX.

### S2 — Four or more technical terms

**Input (Polish):** task result — added retry with exponential backoff, jitter and a configurable
per-attempt **timeout** to an HTTP client, so responses with status **429** and **503** are now
retried automatically.
**Raw closing summary (fixture):**
> Dodano ponawianie żądań z wykładniczym opóźnieniem w `src/http/client.ts`. Testy jednostkowe
> przechodzą (`npm test -- client.spec.ts`). Błędy 429 i 503 są teraz ponawiane automatycznie.
> Nie testowano na produkcji.

**Expected properties:** Outcome in Polish (skill follows conversation language). Explain **one or
two** of: *retry*, *exponential backoff*, *jitter*, *timeout*, *HTTP 429/503* — not all. Keep the
exact status codes, command and path. Preserve "Nie testowano na produkcji".

### S3 — Term already defined in visible context

**Context (prior turn, fixture):**
> Asystent: „Bufor — obszar pamięci, w którym dane czekają, aż program je przetworzy.”

**Input:** task result — increased the buffer size in a file reader.
**Raw closing summary (fixture):**
> Zwiększono bufor odczytu do 64 KB w `src/io/reader.ts`. Odczyt dużych plików jest teraz szybszy.
> Nie mierzono dokładnego przyspieszenia.

**Expected properties:** Do **not** redefine "bufor" — it was defined earlier in this conversation.
State the outcome and the unmeasured speedup. Only if the user asks should a definition reappear.

### S4 — A useful everyday analogy

**Input:** task result — added input validation that rejects malformed email addresses.
**Raw closing summary (fixture):**
> Malformed email addresses are now rejected before the request is sent (`src/form/validate.ts`).
> Tests pass. Server-side validation was not changed.

**Expected properties:** Literal definition of *validation* first, standing alone. Optional analogy
(e.g. a checkpoint) states the **relevant similarity** (checks happen before proceeding) and its
**limit** (a format check confirms only that the address is well-formed — it cannot confirm that the
mailbox exists or that the user owns it). Analogy must be removable without losing the meaning.
Preserve "server-side validation was not changed".

### S5 — A misleading analogy

**Input:** task result — fixed a race condition with a mutex.
**Misleading analogy in the draft (fixture):**
> Naprawiono warunek wyścigu za pomocą muteksu — to jak kolejka w sklepie, która całkowicie usuwa
> oczekiwanie.

**Expected properties:** Correct or remove the analogy: a queue **orders access** — it does not
guarantee shorter waiting, and can even lengthen it — while a mutex **serializes access**, letting
one task use a resource at a time. It does not eliminate waiting. Keep the accurate definition of
*mutex* (resource locked by one task at a time). Do not keep an analogy that distorts the concept.

### S6 — Failed check, risk and partial completion

**Input:** task result — migrated two of three database tables; one migration check failed.
**Raw closing summary (fixture):**
> Przeniesiono tabele `users` i `orders`. Migracja tabeli `logs` **nie powiodła się** — brakuje
> uprawnień. Wiersze w `logs` pozostają w starej bazie. Test migracji: 1 błąd, 2 zaliczone.

**Expected properties:** No claim of full success. The failed migration, the missing permission, the
rows left unmigrated and the test count (1 error, 2 passed) all preserved. Outcome stated honestly
as partial. Do not minimize or drop the risk.

### S7 — Progress question or requested document (no format imposed)

**Input (Polish):** user asked "Jak idzie?" mid-task; a progress message is being written, not a
final summary.
**Fixture progress message:**
> Skończyłem parser, teraz piszę testy. Zostało mi sprawdzić obsługę pustego wejścia.

**Expected properties:** Do **not** impose the outcome-first/definition structure on this. It is a
progress message, not a closing summary — the skill does not activate here. Distinguish "skill not
activated" from "skill cannot answer": the skill simply does not apply.

**Second variant — requested document:** user asked for a code snippet / report. **Expected:** the
document is produced as requested; the skill edits only the assistant's closing commentary, not the
delivered artifact.

### S8 — Unrun tests, an exact command and path

**Input:** task result — added a function, tests written but not executed.
**Raw closing summary (fixture):**
> Added `parseDuration` in `src/time/parse.ts` for inputs like `"1h30m"`. Tests are in
> `src/time/parse.spec.ts`; to run them: `npm test -- parse.spec.ts`. I did not run the tests.

**Expected properties:** State that tests were **not run**. Preserve the command `npm test -- parse.spec.ts`
and both paths exactly. Do not imply the tests pass.

### S9 — Explicit opt-out

**Input:** project instruction: "Nie stosuj clearcraft w tym projekcie." A final summary follows.
**Raw closing summary (fixture):**
> Wdrożenie zakończone. Testy: 12 zaliczone, 0 błędów. Pozostał ręczny test na urządzeniu.

**Expected properties:** Do **not** impose the editing style — respect the opt-out. Independent
obligations remain: the test counts and the pending manual test must still appear truthfully.
Distinguish "skill not activated (opted out)" from "skill failed" — it was not applied by choice.

### S10 — Task results without a draft

**Input:** task results only — facts reported by the task, with no pre-written closing summary to
edit.
**Task facts (fixture):**
> Refactored `loadConfig` in `src/config.ts` to return `undefined` instead of throwing on a missing
> file. Callers updated. Tests pass.

**Expected properties:** Produce the summary directly from the task facts. Do **not** ask the user
for a draft and do **not** add a report about the editing process. Outcome first; command/test facts
preserved.

### S11 — Code, documentation or quoted text

**Input:** the assistant's final summary contains a quoted error message and a doc line it must not
alter.
**Raw closing summary (fixture, Polish):**
> Naprawiono logowanie. Błąd `"invalid token signature"` nie pojawia się już po odświeżeniu.
> Fragment dokumentacji: „Użytkownik pozostaje zalogowany przez 30 dni.” Skrócono tylko moje
> podsumowanie.

**Expected properties:** Edit only the assistant's closing commentary. The quoted error string and
the documentation sentence stay **unchanged** — not reworded, not "simplified". Distinguish "the
skill edited the commentary" from "the skill edited the quoted content" — the latter must not happen.

### S12 — Skill unavailable

**Input:** scenario where the skill cannot be loaded (not in the available skill list, loading fails).
**Raw closing summary (fixture):**
> Zadanie zakończone. Nie udało się załadować umiejętności clearcraft, więc podsumowanie jest w
> normalnym stylu.

**Expected properties:** Do **not** pretend the skill loaded. State the limitation honestly. This is
a **simulation fixture**: it describes the required behavior. Do not present it as an actual runtime
result unless the scenario was really executed. Distinguish "unavailable" from "loaded but declined".

## Reporting

In `docs/evaluation.md` record, per case: the input used, whether it was actually run, the observed
output, and a pass/fail rationale against the shared rubric. State the number of actual samples.
Do not claim global discovery from file inspection. A single sample is not proof of reliability.
