# Clearcraft — evaluation report

Evaluation of the `clearcraft` skill against the fixtures in
[`skills/clearcraft/eval.md`](../skills/clearcraft/eval.md) and the contract in
[`docs/specs/clearcraft.md`](specs/clearcraft.md).

## Method and honesty statement

This report is a **single assistant pass** with the skill text loaded in **one context**. Each
case below was applied by the same model instance reading `SKILL.md`, `eval.md` and the spec into
context, then producing the edited closing summary in that one pass.

This is **not** OpenCode runtime auto-discovery. Discovery was **not checked**: no fresh OpenCode
session was started and no skill-list entry was observed. The method here therefore provides only
**model-sample** evidence and **structural inspection** evidence, reported separately below. One
pass per case is a single sample; a single sample is not proof of reliability.

Not applicable in this repository: there is **no application build and no unit-test suite** (see
spec §10), so no build or test-runner evidence is claimed and none was run.

Counts for this report:

- **Real generated samples: 10** — S1, S2 (two attempts: Attempt 1 FAIL, Attempt 2 corrected), S3,
  S4, S5, S6, S8, S10, S11. S2 counts as two real generated outputs because its first output was
  wrong and a second corrected output was generated; the failed attempt is kept, not hidden.
- **Non-applicable / boundary (no rewrite imposed): 2** — S7, S9. Reviewed as expected boundaries,
  not observed runtime activations (runtime not checked).
- **Requirement reviewed (not executed): 1** — S12 (the unavailable-skill scenario is a fixture
  describing required behavior; it was **not** an actual runtime failure and no pass is claimed).

S7 and S9 are boundaries where the skill must *not* apply, so they are not counted as generated
summaries. S11 is a generated sample *and* a boundary case (scope limited to the closing
commentary).

The lead reviewer (a separate review pass, not a fresh model engine) found that the first S2 output
was wrong: it omitted the added jitter and the configurable per-attempt timeout, and it described
exponential backoff as merely lengthening the interval instead of exponentially multiplying it.
S2 was rerun and corrected below. No failure has been removed.

## Structural inspection

Structural inspection proves only that files are well-formed. It does **not** prove discovery or
behavior.

| Check | Result |
| --- | --- |
| `SKILL.md` has YAML frontmatter | Pass — `---` block present |
| Frontmatter `name` equals directory-derived ID `clearcraft` | Pass — `name: clearcraft` |
| `description` present and describes trigger/scope | Pass |
| Activation markers referenced for install | Present in `README.md` and spec (`<!-- clearcraft:start -->` / `<!-- clearcraft:end -->`) |
| Private paths / absolute home dirs / emails in public files | None found. `grep` hits for `absolute` were CSS `position: absolute` rules only |
| Files expected by spec present | `SKILL.md`, `eval.md`, `README.md`, `LICENSE`, `docs/specs/clearcraft.md`, `docs/specs/clearcraft.html`, `docs/plan.md`, `.gitignore` |
| OpenCode discovery | **Not checked** — no fresh-session listing was run |

These structural commands were actually executed with a shell; results are reproduced above. No
build check applies.

## Shared rubric

Used for every generated sample (from `eval.md`):

1. Outcome first.
2. Details separate.
3. 0–2 new definitions, only when useful.
4. Analogy optional, literal-first.
5. Truth and limits preserved; made vs. verified distinguished.
6. Exact command/path preservation.
7. Scope = closing summary only.

## Generated samples

### S1 — Small change without jargon

- **Input fixture:** renamed the submit button label; build passed; no tests cover the file.
- **Actually run:** Yes (generated sample).
- **Observed output:**

> The checkout button now says **Create account** instead of **Send** (`src/app/checkout.html`).
>
> Build passed. No tests cover this file.

- **Rationale:** Outcome first (rubric 1). Build result and missing coverage in a separate short
  sentence (2). Zero definitions, no invented HTML/UX lesson (3, 5). Path unchanged (6).
  **Pass.**

### S2 — Four or more technical terms

- **Input fixture (Polish):** retry with exponential backoff, jitter, configurable per-attempt
  timeout; 429/503 now retried (`eval.md` lines 58–60).
- **Actually run:** Yes (generated sample) — two attempts.

