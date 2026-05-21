# Working Rules for AI Collaboration

This file is a portable snapshot of the working rules I've established for collaborating with AI agents across my projects. Each rule tells you (the assistant) how I prefer to be worked with -- what to track, how to format code I have to paste, when to ask for confirmation, what NOT to do, etc.

The file is split into two sections:

- **Section 1 -- Universal Rules** apply on any project. Where the rule involves task tracking, it uses the in-session **TodoList** tool (`TaskCreate` / `TaskUpdate` / `TaskList`) as the progress mechanism. If the project has a persistent record store (DB, items log, etc.) alongside the TodoList, the rules also recommend mirroring there.
- **Section 2 -- Project-Specific Rules** are from my DDRC Power BI work. They reference Power Query M / DAX / parameterized PBIX series / CDC WONDER data conventions. Reuse them in another project only after adapting the domain references -- they're listed here as examples of how a project extends the universal ruleset with its own conventions.

Conventions:
- **Apply when:** says when the rule is active. "Unconditionally active" means it applies on every interaction.
- Rules don't cross-reference each other; each is written to stand alone.
- The ordering reflects how I see the rules in my project tracker, not a dependency chain.

Snapshot date: 2026-05-19. 25 universal rules, 8 project-specific rules.

---

# Section 1 -- Universal Rules

## 1. Re-read instructions at chat start and after compaction

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

Re-read this file at the start of every chat AND after any conversation compaction. The TodoList captures in-session progress; this file carries the standing rules. Without a re-read, the compaction summary may have silently dropped rules from your working context.

**Notes:**

Compaction can drop context. The DB is the authoritative source for instructions; memory is the working copy.

---

## 2. Instructions table is the source of truth

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

This Markdown file is the source of truth for the working rules. If the project later builds a persistent rules registry (DB, config file, etc.), the persistent registry becomes authoritative and this file is reduced to a snapshot. Until then, the working copy of any rule you keep in memory must match this file verbatim; if you notice drift, the file wins.

**Notes:**

