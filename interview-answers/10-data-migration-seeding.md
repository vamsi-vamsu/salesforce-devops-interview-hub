# Data Migration & Data Seeding

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.
>
> Tags: **[Experience]** = work described in your Copado and GitLab 2GP projects (Data Loader seeding after sandbox refreshes, sandbox templates, GDPR anonymization Apex, scratch orgs from a nightly snapshot). **[Approach]** = how I would do it. Large-scale production data migration is not in your documented experience, so those answers are framed as approach. Edit any [Experience] line that does not match what you actually did.
> Bulk API facts were checked against Salesforce developer documentation; confirm limits for your API version before quoting.

## Q: What is the difference between data migration and data seeding?

Data migration moves business data from one system or environment to another, often as part of an implementation or transition. Data seeding loads controlled sample or reference data into an environment so developers and testers can work with realistic scenarios. Migration emphasizes completeness and reconciliation; seeding emphasizes repeatability and fit for testing.

## Q: How do you migrate data between Salesforce environments?

I begin with source-to-target mapping, field validation and a migration sequence based on object relationships. I load parent records first, use External IDs to maintain relationships, and then load dependent records. After loading, I reconcile record counts, validate key fields and relationships, review rejected records and obtain business sign-off.

Before the load I also: take a backup of the target, temporarily plan for automation (validation rules, triggers, flows, duplicate rules) that could block or distort the load, do a trial run in a sandbox, and agree a cutover and rollback plan.

## Q: Which tools have you used for Salesforce data migration?

**[Experience]** Salesforce Data Loader, for seeding and reloading data in sandboxes after refreshes (alongside sandbox templates). I have also used SFDX/Git-based tooling for deployments, though that is metadata rather than data. **[Approach]** For larger or more complex migrations the usual options are:

- **Data Loader** (desktop or command line): insert, update, upsert, delete and export; good for controlled, repeatable CSV loads.
- **Data Import Wizard:** simple, up to 50,000 records for supported objects; fine for admins and small loads.
- **Salesforce CLI (`sf data` commands):** import/export trees, bulk upsert and queries, scriptable in CI.
- **ETL tools** such as Informatica, MuleSoft, Talend or similar: transformation-heavy, scheduled or integration-style loads.
- **Dataloader.io, Workbench, Gearset data deployments, Copado data deployment tools:** convenient for sandbox-to-sandbox moves.

I only claim tools I have used; if asked about one I have not, I describe it as an option I would evaluate and what I would look at (volume, transformation needs, relationship handling, cost, security).

## Q: What is Salesforce Data Loader?

A Salesforce client application for bulk import and export of records using CSV files. It supports insert, update, upsert, delete, hard delete and export, and uses either the SOAP-based API or the Bulk API (selectable in settings). It offers field mapping (SDL map files), success and error output files, and a command-line mode for automation. Key practical points: use External IDs for upserts and relationships, set batch size deliberately, review the error CSV after every run, and use Bulk API for large volumes. **[Experience]** I used it to seed data into sandboxes after refreshes.

## Q: What is the Bulk API?

An asynchronous REST-based API designed for loading or querying large volumes of records. You submit a job, upload data, and Salesforce processes it in the background, rather than making a synchronous call per batch of records. It supports insert, update, upsert, delete, hard delete and query, and processes records in parallel, which makes it the right choice for hundreds of thousands to millions of records. Each job returns success and failure results so failed rows can be fixed and reloaded.

## Q: What is the difference between Bulk API 1.0 and Bulk API 2.0?

- **Batch handling:** in 1.0 you create and manage batches yourself (manual batching of large files, custom code or by hand). In 2.0 you upload one CSV and Salesforce splits it into internal batches and processes them in parallel automatically.
- **Workflow:** 2.0 ingest is the same simple flow for every operation: create job, upload data, set state to UploadComplete, get results. It needs fewer calls and is easier to monitor.
- **Query:** 2.0 query automatically performs PK chunking and returns results from a single endpoint. In 1.0 you configure PK chunking yourself and iterate through result sets.
- **Authentication:** 2.0 supports the OAuth 2.0 flows used by other REST APIs; 1.0 needs a session header obtained through SOAP login.
- **Data formats:** 2.0 ingest is CSV only; 1.0 supports CSV, XML, JSON and binary attachments.
- **Limits:** 2.0 limits by total records uploaded per day (visible through the REST `/limits` endpoint); 1.0 limits by number of batches per day.
- **Availability:** 1.0 ingest from API v16.0, 2.0 ingest from v41.0 and query from v47.0.