**Attempt 1 — FAIL.** Observed output:

> Klient HTTP ponawia teraz automatycznie żądania, które zwróciły 429 lub 503 (`src/http/client.ts`).
>
> Ponawianie (retry) to powtórzenie żądania po nieudanej próbie. Wykładniczy backoff wydłuża przerwę
> między kolejnymi próbami, żeby nie zalewać serwera.
>
> Testy jednostkowe przechodzą (`npm test -- client.spec.ts`). Nie testowano na produkcji.

- **Failure reason:** The output **omits two provided facts** — the added **jitter** (random spread
  of the intervals) and the **configurable per-attempt timeout** — both stated in the input fixture
  (`eval.md` lines 58–60). It also describes **exponential backoff** as simply "wydłuża przerwę"
  (lengthens the interval), which is not the accurate mechanism: the interval is **multiplied** by a
  factor on each successive attempt. Dropping required facts and giving a loose definition violates
  rubric 5 and spec §3. **Fail.**

**Attempt 2 — corrected.** Observed output:

> Klient HTTP ponawia teraz automatycznie żądania zakończone błędem 429 lub 503 (`src/http/client.ts`).
>
> Ponawianie (retry) to powtórzenie żądania po nieudanej próbie. Przy wykładniczym backoffie przerwa
> między próbami nie rośnie o stałą wartość, lecz mnoży się przy każdej kolejnej próbie, dlatego
> serwer dostaje coraz więcej czasu. Do przerwy dodaje się losowy rozrzut (jitter), żeby ponowienia
> wielu klientów nie trafiały w ten sam moment, a limit czasu pojedynczej próby (timeout) można
> ustawić w konfiguracji.
>
> Testy jednostkowe przechodzą (`npm test -- client.spec.ts`). Nie testowano na produkcji.

- **Rationale:** Outcome in Polish, conversation language preserved (spec §1, not rubric 7 — rubric
  7 is scope). Exactly two terms explained at first use: retry and exponential backoff (rubric 3).
  The corrected definition states that the interval is **multiplied** per attempt, and the summary
  now carries the two previously missing facts in plain words — the random **jitter** spread and the
  **configurable per-attempt timeout**. Status codes 429/503, command `npm test -- client.spec.ts`
  and path `src/http/client.ts` unchanged; "Nie testowano na produkcji" (no production check)
  preserved (rubric 5, 6). **Pass on Attempt 2.**

### S3 — Term already defined in visible context

- **Input fixture (Polish):** buffer size increased; earlier turn defined "bufor".
- **Actually run:** Yes (generated sample).
- **Observed output:**

> Odczyt dużych plików jest teraz szybszy — bufor odczytu zwiększono do 64 KB (`src/io/reader.ts`).
>
> Nie mierzono dokładnego przyspieszenia.

- **Rationale:** Outcome first (rubric 1). "bufor" **not** redefined (3, spec §3). Unmeasured
  speedup preserved (5). Path and value unchanged (6). **Pass.**

### S4 — A useful everyday analogy

- **Input fixture:** email-format validation added; tests pass; server-side validation unchanged.
- **Actually run:** Yes (generated sample).
- **Observed output:**

> Malformed email addresses are now rejected before the request is sent (`src/form/validate.ts`).
>
> Validation is the check that a value meets the rules required before the code continues. A useful
> comparison: validation is like a checkpoint at a door — it inspects the value before it proceeds.
> The limit: a format check confirms only that the address is well-formed; it cannot confirm that the
> mailbox exists or that the user owns it.
>
> Tests pass. Server-side validation was not changed.

- **Rationale:** Literal definition stands alone before the analogy (rubric 4). Analogy states the
  relevant similarity (values inspected before proceeding) and its limit (format only) (4, spec §3).
  "Server-side validation was not changed" preserved (5). **Pass.**

### S5 — A misleading analogy

- **Input fixture (Polish):** race condition fixed with a mutex; draft analogy claimed a queue
  "completely removes waiting".
- **Actually run:** Yes (generated sample).
- **Observed output:**

> Naprawiono warunek wyścigu za pomocą muteksu.
>
> Muteks blokuje dostęp do zasobu tak, że w danym momencie korzysta z niego tylko jedno zadanie.
> To szereguje dostęp — nie usuwa samego oczekiwania; kolejkowanie może je nawet wydłużyć.