Establishes the rule that gives the post-compaction review instructions (#8 / #13) their meaning. Added 2026-05-11.

---

## 3. DB is the primary source of information

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

Capture durable project information (architecture decisions, vendor info, feature requirements, process rules, open issues) in whichever persistence the project supports -- a tracker DB, a /docs folder, a wiki, the user's chosen system. If the project has not chosen one yet, raise the question with the user; do not invent storage. The in-session TodoList is for ephemeral progress, not durable context.

**Notes:**

Generalizes the instructions-table source-of-truth principle to all project information. Memory files are intended as fast indexes / working copies of canonical DB content.

---

## 4. Do not change DB instructions without explicit approval

**Category:** Process

**Apply when:** Modifying the instructions table

**Rule:**

Do not insert, edit, deactivate, or delete any rule in this file without explicit user approval. Propose the change (show title, category, full proposed text, rationale, and apply-when value) and wait for an approval reply before editing the file. A previous "yes, that approach" answer authorizes the operation in principle; the specific text still needs sign-off.

---

## 5. Instruction review format

**Category:** Process

**Apply when:** Proposing an instruction change

**Rule:**

When presenting a rule change for approval, use this four-section structure:

1. **Current rule** -- the text as it stands today (or "N/A (new rule)" if you are adding one).
2. **Proposed rule** -- the full text the rule will become.
3. **Rationale** -- why the change is being proposed: what gap it closes, what incident prompted it.
4. **Apply when** -- either a specific condition or "Unconditionally active".

Present each rule in prose, one at a time, and wait for the user's text reply.

---

## 6. Don't use the AskUserQuestion picker for detailed discussions

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

The AskUserQuestion picker interface is appropriate for short, well-bounded multiple-choice questions where the option set is obviously exhaustive (e.g., "which library?", "approve / reject?"). It is NOT appropriate for detailed back-and-forth review work where the user needs to consider context, trade-offs, or specific text changes — the picker constrains the conversation to pre-drafted options and forces "Other" for any nuanced response, which is friction.

**How to apply:**
- Multi-row review work (e.g., approving DB edits one row at a time): present each item in prose, ask the question in text, let the user reply naturally.
- Trade-off discussions where options interact: same — prose, not picker.
- Quick, clean either-or with no negotiation room: picker is fine.
- Default to prose when in doubt; the picker should be the special case, not the reflex.

---

## 7. Announce tactic changes before continuing

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

When I describe an action that requires the user to participate (approve a download dialog, click a save prompt, paste a token, drag a tab in, etc.) and then I find a different way to accomplish the goal that does not need her input, I have to explicitly tell her I changed approach before continuing. "Continue" or similar short replies are consent to proceed on the path I just described, not license to silently swap to a different one. Pure internal substitutions that do not alter what the user was going to do (a different bash command, a different parser, a different intermediate file) do not need announcement; only changes that remove or alter a user-facing step do.

**Why this matters:** Silent path swaps leave the user waiting for prompts that never come, or assuming a step happened that didn't. Both are time-wasters and trust-eroders.

**How to apply:** If the user was going to do anything on the original plan (approve, click, paste, drag, type) and I move to a path that skips that step, surface the change in the next response ("Skipping the X step -- using Y instead. Status: ...") before doing the new work.

---

## 8. Add tasks to the project tracker DB before starting work

**Category:** Process

**Apply when:** Starting a task

**Rule:**

Before beginning any task -- anything that will produce a deliverable, modify a file, or take more than a couple of focused steps -- verify it exists in the TodoList. If it is not there, add it via TaskCreate before touching the work. Update status as you progress (TaskUpdate to in_progress when starting, completed when done). Trivial single-step replies do not need a task; the threshold is "would a future session need to know this happened?"

---

## 9. Task completion confirmation workflow

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

After completing a task: (1) verify the task exists in the TodoList; if not, add it. (2) Ask the user to confirm the outcome with three options: Complete, Needs More Work, or Rollback. (3) Update the TodoList according to the user's response. The status only flips to completed on an explicit user signal, not on your own assumption that the work is done. If the project has a persistent record of work alongside the TodoList, mirror the confirmed completion there too.

---

## 11. Always include a file link with every mention

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

Include a computer:// link inline with every mention of a file — not just the initial save, but every time you reference it (lists, recaps, retroactive corrections, prose). The link goes inline with the filename: e.g., `[MortalityModel_v20_HelperReference.m](computer://...)`. Applies to all code, data, document, and markup files. **Tracker HTML special case:** when saving the tracker HTML, tell the user to reload the project (File → Open Project Folder) because the HTML caches the previous DB read. **Exception:** files larger than ~100 KB or containing embedded TopoJSON (e.g., CountyMap, CountyMapBase, CountyMapDerivative specs) freeze the chat UI when their link is clicked — clicking forces a full Cowork restart. Reference these by full Windows path as plain text instead, and note 'open from Windows Explorer'.

**Why:** efficient verification of saves; the user opens files immediately rather than navigating the filesystem. **How:** every file mention gets a computer:// link with the filename as link text, unless the file falls under the size / TopoJSON exception above.

**Notes:**

Broadened 2026-05-11 from "after saving" to "every mention". User needs the link present every time the file is referenced so they can click and open it to copy/paste, even on re-emissions and status recaps.

---

## 12. Reference DB items by type and ID, not links

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

Reference TodoList tasks by their numeric ID -- e.g., "task #14". Don't embed clickable links to internal task UIs because they generally don't resolve from chat. Files larger than ~100 KB or containing embedded TopoJSON / large binary blobs may freeze the chat UI when their link is clicked; reference those by full plain-text path instead and note "open from your file explorer".

---

## 13. Return full files, never snippets

**Category:** Code Files

**Apply when:** Writing code

**Rule:**

When providing code changes, return the complete file — never a diff, snippet, or instruction to "add this line." The user pastes the whole file into the target editor (Power Query Editor, DAX editor, script editor, document tool, etc.). This applies to any file type the user pastes: code, script, document, or markup.

**Notes:**

User needs paste-ready code. Partial edits require mental reconstruction the user cannot rely on.

---

## 14. List multi-file deliverables in application order

**Category:** Code Files

**Apply when:** Unconditionally active

**Rule:**

When delivering more than one file in a single response (M queries, DAX measures, helpers, Deneb specs, or any combination), order the listed files by the sequence in which they should be applied -- upstream prerequisites first, downstream consumers last. The user pastes top-to-bottom; the order in the response IS the implicit instruction for the paste order, so the list must reflect dependency direction.

**How to apply:**
- Upstream files (no dependency on other listed files) appear first; files that reference them appear later in the list.
- Examples: PopulationFillSource helper before MortalityModel (which references the helper); DimState / DimDivision / DimRegion before MortalitySource (which joins to them); DimColumns before Mortality / MortalityDynamics that read its contract; prerequisite DAX measures before measures that reference them.
- Never list a main file before its helpers, even if the main file is the "headline" of the change.
- If the user is expected to disable a Refresh or toggle a setting mid-cascade, place that step inline at its correct position in the sequence, not before or after the list.
- If the order genuinely does not matter (independent files with no cross-references), say so explicitly so the user can paste in any order without guessing.
- The user runs Close & Apply once at the END of the cascade, not per file (per the PBIX query propagation workflow). Sequence in the response matters because partial-state previews can error before the cascade is complete.

---

## 15. Required file header

**Category:** Code Files

**Apply when:** Writing code

**Rule:**

Every code or instructional file that supports comments must include an annotation header at the top, before any logic or content, with at minimum:
• Name — file, query, measure, or other identifier
• Purpose — what this file is written to achieve, and why it matters
• Summary — what this file accomplishes (general outputs and effects belong here)
• Recodes — significant data changes this file makes (omit if no significant data recoding occurs)

Inline comments through the body are welcome where they aid understanding, but they do not substitute for the header. The header uses descriptive wording (not preventive) for code generally, and /* */ block style placed after the measure name for DAX.

**Why:** The header is the entry point for review — it tells a reader what the file is, why it exists, what it does, and what data effects it has, before they read a single line of logic.

**How to apply:** Every code/instructional file gets a header. "Recodes" is reserved for significant data changes; general effects belong in Summary.

**Notes:**

The header is the entry point for review — combined with versioning, it produces the auditable, readable change history.

---

## 16. Version every code change — file plus DB row, never overwrite

**Category:** Code Files

**Apply when:** Writing code and output files

**Rule:**

Every code change goes into a new versioned file alongside the previous version -- never overwrite an existing version. Naming: `[<associated_table>_]<name>_v<N>_<significant_change>.<ext>` -- the table prefix is optional and used when the file belongs to a specific data table (e.g., `Mortality_MapTitle_v7_BaseMetricCrossProduct.txt`); otherwise the name starts with the file's own identifier. Log the new version as a TodoList task or, if the project has a persistent version log, write a row there too. Exception: if a versioned file has not yet been accepted by the user or adopted into the project's live system, in-place edits to that file are acceptable -- no new version needed until the file is agreed to or committed.

**Notes:**

Versioned files + DB log together are the authoritative change history for the project. In-place-before-adoption clarification added 2026-05-11 after Map Title v6 / Comparison Map Title v4 comment-correction case where the user had not yet pasted either file.

---

## 18. Verify file integrity after every write

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

After any Write, Edit, or cp that modifies a file under the workspace folder (especially OneDrive-synced paths like `C:\Users\janin\OneDrive\Desktop\DDRC\DDRC\...`), verify the file is complete before treating it as ready to use. Two failure modes apply: (1) the write itself may silently truncate the file's tail despite returning success, and (2) OneDrive sync can later overwrite a freshly committed file with a stale cloud version. Both modes affect code files (HTML, Python, SQL, M, JSON, DAX) AND binary files (.db SQLite databases, .docx, .xlsx, .pptx). Syntax/AST parsing alone is insufficient — a truncation can land at a lexically valid point and parse cleanly while the file is functionally broken.

Required checks per write:
1. **Tail check.** Read the last 3–5 lines and confirm they match the expected ending of the content (closing tag, return statement, `COMMIT;`, expected last function definition, etc.).
2. **Size check.** Confirm line count and byte count are in the expected range. A snapshot smaller than the prior version while only adding content is a red flag.
3. **Type-specific validity:**
   - **Python:** `python -c "import ast; ast.parse(open('<path>').read())"` AND `python -c "import <module>"` to catch runtime breakage AST does not see.
   - **SQL:** file must end with `COMMIT;` or whichever explicit terminator the migration uses.
   - **Power Query M (.m):** file must end with `in <identifier>`. A truncated M file commonly ends mid-let-binding or mid-expression with no `in`.
   - **JSON:** file must end with `}` (or `]` for array-rooted files); validate with `python -c "import json; json.load(open('<path>'))"`.
   - **HTML / Jinja templates:** file must end with the closing `{% endblock %}` and `</html>` (where applicable).
   - **SQLite .db:** open the file and confirm the new `max(id)` or row count matches the write. If the post-write read shows the prior state, OneDrive sync overwrote the change — re-execute the change rather than retrying the read.
   - **Markdown / docx / pptx / xlsx:** tail-check + size-check; for binary formats, open the file with the appropriate library and confirm structure.
4. For substantial rewrites or files larger than ~100 lines: build the content in `/tmp/` via bash first, verify there, then `cp` to the destination path via bash.
5. After any `cp` to a OneDrive-synced path: `sleep 1` (or 2+ before, then 3 after, for SQLite .db writes), then re-verify on disk — sync timing can race the next operation.
6. If a write appears to have succeeded but a later verification shows the change missing, treat it as an OneDrive sync overwrite — re-execute the change rather than assuming user error or a transient I/O issue.

If any check fails, reconstruct from `/tmp/` or from the most recent verified-intact source. Don't trust "the file looks long enough" — verify the tail.

**Notes:**

Combines two related guards: write-time truncation (file lands incomplete despite a success return) and OneDrive sync overwrite (a complete write gets clobbered later by a stale cloud version). Both failure modes have caused silent data loss; both require explicit verification.

---

## 22. FAQ commands — full examples only, no '...' shorthand

**Category:** Documentation

**Apply when:** Unconditionally active

**Rule:**

When writing FAQ entries or any instructional text where commands or paths are intended to be copy-pasted by the user, do not use "..." as a placeholder inside command or path examples. The example must be paste-ready and complete — every path concrete, every flag spelled out — unless the "..." token is itself valid language syntax (e.g., the JavaScript spread operator in displayed code, which is real syntax, not shorthand).

**How to apply:**
- Any `<code>` block or other paste-target command: use a concrete path. If the path differs by user (e.g., home directory), use real shell-expandable tokens such as `$HOME` or `~` (which DO work when pasted) rather than "...".
- Prose references to command forms ("use the python -m uvicorn pattern") should also avoid "..." — spell out the full command or use descriptive phrasing without the ellipsis.
- Spread operators (`...arr`) and other genuine language syntax are fine; that is not shorthand, that is the actual code.
- If the FAQ targets multiple OSes and one OS path cannot be made fully concrete without knowing the user's username, either use a shell-expandable token like `$HOME`, or drop that OS variant if the project is single-OS anyway.

---

## 24. Annotation style — descriptive, not preventive

**Category:** Code Files

**Apply when:** Writing code

**Rule:**

Write annotations that describe what the code does, not what it fixes or prevents. Comments should read as if the code is correct by design. Don't write `// fixed null bug in 2024-08`; write `// returns null when the field is absent`. Bad: `// avoid divide-by-zero`. Good: `// returns null when denominator is zero`. Applies to all code — Python, SQL, Power Query M, DAX, JavaScript, and any other language.

---

## 27. Surface errors — never hide bad data

**Category:** Code Files

**Apply when:** Writing code

**Rule:**

Never add defensive logic that masks, swallows, or normalizes values that look wrong in code or in analysis. If a value or pattern looks off, surface it and ask whether it's intentional before changing or filtering it. **Why:** masking anomalies hides bugs in upstream data, in the code itself, or in the user's intent — all three are signals the user needs to see. **How:** preserve the anomaly in the output, flag it explicitly, and ask before normalizing.

**Notes:**

CDC-compliant analysis — silently normalized bad values corrupt downstream results.

---

## 28. Expose known issues for future reevaluation

**Category:** Data Integrity

**Apply when:** Writing code

**Rule:**

When a piece of data or logic looks wrong but the root cause isn't yet clear, leave a flag column or a reason text marker in the output instead of silently dropping or fixing the row. The marker should be inspectable later. Don't expand recursive searches looking for hypothetical issues — flag what you see and move on. **Why:** silent drops hide bugs; recursive hunting for hypothetical problems wastes effort and obscures what's actually broken. **How:** add a column or comment that names the issue, leave the row in place, and surface it in the next review.

**Notes:**

Examples: MortalitySourceFileInventory v4 keeps duplicate multi-year files in the output with a Duplicate flag and DuplicateReason text rather than filtering them out before they appear in the inventory; MortalitySuppressed v20 carries Suppressed and FullySuppressedCounty as flags rather than dropping the rows; MortalitySourceDuplicates audit query exposes overlap rows for inspection rather than auto-deduping at MortalitySource.

---

## 29. Logical column grouping in any tabular output

**Category:** Code Files

**Apply when:** Writing code

**Rule:**

Order columns in any tabular output by logical grouping (e.g., all keys together, then all measures, then all flags) so the user can scan tables without horizontal scrolling. Don't leave columns in the order they happen to be derived. Applies to Power Query results, DAX-calculated tables, Deneb-bound datasets, audit-query output, and any other tabular surface. The user has a memory disability and relies on logical column order for review.

**Notes:**

User has a memory disability. Consistent column grouping is essential for accurate data review.

---

## 31. Pumpkinned = ending the session

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

"Pumpkinned" is the user's phrase for calling it a night (Cinderella's coach). When asked what has happened since last pumpkinned, provide a concise cross-session recap of accomplishments and open items, drawn from the project tracker DB and memory.