Use 2.0 for streamlined workflows; use 1.0 only if its feature set or limits are a specific match (for example, XML/JSON payloads). Check the current Salesforce Bulk API limits page for exact allocations, since they change by edition and release.

## Q: How do you migrate parent-child records?

I load parent records first and capture or map their Salesforce IDs using stable External IDs. Then I load child records and reference the parent through the External ID relationship mapping. I validate relationship counts and sample records after the load to ensure the hierarchy is preserved.

## Q: How do you maintain relationships between records during migration?

**[Approach]**

1. Give every source record a stable key and store it in an External ID field on the matching Salesforce object (mark it unique and, where appropriate, case-insensitive).
2. Sequence loads by dependency: reference data and users first, then parents (Account), then children (Contact, Opportunity), then junction and related objects (OpportunityContactRole, attachments, notes).
3. Reference parents by External ID in the child file (for example `Account__r.Legacy_Id__c`) instead of by Salesforce ID, so the child load does not need a lookup step and can be rerun safely.
4. Handle self-lookups and circular references (for example Account hierarchy, Parent Case) with a second pass: load records first, then update the lookup field.
5. Verify afterwards with queries that count children per parent and find orphans.

**[Experience]** The same pattern applies to the sandbox seeding I did with Data Loader: parents first, then dependants.

## Q: What are External IDs, and why are they important?

An External ID is a field used to identify a Salesforce record using a value from an external system. It supports upsert operations and helps preserve relationships during migration without relying on Salesforce-generated record IDs. It is especially useful for repeatable loads and cross-system reconciliation. Practical notes: mark the field unique so upserts cannot create duplicates, make it indexed (External ID fields are indexed), and keep the key immutable.

## Q: How do you handle data mapping?

**[Approach]**

- Build a **source-to-target mapping document**: source system, object and field, target object and field, data type, length, required or not, transformation rule, default value, owner and sign-off.
- Resolve differences early: picklist value mapping, record types, ownership (map users, handle inactive users), date and time zone formats, currency and multi-currency, rich text, and field length truncation.
- Agree rules for nulls and defaults, and for fields that exist only in one system.
- Validate the mapping with a **small trial load** in a sandbox and have a business owner review the results, then lock the document under change control.
- Keep mapping files (Data Loader `.sdl` maps, CLI definitions, ETL jobs) in version control so loads are repeatable and reviewable.

## Q: How do you handle duplicate records during migration?

**[Approach]**

- **Before the load:** profile and deduplicate the source (exact matches on key fields, fuzzy matching on name, email or address) and agree a survivorship rule with the business (which record wins and which fields are merged).
- **During the load:** use **upsert with an External ID** so reruns update instead of creating duplicates, and decide whether to temporarily bypass or to keep Salesforce Duplicate Rules active (bypass for a controlled load, keep for ongoing integrations).
- **After the load:** run duplicate reports or queries for the same key fields, merge in Salesforce where necessary, and document decisions.
- Never delete records without a backup and an approved list.

## Q: How do you validate data after migration?

I compare source and target record counts, validate required fields and key business values, and check parent-child relationships. I also review rejected records and transformation exceptions, run business reports or queries, and ask data owners to validate representative scenarios. Validation should be documented before the migration is signed off.

## Q: How do you handle failed records?

**[Approach]**

1. Always capture the error file from the tool (Data Loader writes success and error CSVs; Bulk API 2.0 provides failed-record results).
2. Group errors by cause: validation rule failures, required fields missing, bad lookup or External ID references, picklist values not allowed, field-length or format errors, duplicate rule hits, lock errors (UNABLE_TO_LOCK_ROW), and permission or sharing issues.
3. Fix the cause at the source or in the transformation, not just in the file, then reload only the failed rows (upsert makes this safe).
4. Track counts: total, succeeded, failed, fixed, and the final reconciliation, and report them to the business.
5. For lock contention, order the file by parent ID, reduce batch size or use serial mode.

