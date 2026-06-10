# High-Signal Review Checklist

Concrete checks for the categories that automated PR reviewers (e.g. GitHub Copilot)
most frequently flag. These are the findings that generic "review for quality/security"
prompts miss because they are not operationalized. **Apply every category below to the
diff and report each hit as a finding** with file, line, severity, and a concrete fix.

These checks are language-agnostic in intent; examples use Ruby/Rails and TypeScript.

---

## A. Spec ↔ implementation consistency (highest-frequency miss)

Automated reviewers cross-reference the **PR description / commit messages / linked
issue** against the **actual diff**. The review MUST do the same. For every factual
claim in the description, find the corresponding code and confirm it is true.

- [ ] **"No DB / migration changes" but a migration exists** (or vice versa). Scan the
      diff for `db/migrate/**`, `schema.rb`, `*.sql` and reconcile with the description.
- [ ] **Stated definition ≠ coded definition.** e.g. description says "conversion =
      users with `activated_at`" but code filters on a different column/flag. Quote both
      and flag the mismatch; tell the author which side to fix.
- [ ] **Stated scope ≠ changed files.** Description claims one app/layer but the diff
      touches more (or fewer). Flag scope drift.
- [ ] **Stated behavior ≠ branch taken.** "falls back to X" / "blocks when Y" — trace
      the actual control flow and confirm.
- [ ] **Deploy/impact notes stale.** New index, new env var, new required column, data
      backfill — must be reflected in the description's impact/deploy section.

> If no PR exists yet, use the commit messages and the linked issue (Linear/Jira) as the
> source of truth and cross-check the same way.

## B. Enum / constant hardcoding & convention drift

Reviewers flag literals where the codebase has a typed accessor. Grep the diff for these.

- [ ] **Enum value as a bare literal** instead of the enum's value object / predicate.
      - Ruby/Enumerize: `status.to_sym == :guest` → `status.guest?`; comparison value
        `"guest"` → `User.status.guest.to_s`; spec `:round_robin` →
        `Model.column.round_robin`.
      - TS: `role === "admin"` where a `Role` enum/const exists → use the enum member.
- [ ] **Magic number / string** for a domain concept → named constant.
- [ ] **Reimplementing an existing helper / scope / predicate** instead of reusing it
      (`least_utilized?`, an existing query scope, an `escapeHtml` already in the file).
- [ ] **Pattern divergence from neighbors.** Same concept implemented differently than
      the sibling file 5 lines away (factories, scopes, error handling). Match the local
      idiom — project-specific patterns win over generic best practice.

## C. Nil / nullable / blank & input normalization

- [ ] **Nullable column used without normalization.** A column with no NOT NULL / no
      default reaches `.map`/`.size`/`.each` → `NoMethodError` on `nil`. Normalize with
      `Array(x)` / `x.to_a` / a default, and don't rebuild the array inside a loop.
- [ ] **`present?`-but-invalid.** A value can be present yet blank or malformed
      (whitespace, wrong case, unsafe chars) before being interpolated into a URL / host /
      SQL / filename. Normalize and validate (e.g. DNS-safe label, treat blank as nil)
      with a clearly-defined fallback.
- [ ] **Unbounded user input** used as array size, range, limit, or regex.

## D. Time & range boundaries

- [ ] **`Time.current` / `now()` where a day boundary is meant.** Intraday drift: use
      `end_of_day` / `beginning_of_day` to match a "from date .. to date" spec, and match
      the boundary style already used elsewhere in the same file.
- [ ] **Half-open vs inclusive range bug.** Filtering on `>= from` but ignoring the `to`
      upper bound (future-dated / backfilled rows leak in). Always bound both ends.
- [ ] **Timezone / UTC vs local** mismatch in comparisons and persisted values.
- [ ] **Off-by-one** on `<` vs `<=`, `length` vs `length - 1`.

## E. Concurrency & idempotency

- [ ] **`find_or_create_by!` / get-or-insert race.** Concurrent arrival → `RecordNotUnique`
      / unique-violation → 500. Rescue and re-fetch (mirror the existing pattern, e.g. how
      the Stripe/webhook path handles it), or rely on a DB unique constraint + upsert.
- [ ] **Idempotency record written before the work succeeds.** If the marker is created
      first and the handler then raises, the retry is skipped and the event is lost. Confirm
      "processed" only **after** success (status/`processed_at` flipped post-handler under
      lock).
- [ ] **Connection / resource leaked in a thread.** Work spawned in `Thread.new` that runs
      DB queries must check the connection back in (`with_connection`) and not let an
      exception kill the thread silently; define the failure result.

## F. DB indexes & query efficiency

- [ ] **New column used for filter / sort / count has no index.** Any `WHERE` / `ORDER BY`
      / `COUNT` on a newly added column (or a new composite predicate) → add the matching
      (composite) index in the migration, or justify its absence.
- [ ] **Load-then-filter in Ruby** what the DB could filter. Scope the query to the needed
      rows/types instead of loading all active rows and filtering in memory.
- [ ] **Redundant / duplicated query** per call (resolving the same value twice; a method
      that re-runs a query a caller already ran). Compute once, pass it down.
- [ ] **N+1** across the changed associations.

## G. Web / frontend security & accessibility

- [ ] **String-interpolated HTML → XSS.** Values put straight into an HTML string can break
      markup or inject script. Build with `createElement`/`textContent`, or use the existing
      `escapeHtml`; strictly validate/encode URLs.
- [ ] **`target="_blank"` without `rel="noopener noreferrer"`.**
- [ ] **Missing keyboard focus indicator.** New interactive/link styles define `:hover` but
      no `:focus-visible` / `:focus`.
- [ ] **Output encoding / SSRF / open-redirect** on any value derived from user input.

## H. Test rigor

- [ ] **Assertion doesn't prove the outcome.** A test that only checks "no error raised"
      (or a loose `be >= 1`) passes even when the feature is broken. Assert the concrete
      expected value and, for re-run/idempotency tests, assert the **second** call's result.
- [ ] **Repro diverges from production flow.** Test sets up state with a shortcut
      (`record.update!(version: 2)`) that the real code path never produces (the real path
      creates a new record). Drive it through the real service/flow.
- [ ] **Factory data doesn't satisfy the logic under test.** If the service identifies rows
      by attribute X, the factory must set X, or the example silently tests nothing.
- [ ] **Missing error / edge / boundary cases** alongside the happy path.

## I. Message / i18n ↔ logic consistency

- [ ] **User-facing message contradicts the code.** Error text says "confirmed
      reservations" but the guard checks "any non-terminal future reservation". Align the
      copy (and the test's expected string) with what the code actually enforces.
- [ ] **Locale key added/changed in one language only**, or referenced key missing.