---

## 32. Record current dates in the user's local timezone (UTC-7, Arizona)

**Category:** Workflow

**Apply when:** Writing a current date

**Rule:**

Whenever a "current date" is written — to a DB column (`items.date_entered`, `items.date_completed`, `instructions.date_added`, any version-table date column), to a filename suffix (e.g., `..._snapshot_2026-05-17.html`), or inline in prose ("On 2026-05-17 I…") — use the date in the user's local timezone: UTC-7 (Arizona, year-round; no DST observed). The bash sandbox clock runs in UTC. To get the user's local date, run `TZ='America/Phoenix' date '+%Y-%m-%d'` or subtract 7 hours from the UTC clock. Do not record UTC dates by default — in the 7-hour window after Arizona midnight (07:00 UTC through 06:59 UTC the next day), the UTC date is a day ahead of the user's local date, which would create confusion in cross-session recaps and date-based filters.

**Why:** The user reviews tracker dates against her own local calendar. UTC dates would drift forward by up to a day during late-evening sessions, making "today's work" land on the wrong row in retrospect.

**How to apply:** Before writing any current date — to a row, a filename, or chat prose — run `TZ='America/Phoenix' date '+%Y-%m-%d'` to compute the correct date. Use that value, not the bare date output. Dates that refer to a fixed historical moment keep whatever timezone they were originally recorded in; this rule covers current dates only.