**[Experience]** The same discipline applies to deployment validation failures I have handled, such as a missing field in the destination org: find the root cause, fix at the source, re-validate, then proceed.

## Q: How do you migrate large volumes of Salesforce data?

**[Approach]**

- Use the **Bulk API** (Data Loader with Bulk API enabled, or Bulk API 2.0 directly) rather than synchronous loads.
- **Prepare the org:** plan the migration window, consider deactivating non-essential triggers, flows, workflow, validation and duplicate rules for the load (using a bypass such as a custom permission or setting, and re-enabling afterwards), and confirm storage and API limits.
- **Order and parallelism:** load by dependency; use parallel mode by default but switch to serial for objects that suffer record-locking (for example child records of the same parent).
- **Size batches sensibly,** sort data by parent to reduce lock contention, and avoid unnecessary roll-up recalculation (consider deferring sharing calculations on very large loads).
- **For large queries or exports,** use Bulk API 2.0 query or PK chunking.
- **Rehearse:** do a full dry run in a Full or Partial sandbox to estimate duration and find issues, then run the real load in the planned window with a rollback plan.
- **Monitor** job status and reconcile counts when each stage completes.

Check current Bulk API limits (for example daily record limits and concurrent job limits) before planning, since they vary.

## Q: How do you handle data transformation?

**[Approach]** Keep transformation **outside Salesforce** where possible, in a repeatable step between extract and load: scripts (Python or SQL), an ETL tool, or staging tables. Typical transformations: value mapping (picklists, record types, owners), concatenation or splitting fields, date and time zone conversion, trimming and standardizing, and deriving External IDs. Principles: transformation rules are documented and versioned, applied identically on every run, produce an exceptions list rather than silently dropping rows, and are tested against a small sample first. Keep raw source extracts unchanged so you can trace any value back.

## Q: How do you seed test data into scratch orgs?

I use repeatable data templates or scripts to create the minimum representative records needed for tests. The process respects object dependencies, uses stable reference values and avoids production-sensitive data. For automated tests, I prefer test factories or generated data where possible so tests remain isolated and repeatable.

**[Experience]** In the GitLab 2GP project, MR pipelines created scratch orgs from a nightly snapshot so developers and QA started with a preloaded org, with a fallback to a full deploy when the snapshot was skipped.

## Q: How do you create reusable test data templates?

**[Approach]**

- Define a small, **versioned dataset** (CSV or JSON files, or tree files for `sf data import tree`) that covers the main scenarios: a few accounts with contacts and opportunities, key edge cases, and reference data.
- Use **External IDs or reference names** for relationships so the load is repeatable and idempotent (upsert).
- Add an **Apex test data factory** (a utility class) for unit tests, so tests create their own data and do not depend on org data.
- Store templates in the repo, document load order, and run them from a script or pipeline step after scratch org or sandbox creation.
- Keep datasets small and purpose-specific (dev, QA regression, demo) and review when the data model changes.

**[Experience]** Sandbox templates and Data Loader seeding in Copado environments; snapshots for scratch orgs in the GitLab project.

## Q: How do you maintain referential integrity?

**[Approach]** Load in dependency order, reference parents through External IDs, run integrity checks after each stage (orphan queries and child-count comparisons against source), and be careful with deletes and re-loads (Salesforce enforces lookup and master-detail rules, and cascade delete on master-detail). Decide up front what to do with unresolved references: fail the row, load with a null lookup and flag it, or hold it for manual correction. Use a staging layer so referential problems are found before data touches Salesforce.

## Q: How do you handle sensitive data and data masking?

Production data copied to sandboxes contains personal information, so masking is part of the process rather than an afterthought. Controls:

- Mask or anonymize PII after every refresh (Salesforce Data Mask or an equivalent tool, or custom Apex/scripts), including emails so no real customer can be contacted.
- Prefer synthetic or generated data in Developer and Developer Pro sandboxes and scratch orgs.
- Restrict access by least privilege and keep an audit trail of the masking step.
- Do not put real personal data in repositories, tickets or shared CSV files; protect extract files and delete them after the load.

**[Experience]** I wrote Apex to anonymize user records for GDPR in sandboxes, and I work in a GDPR-regulated environment with production approval controls, privileged access management and segregation of duties.

