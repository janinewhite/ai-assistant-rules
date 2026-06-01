# Working Rules for an AI Project Assistant
*A small ruleset that governs how an AI assistant collaborates on project work — code, documentation, data, and the everyday administrative work that surrounds them*

*The multi-project version is at [centralized-ai-instructions](https://github.com/janinewhite/centralized-ai-instructions). This repo is the foundation pattern — start here if you're new.*

I work with an AI assistant on a mix of project work: data engineering, household healthcare administration, gardening planning, document writing, and the everyday code (Power Query M, DAX, Python, SQL, JavaScript) that supports those projects. The single highest-leverage thing I've built with that assistant isn't any one query or document — it's the working agreement we share, captured as a small set of rules in each project's tracker database.

This README is the current canonical version of that ruleset, in case it's useful to others working in similar conditions.

## Why this exists

An AI assistant rebuilds context from scratch every conversation. Anything you want it to remember about *how* to work — annotation style, error handling, what "done" means, naming conventions, how to negotiate trade-offs — has to live somewhere durable that the AI can re-read on its own. Putting that agreement in a queryable, version-controlled table turns "remember to..." into "the rule covers it," and it makes the working agreement auditable in the same way a code change is auditable.

The rules below evolve. Some get consolidated. Some get retired. New ones get added when I notice a gap. The point isn't to lock the agreement in — it's to make the agreement explicit enough to argue with.

## How it works

The rules live in an `instructions` table inside the project's tracker database (a SQLite file the AI can read directly). Each project I work on has its own tracker DB with its own copy of the table; the rules can diverge between projects as each project's domain demands.

The schema, as it currently stands:

```sql
CREATE TABLE instructions (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    title         TEXT NOT NULL,
    category      TEXT,
    instruction   TEXT NOT NULL,
    active        TEXT DEFAULT 'Yes' CHECK(active IN ('Yes', 'No')),
    active_when   TEXT,
    display_order INTEGER,
    date_added    TEXT,
    notes         TEXT
);
```

Two of the columns are recent additions worth calling out:

**`active_when`** — Optional context describing the condition under which a rule applies. Examples from the live ruleset: `"Writing code"`, `"Project includes Power BI development"`, `"Modifying the instructions table"`, `"Starting a task"`. A NULL value means the rule is unconditionally active. This lets the schema carry conditionality that previously had to live as wrapper framing in the AI's memory (e.g., "dormant pending PBI work"). It also makes it easy to teach the AI which rules to apply when: re-reading the table, the AI can match the `active_when` value against the current activity and skip rules that don't fit.

**`display_order`** — Integer that controls the order in which rules surface when the AI re-reads them at session start. The order isn't cosmetic — putting governance rules first ("re-read instructions", "DB is the source of truth", "don't change DB instructions without approval") means the AI encounters the foundational ones before the domain-specific ones, which catches mistakes earlier. Ties fall back to `id`; NULL falls back to `id`. In the current ruleset every active rule has an explicit `display_order`.

Two of the rules below describe the loop itself: the rule that tells the AI to re-read the table at every chat start and after any conversation compaction (foundational), and the rule that tells the AI not to change rows in the table without explicit approval (governance). Both belong at the top of the `display_order` so the AI encounters them first.

Rules are retired by setting `active='No'` rather than deleted, so the history of what we used to do remains readable.

## The ruleset

30 active rules across 9 categories. Within each category, rules are listed in the order they currently surface to the AI at session start.

---

### Process (governance and lifecycle)

**Instructions table is the source of truth**

The instructions table in the project tracker DB is the canonical source of truth for project workflow rules. Memory files (a working copy the AI builds in its own scratchpad) mirror the DB. When a workflow rule changes: (1) update the DB instructions row first, (2) update the corresponding memory file to match, (3) update the memory index if needed. If memory and DB ever conflict, the DB wins. Re-sync memory from DB at session start and after compaction.

**DB is the primary source of information**

The project tracker DB is the canonical source of information for the project. Architectural decisions, design specs, project context, vendor info, feature requirements, and process rules belong in the DB — typically in the `notes` table for free-form content, `scripts` for code, `instructions` for working rules, `items` for actionable Todos / Defects / Accomplishments, or the appropriate version table. Memory files act as pointers and working copies, not as the source of truth. When information appears in both memory and the DB and they diverge, the DB wins. When new project information emerges in conversation, capture it in the DB before (or instead of) memory.

**Do not change DB instructions without explicit approval** *(active_when: Modifying the instructions table)*

Instructions in the project tracker DB are canonical rules and must not be inserted, edited, deactivated, or schema-changed without explicit user approval. Propose changes — show the title, category, instruction text, and `active_when` value, plus a diff for edits — and wait for an approval reply before writing to the table. A previous "yes, that approach" answer authorizes the operation in principle; the specific text or values still need a sign-off. This applies to INSERTs of new rows, UPDATEs of any column, ALTER TABLEs of the schema, and DELETE / deactivation of existing instructions. Memory edits do not need approval; only DB writes to the instructions table.

**Instruction review format — Current / Proposed / Rationale / Apply when** *(active_when: Proposing an instruction change)*

When presenting a DB instruction change for approval, use this four-section structure:

1. **Current instruction** — the canonical approved text.
2. **Proposed instruction** — the full text the row will become.
3. **Rationale** — why the change is being proposed.
4. **Apply when** — the `active_when` value, or "Unconditionally active" if it will be NULL.

Present each row in prose, one at a time, and wait for the user's reply in text.

**Add tasks to the project tracker DB before starting work** *(active_when: Starting a task)*

Before beginning work on a task — anything that will produce a deliverable, modify a file, or take more than a couple of focused steps — verify it exists as a row in the `items` table of the project tracker DB. If it is not already there, add it: `type = Todo` (or `Defect`, if surfacing a bug), `status = Open` or `In Progress`, `date_entered = today`, and a clear description. Trivial single-step replies do not need a row; the threshold is "would a future session need to know this happened?"

**Task completion confirmation workflow**

After completing a task: (1) Verify the task exists in the DB (`items` table) — it should, per the pre-work-entry rule. If it is somehow missing, add it now and note the gap (this means the pre-work-entry rule was missed). (2) Ask the user to confirm the outcome with three options: Complete, Needs More Work, or Rollback. (3) Update the task record according to the user's response.

---

### Workflow (everyday cadence)

**Re-read instructions at chat start and after compaction**

At the start of every new chat in this project, and after any conversation compaction, read the instructions table in the project tracker DB and verify every active instruction is understood and reflected in working memory. Update memory files for any rule that has been added, edited, or deactivated. All active instructions are important and shape how we work together; none can be skipped or partially applied.

**Don't use the AskUserQuestion picker for detailed discussions**

When the AI client provides a multi-choice picker UI (e.g., Anthropic's AskUserQuestion in Claude Code / Cowork), use it only for short, well-bounded multiple-choice questions where the option set is obviously exhaustive ("which library?", "approve / reject?"). It is NOT appropriate for detailed back-and-forth review work where the user needs to consider context, trade-offs, or specific text changes — the picker constrains the conversation to pre-drafted options and forces "Other" for any nuanced response. Default to prose when in doubt.

**Announce tactic changes before continuing**

When the assistant describes an action that requires the user to participate (approve a download dialog, click a save prompt, paste a token, drag a tab in, etc.) and then finds a different way to accomplish the goal that does not need her input, it has to explicitly tell her it changed approach before continuing. "Continue" or similar short replies are consent to proceed on the path just described, not license to silently swap to a different one. Pure internal substitutions that do not alter what the user was going to do don't need announcement; only changes that remove or alter a user-facing step do.

**Log Accomplishments only after confirmed Complete**

Do not insert an Accomplishment row into the `items` table when the work finishes. Summarize the change, ask the user to confirm with Complete / Needs More Work / Rollback, and log the Accomplishment only after the user replies Complete. If the reply is Needs More Work, keep iterating without logging; any eventual Accomplishment covers the final, approved state. If the reply is Rollback, do not log an Accomplishment. Ask for clarification if completion is ambiguous rather than inferring. Defect rows, item-status changes, and version-table rows can still be logged as the work happens — only the Accomplishment entry in `items` waits for confirmation.

**Provide a file link after every save**

After saving any file in the Queries folder or any other deliverable, always include a `computer://` link at the end of the response so the file can be opened directly. Special case — tracker DB updates: when the change is to the project tracker DB, link to the project tracker HTML instead. The user views tracker data through the HTML page, not the .db file directly.

**Reference DB items by type and ID, not links**

When reporting inserts or updates to the project tracker DB, reference the affected row as `<Item Type> #<ID>` (e.g., `Defect #74`, `Accomplishment #101`, `Todo #73`). Do not provide a `computer://` link to the DB file in that context. File links for saved code deliverables remain per the file-link-after-every-save rule.

**Version every code change — file plus DB row, never overwrite** *(active_when: Writing code and output files)*

Whenever any computer-readable code or instructional file changes — for example .md, .html, .py, .sql, .json, .txt, .m, .dax, or any other text-based code or document — save the result as a new versioned file using the naming convention `[<associated_table>_]<name>_v<N>_<significant_change>.<ext>`. Add a row to the appropriate version table in the tracker DB (`scripts`, `data_items`, `query_versions`, `dax_measures`, `audit_queries`, or whichever schema applies). Never overwrite an existing versioned file.

**Snapshot tracker HTML before any edit**

The project tracker HTML files are code files in the versioning instruction's sense — every edit must produce a versioned snapshot. They live at the project root rather than in `Queries/`, but that does not exempt them. Before any edit, copy the live file to `Queries/tracker_html_snapshots/[project]_Tracker_v[N]_[change]_[date].html`. After the edit, save a post-edit snapshot with an incremented N. The same rule applies to any HTML file with embedded logic.

**Verify file integrity after every write**

After any write to a file under the workspace folder, verify the file is complete before treating it as ready to use. Syntax/AST parsing alone is insufficient — a truncation can land at a lexically valid point and parse cleanly while the file is functionally broken. Required checks per write: tail check (last 3–5 lines match expected ending), size check (line and byte count in expected range), and type-specific validity (Python AST + module import, SQL ends with `COMMIT;`, M ends with `in <identifier>`, JSON loads, HTML closing tags present). For substantial rewrites or files >100 lines: build the content in `/tmp/` first, verify, then copy into the destination.

**Cloud-synced file writes — verify after every save**

Files inside cloud-synced project folders (OneDrive, Dropbox, Google Drive sync, iCloud, etc.) can experience silent corruption through two related failure modes: (1) write operations occasionally truncate the trailing portion of a file without raising an error, and (2) the sync client can later overwrite a freshly committed file with a stale cloud version. Both modes affect code files AND binary files (SQLite databases, .docx, .xlsx, .pptx). After every write to a cloud-synced path, verify on disk (tail check, size check, type-specific validity). For SQLite `.db` writes specifically: copy the live DB to a local temp path, make the change there, sleep, copy back, sleep, then re-read from disk and confirm the new row count or `max(id)`. If the read shows the prior state, sync overwrote — retry.

**Record current dates in the user's local timezone** *(active_when: Writing a current date)*

Whenever a "current date" is written — to a DB column (`items.date_entered`, `items.date_completed`, `instructions.date_added`, any version-table date column), to a filename suffix (e.g., `..._snapshot_2026-05-17.html`), or inline in prose ("On 2026-05-17 I..."), use the date in the user's local timezone. The bash sandbox (and many AI execution environments) runs on UTC by default. To get the user's local date, run `TZ='<user's timezone>' date '+%Y-%m-%d'` — for example, `TZ='America/New_York'` or `TZ='Europe/London'` — or adjust the UTC clock by the appropriate offset. Do not record UTC dates by default — when the user is several timezones west of UTC, the UTC date is a day ahead of the user's local date for a portion of every 24-hour cycle, which creates confusion in cross-session recaps and date-based filters.

Hardcode the user's timezone in the rule body if the user isn't planning to move; otherwise carry it in a separate user-context memory and refer to it. Dates that refer to a fixed historical moment (e.g., "the v1 README was published 2025-12-04") keep whatever timezone they were originally recorded in; this rule covers *current* dates only.

**Check Queries folder before requesting code** *(active_when: Working with code)*

All queries and other code files live in the `Queries` folder of the project. Subfolders may be used to categorize files by language, role, or function. Before asking the user to paste code into the chat, check the Queries folder (and any subfolders) for the latest versioned file. If code is missing: (1) request it from the user, (2) create an annotated version and save it to the appropriate location, and (3) add it to the database.

**Update template prompt when tracker changes** *(active_when: Modifying the project tracker DB or HTML)*

When a change is made to the project tracker DB schema or HTML — new table, new column, new modal field type, new section, new report, new migration, new FAQ entry, new menu item, new styling convention — update the FAQ if new functionality was added and update the project's `Queries/Project_Tracker_Template_Complete.md` in the same session so future project trackers can be generated from the template in one pass. Treat the template update as part of the change, not a follow-up task. The template should enable the creation of the DB and HTML without any other sources.

**FAQ commands — full examples only, no '...' shorthand**

When writing FAQ entries or any instructional text where commands or paths are intended to be copy-pasted by the user, do not use "..." as a placeholder inside command or path examples. The example must be paste-ready and complete — every path concrete, every flag spelled out — unless the "..." token is itself valid language syntax (e.g., the JavaScript spread operator). Use shell-expandable tokens like `$HOME` or `~` instead of placeholders when the path is genuinely user-specific.

**Don't preserve instruction IDs across projects**

When copying the `instructions` table (or other free-form catalog content) from one project tracker DB to another, do not preserve the source IDs. Each project has its own ID space, and instructions get edited per project to be domain-specific over time — so cross-project ID alignment is not a goal worth optimizing for. Default behavior: insert without specifying `id` so SQLite auto-assigns contiguous values. Only preserve source IDs if the user explicitly asks. Catalog tables with referential meaning (lookups referenced by FKs) are different — those IDs should not be renumbered without checking dependents.

**Expose known issues for future reevaluation**

When a query, filter, or transformation identifies an issue (duplicate rows, orphan keys, suspect values, parameter-driven exclusions), surface the issue in the output via a flag column, audit table, or visible note rather than silently dropping or normalizing it. Pair the flag with enough context (reason, source, year coverage, etc.) for future reevaluation if conditions change. Do not, however, expand recursive searching to hunt for hypothetical issues that have not been identified — limit instrumentation to known issues, not speculative ones.

---

### Code Files

**Return full files, never snippets**

When discussing any improvement to a code, script, document, or markup file, return the complete file — not a snippet or targeted replacement. If multiple files are affected, return each one in full.

**Required file header — Name, Purpose, Summary, Recodes** *(active_when: Writing code)*

Every code or instructional file that supports comments must include an annotation header at the top, before any logic or content, with at minimum:

- **Name** — file, query, measure, or other identifier
- **Purpose** — what this file is written to achieve, and why it matters
- **Summary** — what this file accomplishes (general outputs and effects belong here)
- **Recodes** — significant data changes this file makes (omit if no significant data recoding occurs)

Inline comments through the body are welcome where they aid understanding but do not substitute for the header. Header style follows the language-specific conventions in the existing annotation rules.

**Annotation style — descriptive, not preventive** *(active_when: Writing code)*

Code annotations and comments describe what the code does, not what it fixes or prevents. Write in active, descriptive terms ("deduplicates on normalised key"). Never use phrases like "to avoid X", "preventing Y", "no X needed", or references to what a prior version did wrong. Code should read as correct by design. This applies to all code — Python, SQL, Power Query M, DAX, JavaScript, and any other language.

---

### Data Integrity

**Surface errors — never hide problems in code or analysis**

Never add defensive logic that masks, swallows, or normalizes values or behaviour that look wrong — neither in code (catching errors, replacing nulls, coercing types, filtering suspicious rows) nor in analysis (rationalizing the unexpected, smoothing over conflicts in the data, omitting outliers from a finding). Reveal potential inaccuracies or problems as soon as they're noticed. Before any change that catches an error, replaces a null, coerces a type, or filters out suspicious rows, ask whether the data is intentional. Default to preserving the anomaly and letting it surface — reliable, accurate results depend on visibility.

---

### Standards

**Logical column grouping in any tabular output**

Always preserve logical column grouping in any tabular output — medication lists, vitals logs, appointment tables, transport schedules, query results, report tables. Time dimensions together, geography together, people/demographics together, measures together, flags together. Never flatten or alphabetize column order arbitrarily.

---

### DAX (Power BI–specific)

**Annotation style — DAX** *(active_when: Project includes Power BI development)*

Use `/* */` block comments placed after the measure name and `=` sign. Never use `//`. Format:

```
Measure Name =
/*
Measure/Column Name: ...
Home Table: ...
Category: ...
Purpose: ...
*/
VAR ...
RETURN
    ...
```

The entire file must be pasteable directly into the Power BI measure editor. The right-arrow character (`→`) may not paste cleanly through some keyboards; prefer the ASCII `->` in comment text.

---

### Query Files (Power BI–specific)

**Parameter-aware queries** *(active_when: Project includes Power BI development)*

If the PBIX has parameters, every query must function across every valid combination of project parameters. Any parameter may be set to any value listed in its `parameters` table `valid_values` in any PBIX in this series, including combinations not yet exercised. When authoring or updating a query:

1. Detect column presence at runtime via `Table.ColumnNames` / `List.Contains` rather than assuming columns exist.
2. Emit a fully-typed empty fallback table when a parameter combination removes the inputs the populated branch reads.
3. Wrap conditionally-evaluated branches in functions whose body is only evaluated on invocation, when Power Query's branch type-inference might otherwise touch a branch that references absent columns.
4. Update parameter handling and column-presence guards in the same version as any change that introduces a new column or shifts which columns exist under which parameter combination.
5. Reference the `parameters` table `valid_values` when reasoning about which combinations the query must handle.

---

### Vocabulary

**Pumpkinned = ending the session**

"Pumpkinned" is my phrase for calling it a night (from Cinderella's coach turning back into a pumpkin at midnight). When asked what has happened since last pumpkinned, provide a concise cross-session recap of accomplishments and open items, drawn from the project tracker DB and memory.

---

## Adapting this for your own work

If you want to try a similar setup, the bones are:

1. **A small SQLite DB with the schema above.** One `instructions` table is enough to start; you can layer on `items`, version-history tables, `notes`, etc. later.
2. **A habit of asking the AI to read it at session start.** This is the rule that bootstraps every other rule.
3. **A habit of revising.** Most of the value comes from the conversations that change the rules, not the rules themselves.

The rules I've found highest-leverage:

- **A rule that tells the AI to re-read the rules.** Without it, every other rule is fragile.
- **A rule that gates the AI from editing the rules without your approval.** Otherwise it will "helpfully" expand or consolidate them while you're not looking, and the working agreement drifts.
- **A rule about confirmation.** Code you think is done isn't logged as Done until you say so. It is the difference between an Accomplishments log and an "I tried things" log.
- **A rule about pre-work entry.** Work that gets done without a row in `items` first is work that disappears between sessions. Putting the row in *before* doing the work makes the AI's behavior self-correcting.
- **A rule about annotation headers.** Every code file opens with Name, Purpose, Summary, Recodes. Combined with versioning, it produces a readable change history.
- **A rule about versioning.** Files are immutable; changes become new versioned files. Auditability and the ability to revisit useful algorithms that may have been inadvertently overwritten come from this kind of infrastructure.
- **A rule about verifying writes** when the workspace lives on a cloud-synced folder. Silent truncation and stale-version overwrites are common enough to plan for.

The rest builds on those.

## Where this goes next

After running this pattern across multiple projects, the per-project copy of the rules started chafing — improving a rule meant editing it in five places, wording drifted across copies, new rules I learned in one project sat unused in the others. So I built the next layer: one shared `Instructions.db` that every project reads from, with conditional applicability and an audit log. Same philosophy as this one (local SQLite, queryable, version-controlled, no cloud), with the rule duplication removed.

If one rule table per project starts feeling like duplication, that's the next step: **[centralized-ai-instructions](https://github.com/janinewhite/centralized-ai-instructions)**.

If you're still working out the basics, stay here — this is the foundation pattern, and most readers don't need v2 to get real value out of v1.

---

*This is a living document. The canonical source is the `instructions` table inside each project's tracker DB; this README is a snapshot exported for sharing.*