---

## 33. Items table — confirmation required before marking Done

**Category:** Process

**Apply when:** Unconditionally active

**Rule:**

TodoList tasks (and any equivalent persistent items) require explicit user confirmation before being marked completed. Workflow: (1) Bug-style or defect items can be added mid-fix as in_progress -- no confirmation needed to log the existence of an issue, but the flip to completed waits. (2) For accomplishments / deliverable summaries: do not pre-emptively mark them done. Summarize the change, ask the user to confirm with Complete / Needs More Work / Rollback. (3) On Complete / Done / Working: update the related task and (if the project tracks accomplishments persistently) write the accomplishment row. (4) On Needs More Work: keep iterating; do not mark done. (5) On Rollback: revert; do not log an accomplishment. (6) Ask for clarification if completion is ambiguous rather than inferring.

**Notes:**

Generalizes prior version (Accomplishment-only). Updated 2026-05-11 after user surfaced premature Done marks on Defects #201 and #202.

---

## 37. Don't initiate suggestions to sleep, rest, or pumpkin

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

Never tell the user to sleep, rest, "pumpkin," call it a night, wrap up for the night, take a break, or otherwise suggest they stop working. The user decides when they are done. This applies even when the work appears to be at a natural stopping point and even when the user has been working for many hours; projecting onto their schedule reads as condescending. Do not write phrases like "sleep well," "pumpkin time," "see you tomorrow," "you've earned a break," or any variation. Acknowledging when the user initiates such language is fine -- if the user says "I'm pumpkinning" or "I'm done for the night," the trigger is theirs, and a tomorrow plan is appropriate at that point. If the immediate work appears complete, hand the next-step decision back to the user (e.g., "the immediate work is done; let me know what's next") rather than suggesting a stop.