- **Rationale:** The misleading analogy was removed (rubric 4, 5). Accurate mutex definition kept
  (resource locked by one task at a time; serializes, does not eliminate waiting) (spec §3).
  **Pass.**

### S6 — Failed check, risk and partial completion

- **Input fixture (Polish):** two of three tables migrated; `logs` failed; rows left behind; 1 error,
  2 passed.
- **Actually run:** Yes (generated sample).
- **Observed output:**

> Przeniesiono tylko część tabel: `users` i `orders` są zmigrowane, migracja `logs` nie powiodła się.
>
> Brakuje uprawnień do tabeli `logs`, więc jej wiersze pozostają w starej bazie. Test migracji:
> 1 błąd, 2 zaliczone.

- **Rationale:** No full-success claim; outcome stated honestly as partial (rubric 1, 5). Failed
  migration, missing permission, unmigrated rows and test count (1 error, 2 passed) all preserved
  (5). `logs`/`users`/`orders` names unchanged (6). **Pass.**

### S8 — Unrun tests, an exact command and path

- **Input fixture:** function added; tests written but not executed.
- **Actually run:** Yes (generated sample).
- **Observed output:**

> Added `parseDuration` in `src/time/parse.ts` for inputs like `"1h30m"`.
>
> Tests are in `src/time/parse.spec.ts`; to run them: `npm test -- parse.spec.ts`. I did not run the
> tests.

- **Rationale:** Outcome first (rubric 1). Command `npm test -- parse.spec.ts` and both paths
  reproduced exactly (6). Explicitly states tests were not run; no pass implied (5). **Pass.**

### S10 — Task results without a draft

- **Input fixture:** `loadConfig` refactored to return `undefined`; callers updated; tests pass.
- **Actually run:** Yes (generated sample).
- **Observed output:**

> `loadConfig` in `src/config.ts` now returns `undefined` instead of throwing when the file is
> missing. Callers were updated.
>
> Tests pass.

- **Rationale:** Summary produced directly from facts, no draft request, no editing report
  (rubric 7). Outcome first (1). Path unchanged; test fact preserved (5, 6). **Pass.**

### S11 — Code, documentation or quoted text (scope boundary)

- **Input fixture (Polish):** closing summary contained a quoted error string and a doc line, plus
  an editing report ("Skrócono tylko moje podsumowanie").
- **Actually run:** Yes (generated sample).
- **Observed output:**

> Logowanie działa poprawnie: błąd `"invalid token signature"` nie pojawia się już po odświeżeniu.
>
> Fragment dokumentacji: „Użytkownik pozostaje zalogowany przez 30 dni.”

- **Rationale:** Only the closing commentary was edited (rubric 7, spec §3). The quoted error string
  `"invalid token signature"` and the documentation sentence are **unchanged**. The editing report
  ("Skrócono tylko moje podsumowanie") was removed as scope leakage. **Pass.**

## Non-applicable / boundary cases

### S7 — Progress question or requested document (no format imposed)

- **Input fixture (Polish):** "Jak idzie?" mid-task; progress message, not a closing summary.
- **Actually run as a generated summary:** No — non-applicable.
- **Review type:** Reviewed boundary — expectation inspected, runtime not checked. The activation
  behavior was not exercised in a live OpenCode session; only the fixture's expected boundary was
  read against the spec.
- **Observed/expected behavior:** The skill is expected not to activate. The fixture progress message
  was left unchanged; no outcome-first/definition structure was imposed. This distinguishes "skill
  not activated" from "skill cannot answer" — it simply does not apply.
- **Second variant (requested document):** Not executed as a real deliverable here. Expected
  behavior (spec §7, §11): the requested document is produced as asked; the skill edits only the
  assistant's closing commentary, not the delivered artifact. Reported as a boundary description,
  not as an observed generated document.
- **Rationale:** Matches S7 requirements. **Pass (reviewed boundary), runtime not checked.**

### S9 — Explicit opt-out

- **Input fixture (Polish):** project instruction "Nie stosuj clearcraft w tym projekcie."; final
  summary follows.
