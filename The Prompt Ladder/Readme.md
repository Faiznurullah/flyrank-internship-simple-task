# The Prompt Ladder: Debugging a Laravel "Column not found" Error

**Track:** Backend (Laravel 12 / MySQL)
**Task:** Diagnose `SQLSTATE[42S22]: Column not found: 1054 Unknown column 'employees.sbu_id'`

> **Honesty note on the outputs.** The outputs below are *representative excerpts* showing the behavior each prompt version typically produces. Before submitting, re-run each prompt in a fresh chat with the same model and paste the real output over the excerpt. Keep the notes only if they still match what you see. If a real run behaves differently, rewrite the notes to match the real output.

---

## The Case File (shared by all versions)

Facts are only introduced into the prompt at the version that adds them. Replace these sample values with your real project data.

| Item | Value |
|---|---|
| Full error | `SQLSTATE[42S22]: Column not found: 1054 Unknown column 'employees.sbu_id' in 'where clause' (Connection: mysql, SQL: select count(*) as aggregate from employees where employees.sbu_id = 3 and employees.deleted_at is null)` |
| Where it happens | `SbuController@show`, line 38: `$sbu->employees()->paginate(15)` |
| Relationship | `Sbu::employees()` is `hasMany(Employee::class)`; `Employee::sbu()` is `belongsTo(Sbu::class)` |
| Migrations in repo | `2026_09_20_100000_add_sbu_id_to_employees_table.php` exists |
| Environment | Works on local. Fails on staging, right after the last deploy (`git pull`, `composer install`, no other steps recorded) |

---

## Ladder Overview

| Run | Layer added (exactly one) | Verdict |
|---|---|---|
| V0 | None (baseline) | Embarrassing |
| V1 | Clearer goal | Small gain |
| V2 | Real context | **Biggest gain** |
| V3 | Specified output format | **Mixed: partly worse** |
| V4 | Quality criteria | Clear gain |
| V5 | Verification requirements | Clear gain |

---

## V0: Baseline (the prompt I would have typed a month ago)

**Prompt**

```
Fix this error: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'employees.sbu_id' in 'where clause'
```

**Output (excerpt)**

> This error means the `employees` table doesn't have a column called `sbu_id`. To fix it, create a migration:
>
> ```
> php artisan make:migration add_sbu_id_to_employees_table
> ```
> ```php
> Schema::table('employees', function (Blueprint $table) {
>     $table->integer('sbu_id')->nullable();
> });
> ```
> Then run `php artisan migrate`.

**Notes**

- **What changed in the prompt:** Nothing. This is the starting point.
- **What the output did:** It jumped straight to a fix. It invented a migration with a guessed column type (`integer`, not a foreign key) and never asked whether this column already exists in the project.
- **What still failed:** There is no diagnosis. If a migration for `sbu_id` already exists, following this advice creates a duplicate.
- **What I'd try next:** State what I actually want (a diagnosis, not a patch).

---

## V1: Clearer Goal

**Layer added:** Clearer goal (the only change: say what outcome I want).

**Prompt**

```
Help me debug this Laravel error. Identify the most likely root cause, explain why it happens, and suggest the safest way to verify and fix it.

Error: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'employees.sbu_id' in 'where clause'
```

**Output (excerpt)**

> The most likely cause is that the database table does not contain `sbu_id`. Possible causes: (1) the migration hasn't been run, (2) it ran against a different database, (3) the column has a different name. Inspect the `employees` schema and compare with your migrations.

**Notes**

- **What changed in the prompt:** Added an explicit goal: root cause, reasoning, safe verification.
- **What improved in the output:** Compared with V0, it stopped writing a migration and started listing causes and a way to check them. The "invent a migration" behavior disappeared.
- **What still failed:** The causes are generic and could apply to any Laravel app. There is no ranking, and nothing says which cause is more likely for *my* situation.
- **What I'd try next:** Give it the real facts: the query, the relationship, the migration, and the local-vs-staging difference.

---

## V2: Real Context

**Layer added:** Real context (the only change: facts from the case file).

**Prompt**

```
Help me debug this Laravel error. Identify the most likely root cause, explain why it happens, and suggest the safest way to verify and fix it.

Context:
- Laravel 12, MySQL, HR management system.
- The error comes from SbuController@show line 38: $sbu->employees()->paginate(15)
- Sbu hasMany Employee; Employee belongsTo Sbu.
- A migration named 2026_09_20_100000_add_sbu_id_to_employees_table.php exists in the repo.
- Works on local. Fails on staging right after the last deploy (git pull + composer install).

Error: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'employees.sbu_id' in 'where clause' (SQL: select count(*) as aggregate from employees where employees.sbu_id = 3 and employees.deleted_at is null)
```

**Output (excerpt)**

> The `hasMany` relationship generates `employees.sbu_id` automatically, so the query is fine. Since the migration exists in the repo and works locally, the most likely cause is that the migration was never run on staging. Run `php artisan migrate:status` on staging. If it shows as Pending, run `php artisan migrate`.

**Notes**

- **What changed in the prompt:** Added the query location, relationship, existing migration, and the local/staging difference.
- **What improved in the output:** This was the biggest jump. It now ranks one cause first and gives a reason from my facts (the relationship auto-generates the column name; local works; the migration exists). It stopped giving generic possibilities.
- **What still failed:** It states "the migration was never run" as if it were established, but nobody has checked staging yet. It also says "just run `migrate`" on a shared staging database without mentioning that the migration might do more than add one column.
- **What I'd try next:** Give the answer a fixed structure so facts, guesses, and actions are not blended in prose.

---

## V3: Specified Output Format (this one did not clearly help)