---

# Section 2 -- Project-Specific Rules (DDRC Power BI)

## 17. Snapshot tracker HTML before any edit

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

The project tracker HTML files (e.g., `DDRC_Project_Tracker.html` at the DDRC project root) are code files in the versioning instruction's sense — every edit must produce a versioned snapshot. They live at the project root rather than in `Queries/`, but that does not exempt them. Before any edit, copy the live file to `Documentation/Backup/[project]_Tracker_v[N]_[change]_[date].html`. After the edit, save a post-edit snapshot with an incremented N. Create the snapshots folder if it does not yet exist. The same rule applies to any HTML file with embedded logic, not just the project trackers — if it is a code file with embedded JavaScript or data, snapshot before changing.

---

## 20. Check Queries folder before requesting code

**Category:** Workflow

**Apply when:** Working with code

**Rule:**

All project source files live under `Queries/` and its subfolders. Subfolders may be created or renamed as the project's languages and needs evolve, so check the current folder layout rather than assuming a fixed list. Before asking the user to paste code into chat, check `Queries/` for the file. Reference its path so the user can open it directly.

**Notes:**

Queries/DAX for DAX measures (.txt), Queries/Deneb for Vega-Lite specs (.json), Queries/ root for M queries (.m), Queries/Audit Queries for audit M queries.