- **Actually run as a generated summary:** No — opt-out, never applied.
- **Review type:** Reviewed boundary — expectation inspected, runtime not checked. The opt-out was
  not exercised in a live session; only the fixture's expected boundary was read against the spec.
- **Observed/expected behavior:** The editing style is expected **not** to be imposed. The fixture
  summary was left as written; independent obligations (test counts, pending manual test) remain
  intact. This is "not activated by choice", not "skill failed".
- **Rationale:** Matches S9 requirements. **Pass (reviewed boundary), runtime not checked.**

## Requirement-reviewed case (not executed)

### S12 — Skill unavailable

- **Input fixture (Polish):** scenario where the skill cannot be loaded.
- **Actually run:** No — **not executed**. No real runtime loading failure was produced; this
  fixture describes required behavior.
- **Review type:** Requirement reviewed — expectation inspected, runtime not checked. Not a "pass
  simulation": nothing was executed, so no pass is claimed.
- **Required behavior (not an observed runtime result):** Do not pretend the skill loaded; state the
  limitation honestly (the fixture text does this).
- **Rationale:** Described as required, not presented as an actual result. Distinguishes
  "unavailable" from "loaded but declined". **Not executed (requirement reviewed).**

## Summary table

Columns separate what was **observed** from what was **expected**. "Reviewed" rows were inspected
against the spec; they were not exercised at runtime.

| Case | Type | Observed (actual) | Expected (reviewed) | Status |
| --- | --- | --- | --- | --- |
| S1 | Generated sample | Output produced | Outcome first, no invented lesson | Pass (observed) |
| S2 | Generated sample (2 attempts) | Attempt 1 and Attempt 2 outputs produced | Attempt 1 missing jitter/timeout facts + loose backoff definition; Attempt 2 must state them | Attempt 1 **FAIL**; Attempt 2 Pass (observed) |
| S3 | Generated sample | Output produced | Do not redefine "bufor" | Pass (observed) |
| S4 | Generated sample | Output produced | Literal definition stands alone before analogy | Pass (observed) |
| S5 | Generated sample | Output produced | Misleading analogy removed, accurate definition kept | Pass (observed) |
| S6 | Generated sample | Output produced | No full-success claim; limitations preserved | Pass (observed) |
| S7 | Boundary (progress / document) | Not executed | No closing-summary formatting imposed | Reviewed boundary, runtime not checked |
| S8 | Generated sample | Output produced | Tests-not-run stated; command and path preserved | Pass (observed) |
| S9 | Boundary (opt-out) | Not executed | Editing style not imposed; obligations retained | Reviewed boundary, runtime not checked |
| S10 | Generated sample | Output produced | Summary produced, no draft request or editing report | Pass (observed) |
| S11 | Generated sample + scope boundary | Output produced | Only closing commentary edited; quotations unchanged | Pass (observed) |
| S12 | Requirement (unavailable skill) | Not executed | State the limitation honestly | Not executed (requirement reviewed) |

## Limits of this evaluation

- **Single pass, one context, self-assessed.** Each generated sample was produced by the same model
  instance with the skill text loaded. A separate lead-review pass then read the report against the
  fixture and found that the first S2 output was wrong (missing jitter/per-attempt-timeout facts and
  a loose exponential-backoff definition). S2 was rerun and corrected; Attempt 1 is kept above. This
  was a review pass over the report, **not** an independent model engine or external judge.
- **No runtime auto-discovery.** Discovery was not checked; do not read structural inspection as
  proof that OpenCode lists or loads the skill.
- **No build or tests.** The repository has no application build and no unit-test suite; nothing of
  that kind was run or claimed.
- **10 generated samples** (S2 counted twice: its failed Attempt 1 and corrected Attempt 2) are
  indicative only; they are not proof of reliability across models or sessions.

## Problems found in the skill / fixtures

No blocking issue was found in `SKILL.md`, `eval.md` or the spec. The one defect found during review
was in **this report's first S2 output**, not in the skill or fixtures: it dropped the added jitter
and the configurable per-attempt timeout and defined exponential backoff as plain lengthening
instead of exponential multiplication. That output is retained above as **Attempt 1 — FAIL** and a
corrected Attempt 2 was generated. Per the task constraints, no file other than this report was
modified; nothing here changes the skill, fixtures, README or git state.
