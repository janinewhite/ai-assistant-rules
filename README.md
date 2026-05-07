# Working Rules for an AI Coding Assistant
*A small ruleset that governs how an AI assistant collaborates on Power BI / data-engineering work*

I'm a Quality Engineer and Data Consultant working on Power BI projects with audit and compliance dimensions. I work with an AI assistant on the day-to-day code: Power Query M, DAX, Deneb / Vega-Lite specs, audit queries, supporting Python. The single highest-leverage thing I've built with that assistant isn't any one query — it's the working agreement we share, captured as a small set of rules in the project's tracker database.

This README is the current canonical version of that ruleset, in case it's useful to others working in similar conditions.

## Why this exists

An AI assistant rebuilds context from scratch every conversation. Anything you want it to remember about *how* to work — annotation style, error handling, what "done" means, naming conventions — has to live somewhere durable that the AI can re-read on its own. Putting that agreement in a queryable, version-controlled table turns "remember to..." into "the rule covers it," and it makes the working agreement auditable in the same way a code change is auditable.

The rules below evolve. Some get consolidated. Some get retired. New ones get added when I notice a gap. The point isn't to lock the agreement in — it's to make the agreement explicit enough to argue with.

## How it works

The rules live in an `instructions` table inside the project's tracker database (a SQLite file the AI can read directly). The schema:

```sql
CREATE TABLE instructions (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    title       TEXT NOT NULL,
    category    TEXT,
    instruction TEXT NOT NULL,
    active      TEXT DEFAULT 'Yes' CHECK(active IN ('Yes', 'No')),
    date_added  TEXT,
    notes       TEXT
);
```

Two of the rules below describe the loop itself: rule #8 (the AI re-reads the table at every chat start and after any conversation compaction) and rule #16 (when reporting changes to the tracker DB the AI cites rows as `<Type> #<ID>`, not file links — so changes are tied to the table they live in).

Rules are retired by setting `active='No'` rather than deleted, so the history of what we used to do remains readable.

## The ruleset

18 active rules across 7 categories.

---

### DAX

**#1 Annotation style — DAX**

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

The entire file must be pasteable directly into the Power BI measure editor.

*Note: the `→` arrow character may not paste cleanly — prefer `->` in comment text.*

---

### Data Integrity

**#2 Surface errors — never hide problems in code or analysis**

Never add defensive logic that masks, swallows, or normalizes values or behaviour that look wrong — neither in code (catching errors, replacing nulls, coercing types, filtering suspicious rows) nor in analysis (rationalizing the unexpected, smoothing over conflicts in the data, omitting outliers from a finding). Reveal potential inaccuracies or problems as soon as they're noticed. Before any change that catches an error, replaces a null, coerces a type, or filters out suspicious rows, ask whether the data is intentional. Default to preserving the anomaly and letting it surface — reliable, accurate results depend on visibility.

---

### Process

**#3 Task completion confirmation workflow**

After completing a task: (1) Verify the task exists in the DB (items table). If it is not there, add it. (2) Ask the user to confirm the outcome with three options: Complete, Needs More Work, or Rollback. (3) Update the task record according to the user's response.

---

### Query Files

**#4 Return full queries, never snippets**

When discussing any improvement to a Power Query M query, return the complete query — not a snippet or targeted replacement. Include the standard annotation header (Query Name, Purpose, Summary, Recodes) at the top. If multiple queries are affected, return each one in full.

**#5 Annotation style — Power Query M**

Annotations describe what the code does, not what it fixes or prevents. Write in active, descriptive terms ("deduplicates on normalised key"). Never use phrases like "to avoid X", "preventing Y", "no X needed", or references to what a prior version did wrong. Code should read as correct by design.

**#6 Logical column grouping in M queries**

Always preserve logical column grouping in Power Query output: time dimensions together, geography together, demographics together, measures together, flags together. Never flatten or alphabetize column order arbitrarily.

**#7 Parameter-aware queries**

If the PBIX has parameters, every query must function across every valid combination of project parameters. Any parameter may be set to any value listed in its parameters table `valid_values` in any PBIX in this series, including combinations not yet exercised. When authoring or updating a query:

1. Detect column presence at runtime via `Table.ColumnNames` / `List.Contains` rather than assuming columns exist.
2. Emit a fully-typed empty fallback table when a parameter combination removes the inputs the populated branch reads.
3. Wrap conditionally-evaluated branches in functions whose body is only evaluated on invocation when Power Query's branch type-inference might otherwise touch a branch that references absent columns.
4. Update parameter handling and column-presence guards in the same version as any change that introduces a new column or shifts which columns exist under which parameter combination.
5. Reference the `parameters` table `valid_values` when reasoning about which combinations the query must handle.

**#8 Required code header — Name, Purpose, Summary, Recodes**

Every code or instructional file that supports comments must include an annotation header at the top, before any logic or content, with at minimum:

- **Name** — query, measure, file name, or other identifier
- **Purpose** — what this code is written to achieve, and why it matters
- **Summary** — what this code accomplishes
- **Recodes** — what changes this code makes to the data

Inline comments through the body are welcome where they aid understanding, but they do not substitute for the header. Header style follows the language-specific conventions in the existing annotation rules (descriptive not preventive for M; `/* */` block placed after the measure name for DAX).