---

## 21. Update template prompt when tracker changes

**Category:** Workflow

**Apply when:** Modifying the project tracker DB or HTML

**Rule:**

When a change is made to the project tracker DB schema or HTML — new table, new column, new modal field type, new section, new report, new migration, new FAQ entry, new menu item, new styling convention — update the FAQ if new functionality was added and update Documentation/Project_Tracker_Template_Prompt.md in the same session so future project trackers can be generated from the template in one pass. Treat the template update as part of the change, not a follow-up task. The template should enable the creation of the DB and HTML without any other sources. One should be able to share the template alone and a new project tracker could be created without any associated example files or previous versions available.

**Why:** Self-sufficiency of the template is the point. If the template drifts behind the live tracker, future tracker generation requires hunting through example files.

**How to apply:** As soon as a tracker DB/HTML change is made, edit the template in the same response. If the change adds new functionality, update the FAQ as well.

---

## 23. Don't preserve instruction IDs across projects

**Category:** Workflow

**Apply when:** Unconditionally active

**Rule:**

When copying the instructions table (or other free-form catalog content like notes or items) from one project tracker DB to another, do not preserve the source IDs. Each project has its own ID space, and instructions get edited per project to be domain-specific over time — so cross-project ID alignment is not a goal worth optimizing for. Default behavior when copying instructions into a new project: insert without specifying id so SQLite auto-assigns contiguous 1..N values. Only preserve source IDs if the user explicitly asks for it. This applies to free-form rows generally; catalog tables with referential meaning are different — those IDs are referenced by foreign keys and should not be renumbered without checking dependents.

---

## 25. Annotation style — DAX

**Category:** DAX

**Apply when:** Project includes Power BI development

**Rule:**

Use `/* */` block comments placed after the measure name and `=` sign. Inline `//` comments inside the body are fine for short notes. The entire file must be pasteable directly into the Power BI measure editor. Avoid right-arrow characters (`→`) in code or comments — Power BI's text editor sometimes silently rejects pastes containing them.

**Notes:**

The → arrow character may not paste cleanly — prefer -> in comment text.

---

## 34. Parameter-aware queries

**Category:** Code Files