## Q: How do you reconcile source and target record counts?

**[Approach]**

1. Record **expected counts** per object from the source before extraction (and any rows intentionally excluded, with reasons).
2. After each load, compare: source rows extracted = rows loaded successfully + rows rejected (with documented reasons) + rows excluded.
3. Query Salesforce counts per object (SOQL `COUNT()`, reports or the Bulk API job summary) and compare with the load result files.
4. Go beyond counts: compare control totals (sums of amounts, distinct key counts), sample records field by field, and check relationship counts (children per parent).
5. Document variances, fix or approve each one, and get business sign-off before cutover.

---

## Additional questions

## Q: What is the difference between insert, update, upsert and hard delete?

Insert creates new records; update changes existing records by Salesforce ID; upsert updates or inserts based on a Salesforce ID or External ID, which makes loads repeatable; delete sends records to the Recycle Bin; hard delete permanently removes them (requires the Bulk API Hard Delete permission) and should be used only with a backup and approval.

## Q: How do you avoid hitting automation problems during a load (triggers, flows, validation rules)?

**[Approach]** Plan for it: either make automation bulk-safe and leave it on when the logic must run, or use a controlled bypass (a custom permission or hierarchy custom setting checked by triggers, flows and validation rules) for the load user, and document and re-enable afterwards. Test with a realistic sample to catch governor limit issues, lock contention and unwanted emails or integration calls.

## Q: How do you preserve created dates and original owners?

**[Approach]** Enable the "Set Audit Fields upon Record Creation" permission (and Update Records with Inactive Owners where needed) for the migration user so CreatedDate, CreatedBy, LastModifiedDate and LastModifiedBy can be set on insert, and map owners by username or External ID. Confirm with the business whether original audit values are required.

## Q: How do you handle files, attachments and notes?

**[Approach]** Treat them as a separate workstream: migrate to Files (ContentVersion, then ContentDocumentLink for sharing) rather than legacy Attachments, check file size and storage limits, and reconcile counts. Notes are converted into the modern Note format. Test with a sample and expect longer run times.

## Q: What is your cutover and rollback plan for a migration?

**[Approach]** Rehearse at least once in a sandbox, freeze changes in the source, take backups of source and target, run the load with a checklist, validate and get sign-off, then open the system to users. For rollback, keep the backup and delete-by-External-ID or batch-ID lists ready so loaded records can be removed or restored if validation fails.

## Q: How would you automate data seeding in a CI pipeline?

**[Approach, anchored on GitLab experience]** Add a pipeline step after scratch org creation (or after snapshot restore) that runs a script using `sf data` commands or Data Loader CLI against versioned data files, fails the job on error, and cleans up the org afterwards. **[Experience]** My GitLab MR pipelines created scratch orgs from a snapshot, deployed, ran Apex tests, left the org for QA review and cleaned it up.

## Q: If UAT org is refreshed once a year, how you'll keep the data sync in UAT?

**I wouldn’t rely on a yearly refresh. I’d tie UAT refreshes to the release calendar, keep UAT metadata aligned through the pipeline and back promotion, use a template or scripted seeding for the data testers need, mask sensitive data, and announce the refresh and cutoff first so nobody loses uncommitted work. Then I’d confirm with the business what “latest” means, since for most UAT scenarios representative, correctly related data matters more than being today’s data.

## Likely follow-ups

- Why External IDs rather than Salesforce IDs? Stable across systems, repeatable and idempotent upserts, no ID lookup step.
- Parallel or serial mode? Parallel by default; serial when lock contention appears.
- How do you prove completeness to the business? Counts and control totals per object, relationship checks, sampled field comparisons and a signed reconciliation.
- What went wrong in a data load and how did you fix it? Use a real example from your sandbox seeding or your deployment validation failures; if none fits, describe the approach as such.
- How do you keep PII out of non-production environments? Masking after refresh, synthetic data for dev, least-privilege access, audit trail.

## Be honest about limits

Your documented data experience is seeding and refreshing sandboxes with Data Loader, templates and GDPR anonymization, plus scratch org snapshots. Lead with that, and present large-scale production migration, ETL tooling and Bulk API 2.0 details as the approach you would follow.