**Layer added:** Output format (the only change: required sections).

**Prompt:** V2, plus:

```
Return your answer using exactly these sections:
1. Most Likely Root Cause
2. Evidence
3. Alternative Causes
4. Verification Steps
5. Recommended Fix
6. Risks
```

**Output (excerpt)**

> **1. Most Likely Root Cause:** The staging database is missing the `sbu_id` column because the migration was not applied.
> **2. Evidence:** The error says `Unknown column 'employees.sbu_id'`.
> **3. Alternative Causes:** Wrong database in `.env`; the column was renamed; a cached schema.
> **4. Verification Steps:** Check `migrate:status`; run `DESCRIBE employees`.
> **5. Recommended Fix:** Run `php artisan migrate`.
> **6. Risks:** Back up your database before migrating. Test in a non-production environment first.

**Notes**

- **What changed in the prompt:** Added six mandatory sections.
- **What improved in the output:** It is easier to scan. Verification steps and the fix are now separate, so I can jump to the part I need.
- **What still failed / got worse:**
  - **This version did not add insight.** The Evidence section only repeats the error message I pasted, and it ignores my stronger evidence (local works, migration exists).
  - **"Risks" is filler.** "Back up your database" is advice that applies to any change. The specific risk from V2 was lost: the migration may do more than add a column, and it runs on a shared DB.
  - **Forcing six sections made it invent content for sections with nothing to say.** The total answer got longer without getting smarter.
- **What I'd try next:** The format is not the weakness; the *quality bar* is. The model needs to be told what a good section contains, and that guesses must not be presented as facts.

---

## V4: Quality Criteria

**Layer added:** Quality criteria (the only change: a definition of "good").

**Prompt:** V3, plus:

```
Quality criteria:
- Label every claim as Confirmed (stated in my message), Inferred (reasoned from my message), or Unknown.
- Rank alternative causes by likelihood and state what evidence would rule each one out.
- The Evidence section must use facts from my context, not only the error message.
- Do not invent code, files, or schema details I did not provide.
```

**Output (excerpt)**

> **1. Most Likely Root Cause:** *Inferred:* the staging DB has not applied `2026_09_20_100000_add_sbu_id_to_employees_table`. *Confirmed:* the query is generated by the `hasMany` relationship, and the column is missing on the connected DB.
> **2. Evidence:** *Confirmed:* works on local, fails on staging after deploy; the migration file exists in the repo. *Unknown:* whether staging's `migrations` table contains this migration.
> **3. Alternative Causes (ranked):** (a) migration ran but staging `.env` points to another database (ruled out if `DB_DATABASE` matches the DB you inspected); (b) the migration failed partway (ruled out if `migrate:status` shows Ran and the column exists)...

**Notes**

- **What changed in the prompt:** Added a definition of a good answer: labeled claims, ranked causes, evidence from context, no inventing.
- **What improved in the output:** Confident-sounding guesses are now labeled "Inferred". The Evidence section finally uses my real facts. Each alternative cause now says what would rule it out. It no longer treats the migration as a proven fact.
- **What still failed:** Verification Steps are still a to-do list ("check migrate:status") with no expected results. I would still have to interpret what I see.
- **What I'd try next:** Require every verification step to say what result confirms or rejects each hypothesis.

---

## V5: Verification Requirements

**Layer added:** Verification requirements (the only change: how the answer must prove itself).

**Prompt:** V4, plus:

```
Verification requirements:
- For each verification step give the exact command, the result that would confirm the main hypothesis, and the result that would reject it.
- Before the Recommended Fix, state what evidence would change your diagnosis.
- Make the fix conditional on the verification results. If the information is insufficient to decide, say so instead of guessing.
```

**Output (excerpt)**

> **4. Verification Steps**
>
> | Step | Command | Confirms hypothesis | Rejects it |
> |---|---|---|---|
> | 1 | `php artisan migrate:status` (on staging) | The `add_sbu_id` migration shows **Pending** | It shows **Ran** |
> | 2 | `SHOW COLUMNS FROM employees LIKE 'sbu_id';` | Empty result | Column exists, so check `.env` DB name |
> | 3 | `php artisan migrate --pretend` | Shows only an `ALTER TABLE` adding `sbu_id` | Shows other pending changes, so review before running |
>
> **5. Recommended Fix (conditional):** *Only if steps 1 and 2 confirm,* run `php artisan migrate`. If step 3 shows other pending migrations, run them in a controlled order instead of all at once.

**Notes**

- **What changed in the prompt:** Added requirements for proof: commands, confirm/reject outcomes, a conditional fix.
- **What improved in the output:** The checklist became a decision table I can follow without interpreting anything. The fix is now gated on results, and the `--pretend` step addresses the shared-database risk that V2 and V3 missed.
- **What still failed:** "Risks" is still slightly generic. Details like `--pretend` behavior should be double-checked against the Laravel 12 docs, because the model can be confidently wrong about version details.
- **What I'd try next:** Add a "constraints" layer (for example, never run commands on production) or a review instruction, and test with a second error to check that the prompt generalizes.

---

## What Earned Its Place

| Layer | Contribution | Keep? |
|---|---|---|
| Clearer goal | Stopped the "write a migration" reflex | Yes, cheap and necessary |
| Real context | Turned generic causes into a ranked diagnosis | **Yes, the biggest win** |
| Output format | Easier to scan, but padded and no smarter | Only together with quality criteria |
| Quality criteria | Separated facts from guesses; evidence from context | **Yes** |
| Verification requirements | Turned advice into a testable decision procedure | **Yes** |

The main lesson: the format added nothing until the quality criteria told the model what each section had to contain.

---