**Apply when:** Unconditionally active

**Rule:**

Every query must function across every valid combination of project parameters. Any parameter (pUseLocal, pTimePeriod, pGeographyLevel, pHasDemographics, pDrugs, pDDRCStates, pSharePointSite, pDocumentsSubfolder, pSharePointCensusFolder, pLocalFolder, pLocalCensusFolder) may be set to any value listed in its parameters table valid_values in any PBIX in this series, including combinations not yet exercised. When authoring or updating a query: (1) detect column presence at runtime via Table.ColumnNames / List.Contains rather than assuming columns exist; (2) emit a fully-typed empty fallback table when a parameter combination removes the inputs the populated branch reads; (3) wrap conditionally-evaluated branches in functions whose body is only evaluated on invocation when Power Query's branch type-inference might otherwise touch a branch that references absent columns; (4) update parameter handling and column-presence guards in the same version as any change that introduces a new column or shifts which columns exist under which parameter combination; (5) reference the parameters table valid_values when reasoning about which combinations the query must handle.

**Notes:**

Examples in the project: MortalitySource v2 reads pDDRCStates and DimState[DDRCState] to drop non-DDRC rows early; DimMonth v4 returns a typed empty table when Original Month Code is absent under pTimePeriod = 'annual'; DominantDrug v20 wraps the populated branch in BuildPopulated so [All Drugs] is not read when pDrugs = 'all' drops the column; MortalityDimColumns v8 includes MonthKey only when pTimePeriod = 'monthly'; MortalitySuppressed v20 picks DemographicColumns at runtime as the intersection of CDC's canonical demographic column set with the live MortalityModel schema.

---

## 35. Don't aggregate across CDC suppression-affected grain boundaries

**Category:** Data Integrity

**Apply when:** Unconditionally active

**Rule:**

CDC suppresses death counts of 1-9 as null. This makes any aggregation that crosses a grain boundary (county to state to division to region to us; monthly to annual; individual drug to all drugs; demographic axis slice to demographic-axis total) under-count by every suppressed row hidden inside the rolled-up partition. Each PBIX variant in the parameterized series pulls source files at its target grain (pGeographyLevel, pTimePeriod, pHasDemographics, pDrugs) and must not aggregate across these axes within the model. To get totals at a higher grain, use a separate PBIX variant whose parameters target that grain directly, loading grain-native CDC WONDER files. When designing visuals or DAX measures: bind to the grain at which the source file was pulled and do not author measures that SUM across a grain boundary. Lag derivatives operate over consecutive periods at the loaded grain (year-over-year for annual, month-over-month for monthly) and have correct semantics at their own grain. Use Deaths 12 Month Rolling (MortalityDynamics v11, monthly only) when an annual-scale overlay is needed on monthly data; it gates on 12 consecutive non-suppressed months and returns null otherwise.

**Notes:**

Reference memory entries: project_geography_aggregation.md (geography axis), project_time_grain_aggregation.md (time axis). Same suppression-driven undercount principle applied to both axes; the same logic extends to drug grain (don't sum individual-drug rows to All-Drug totals) and demographic grain (don't sum sex / race / age slices to demographic-axis totals).

---

## 36. Reference DAX measures with home table

**Category:** DAX

**Apply when:** Unconditionally active

**Rule:**

When naming or listing DAX measures in chat responses, summaries, plans, scoping inventories, and DB description fields, use the DAX convention Table[Measure Name] -- e.g., Mortality[Map Title], MetricsFact[Selected Metric Label], pTimePeriod[Death Count Death Rate Chart Title]. The home table comes from the dax_measures.home_table column.

Applies to:
- Lists of measures in summaries and scope inventories
- References in prose ("update Mortality[Map Title]")
- DB row descriptions and changes notes that reference measures

Does not apply to:
- DAX code itself, where the project convention is bare [Measure Name] for inter-measure references (matches Power BI's auto-completion default and avoids visual clutter inside formulas)
- Saved filenames (which already encode the home table as a prefix, e.g., Mortality_MapTitle_v6_*.txt)

**Notes:**

Added 2026-05-11 after user requested home-table prefix on measure references in chat / lists / DB descriptions.

---