---

### Standards

**#9 Column ordering — query editor**

Use logical column grouping in the Power Query editor. Grouping is essential for review — columns that belong together conceptually must appear together, not in the order they arrived from the source.

---

### Vocabulary

**#10 Pumpkinned = ending the session**

"Pumpkinned" is my phrase for calling it a night (Cinderella's coach). When asked what has happened since last pumpkinned, provide a concise cross-session recap of accomplishments and open items.

---

### Workflow

**#11 Provide a file link after every save**

After saving any file in the Queries folder (.m, .json, .txt) or any other deliverable, always include a `computer://` link at the end of the response so the file can be opened directly.

**#12 Re-read instructions at chat start and after compaction**

At the start of every new chat in this project, and after any conversation compaction, read the instructions table in the project's tracker DB and verify every active instruction is understood and reflected in working memory. Update memory files for any rule that has been added, edited, or deactivated. All active instructions are important and shape how we work together; none can be skipped or partially applied.

**#13 Version every code change — file plus DB row, never overwrite**

Whenever any computer-readable code or instructional file changes — for example M, DAX, Deneb / Vega-Lite specs, JSON, Python, R, SQL, Markdown, or any other text-based code — save the result as a new versioned file using the naming convention `[<associated table>_]<name>_v<N>_<significant change>.<ext>`, where `<associated table>` is included when the file targets a specific table (e.g., a DAX measure that lives in a home table) and omitted when not applicable. Add a row to the appropriate version table in the tracker DB (`query_versions`, `dax_measures`, `audit_queries`, `scripts`, or whichever schema applies). Never overwrite an existing versioned file.

**#14 Check Queries folder before requesting code**

All queries and other code files live in the Queries folder of the project or its subfolders (DAX, Deneb, Audit Queries). Before asking the user to paste code into the chat, check the Queries folder for the latest versioned file. If code is missing from the Queries folder: (1) request it from the user, (2) create an annotated version and save it to the appropriate location in the Queries folder, and (3) add it to the database.

**#15 Reference DB items by type and ID, not links**

When reporting inserts or updates to the tracker DB, reference the affected row as `<Item Type> #<ID>` (e.g., `Defect #74`, `Accomplishment #101`, `Todo #73`). Do not provide a `computer://` link to the DB file in that context. File links for saved code deliverables (.m, .txt, .json, .py, .html, etc.) remain per the *Provide a file link after every save* instruction.

**#16 Update template prompt when tracker changes**

When a change is made to the Project Tracker DB schema or HTML — new table, new column, new modal field type, new section, new report, new migration, new FAQ entry, new menu item, new styling convention — update the FAQ if new functionality was added and update `Documentation/Project_Tracker_Template_Complete.md` in the same session so future project trackers can be generated from the template in one pass. Treat the template update as part of the change, not a follow-up task. The template should enable the creation of the DB and HTML without any other sources. One should be able to share the template alone and a new project tracker could be created without any associated example files or previous versions available.

**#17 Log Accomplishments only after confirmed Complete**

Do not insert an Accomplishment row into the items table when the work finishes. Summarize the change, ask the user to confirm with Complete / Needs More Work / Rollback, and log the Accomplishment only after the user replies Complete. If the reply is Needs More Work, keep iterating without logging; any eventual Accomplishment covers the final, approved state. If the reply is Rollback, do not log an Accomplishment. Ask for clarification if completion is ambiguous rather than inferring. Defect rows, item-status changes, and version-table rows (`query_versions`, `dax_measures`, `data_items`) can still be logged as the work happens — only the Accomplishment entry in `items` waits for confirmation.

**#18 Expose known issues for future reevaluation**

When a query, filter, or transformation identifies an issue (duplicate rows, orphan keys, suspect values, parameter-driven exclusions), surface the issue in the output via a flag column, audit table, or visible note rather than silently dropping or normalizing it. Pair the flag with enough context (reason, source, year coverage, etc.) for future reevaluation if conditions change. Do not, however, expand recursive searching to hunt for hypothetical issues that have not been identified — limit instrumentation to known issues, not speculative ones. Together with rule #5 (Surface errors), this keeps the data layer transparent without burning effort on imagined problems.

---

## Adapting this for your own work

If you want to try a similar setup, the bones are:

1. **A small SQLite DB with the schema above.** One table is enough to start; you can layer on items / version-history tables later.
2. **A habit of asking the AI to read it at session start.** This is the rule that bootstraps every other rule.
3. **A habit of revising.** Most of the value comes from the conversations that change the rules, not the rules themselves.

The rules I've found highest-leverage:

- **A rule that tells the AI to re-read the rules.** Without it, every other rule is fragile.
- **A rule about confirmation.** Code I think is done isn't logged as Done until I say so. It is the difference between an Accomplishments log and an "I tried things" log.
- **A rule about annotation headers.** Every code file opens with Name, Purpose, Summary, Recodes. Combined with versioning, it produces a readable change history.
- **A rule about versioning.** Files are immutable; changes become new versioned files. Auditability and the ability to revisit useful algorithms that may have been inadvertantly overwritten come from this kind of infrastructure.

The rest builds on those.

---

*This is a living document. The canonical source is the `instructions` table inside the project's tracker DB; this README is a snapshot exported for sharing.*
