# OwnBackup & Recovery

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.
>
> **Honesty note:** none of your documented project experience (Copado pipeline, GitLab 2GP pipeline, sandbox refreshes, Data Loader seeding, GDPR anonymization Apex) involves operating OwnBackup. Every OwnBackup-specific answer below is therefore written as **[Conceptual]** knowledge or **[Approach]**. **[Related experience]** lines point to adjacent work you can truthfully cite (rollback and back promotion in Copado, release rollback planning, data seeding, GDPR controls). If you have in fact used OwnBackup, edit those answers to say so.
> **Naming:** OwnBackup became Own Company in 2023, Salesforce's acquisition closed in November 2024, and the product is now marketed as **Own from Salesforce**, with Own Recover now called **Salesforce Backup & Recover**. Interviewers may still say "OwnBackup" or "Own Recover", so recognize all three names. Confirm current product names and features on Salesforce's site before the interview.

## Q: What is OwnBackup?

**[Conceptual]** OwnBackup is a cloud-to-cloud backup and recovery platform for Salesforce (and some other SaaS data), founded in 2015. It takes automated backups of Salesforce data and metadata, lets teams compare backups over time, and supports selective restoration of records. The company later broadened into archiving, sandbox data seeding and masking, and security, renamed itself Own Company in 2023, and was acquired by Salesforce (closed November 2024). Today the products sit under "Own from Salesforce": Backup & Recover (formerly Own Recover), Archive, Data Mask & Seed (sandbox seeding), and security tooling.

In short: it is the safety net for data, separate from the deployment pipeline.

## Q: How does OwnBackup support Salesforce backup and recovery?

**[Conceptual]**

- **Automated backups:** scheduled (typically daily) and on-demand backups of data, metadata and files, stored outside the Salesforce org so an org-level problem does not take the backup with it. Some editions add higher-frequency protection (continuous data protection) for critical objects.
- **Monitoring and alerts:** backup status, anomaly and data-change alerts (for example, unusually large deletions or changes) so problems are spotted early.
- **Compare:** compare a backup with current org data (or two backups) to see exactly which records and fields changed or were deleted.
- **Selective restore:** restore individual records, specific fields or whole objects rather than rolling back an entire org, with attention to preserving relationships and not overwriting legitimate later changes.
- **Metadata backup and restore:** backups of configuration so lost or broken metadata can be investigated or recovered.
- **Seeding and masking:** use backed-up data to populate sandboxes with right-sized, masked test data.
- **Compliance:** retention policies, encryption and audit trails to support requirements such as GDPR.

**[Related experience]** My Copado pipeline had rollback, org sync and back promotion enabled on every environment, which covers *metadata* recovery; a backup tool covers *data* recovery.

## Q: What is the difference between metadata deployment and data backup?

- **Metadata deployment** moves *configuration and code* (objects, fields, Apex, flows, permission sets, page layouts) between environments through the pipeline, driven by Git. It is versioned in source control, and a rollback means redeploying a previous version.
- **Data backup** protects *records and files* (Accounts, Contacts, Opportunities, attachments and so on) that live only in the org. Git and the deployment pipeline do not contain business data, so if records are deleted or corrupted, the pipeline cannot bring them back.

Two consequences worth stating in an interview: (1) a good rollback plan covers both: redeploy metadata from Git *and* restore any data affected by the release or by the incident; (2) metadata backups from a backup tool are a second safety net, but Git remains the source of truth for configuration.

**[Related experience]** Rollback planning and back promotion in Copado, and version tracking and rollback planning for production releases in the GitLab 2GP project, are the metadata side of this answer.

## Q: How do you restore Salesforce data using OwnBackup?

**[Approach]** (conceptual, not claimed hands-on)

1. **Scope the incident:** what was lost or corrupted (objects, fields, date range, cause), who is affected, and whether the problem is still happening (stop a runaway integration, batch job or bad load first).
2. **Choose the backup point:** the latest backup *before* the incident; compare it to current data to see precisely what changed.
3. **Select what to restore:** the smallest sensible set of records and fields; avoid overwriting legitimate later changes.
4. **Plan the order and relationships:** restore parents before children, include related records, and think about how lookups, master-detail relationships, record owners and record IDs are handled (restored records may get new IDs, so references need care).
5. **Control automation:** consider the effect of triggers, flows, validation rules, duplicate rules and email alerts during the restore (bypass or pause where appropriate and agreed).
6. **Rehearse:** run the restore in a sandbox first when time allows, check the results, then run in production under change control and approval.
7. **Validate and communicate:** reconcile, get owner sign-off and document what was done (see the validation question below).

## Q: How do you recover from accidental record deletion?

**[Approach]**

1. **Act quickly:** deleted records go to the **Recycle Bin**, where they are kept for 15 days (subject to storage-based limits). Confirm current limits in Salesforce Help. If the records are in the bin, an admin or the owner can undelete them, which restores the record with its original ID and, in most cases, its child records.
2. **If they are not in the bin** (emptied, hard-deleted via the Bulk API, deleted by a user without bin access, or older than the retention period), restore from a **backup** such as OwnBackup/Salesforce Backup & Recover. Salesforce's older Data Recovery Service was retired in 2020, so do not plan around it.
3. **Find the cause:** who or what deleted the records (user, flow, Apex, integration, data load, mass delete) using the Setup Audit Trail, debug logs and the backup compare view, and stop it recurring.
4. **Restore with relationships:** parents before children, mind lookups and master-detail relationships, related files and attachments, and automation side effects.
5. **Validate and prevent:** reconcile counts and sample records, then add guardrails (restricted delete permissions, deletion alerts, approval for mass deletes, backup before every large data operation or release).

**[Related experience]** The sandbox seeding I did with Data Loader and the data-handling discipline from deployments (backup before change, rollback plan) apply here.

## Q: How does OwnBackup fit into a Salesforce DevOps strategy?

**[Approach]**

- **Before risky changes:** take (or verify the freshness of) a backup before a major release, data migration, mass update or integration go-live, so data can be recovered if something goes wrong. Pair it with the deployment rollback plan.
- **Release and incident process:** include "data impact and restore plan" in the release checklist and CAB review, and define who can trigger a restore and under what approval.
- **Sandbox strategy:** use backup data (and Own's Data Mask & Seed, where licensed) to populate sandboxes with right-sized, **masked** data, which supports the GDPR requirement to keep personal data out of non-production environments.
- **Governance and compliance:** retention, encryption, audit trail and segregation of duties around who can back up and restore, which matches environments with production approval controls and privileged access management.
- **Resilience testing:** schedule restore rehearsals and record the results; a backup that has never been restored is untested.
- **Position in the toolchain:** Git plus pipeline (Copado, GitLab CI/CD, Gearset, GitHub Actions) manages *code and configuration*; the backup tool manages *data*. They complement each other rather than overlap.

**[Related experience]** GDPR-regulated environment with production approval controls, privileged access management and segregation of duties; anonymization Apex for sandbox user records; release approvals and CAB for production.

## Q: How do you validate data after restoration?

**[Approach]**

1. **Counts:** compare the number of records restored with the number expected from the backup and the compare report, per object.
2. **Field-level checks:** sample records and compare key fields to the backup (including owner, status, dates and amounts), and confirm no legitimate later changes were overwritten.
3. **Relationships:** check parent-child and lookup relationships (children per parent, orphans), and related files, notes and attachments.
4. **Business validation:** run key reports, dashboards or queries and ask data owners to validate representative scenarios.
5. **Automation side effects:** confirm no unintended emails, integration messages, duplicate records or roll-up distortions were triggered.
6. **Document and sign off:** record what was restored, from which backup, by whom, the validation results and any exceptions, and close the incident with the business owner.

---

## Additional questions

## Q: What are the limits of Salesforce's native recovery options without a backup tool?

**[Conceptual]** The Recycle Bin (15 days, limited by storage), Data Export Service (weekly or monthly exports, manual and slow to restore from), and metadata in Git. These do not give automated daily backups, record-level comparison, relationship-aware selective restore or fast recovery at scale, which is the gap a backup product fills. Salesforce also now offers its own native Backup & Recover product (the former Own Recover).

## Q: What would you put in a data restore runbook?

**[Approach]** Roles and approvals; how to declare an incident; how to stop the cause; where backups live and how to identify the right restore point; the step order (scope, compare, select, restore in order, validate); how to handle automation during restore; communication templates; validation checklist; post-incident review; and a schedule for restore drills.

## Q: How do you decide RPO and RTO for Salesforce data?

**[Approach]** Work with the business to set how much data loss is tolerable (RPO) and how quickly service must be restored (RTO) per object or process, then match the backup frequency and restore process to it: daily backups for most data, higher-frequency protection for critical objects, and a tested restore procedure that can meet the RTO.

## Q: How do you handle sensitive data in backups?

**[Approach]** Encrypt backups, restrict access with least privilege and audit it, apply retention and deletion policies (including GDPR erasure requests, which need a defined process for removing or masking data in backups where applicable), and never use unmasked backup data for development or testing.

## Q: Can OwnBackup replace a Git-based deployment pipeline?

**[Conceptual]** No. A backup restores a state; it is not a controlled, reviewable, repeatable path for delivering changes. Use Git and the pipeline for change delivery and the backup for recovery.

## Q: Tell me about a time you had to roll back or recover from a failed change.

Use a real example. Candidates from your documented experience: a deployment validation failure where an Apex class referenced a field missing in the destination org (you recommitted the field to the user story, re-ran validation and deployed), or the Copado rollback and back-promotion capabilities you enabled. Be clear that these were metadata recoveries; if asked about data recovery, describe your approach.

## Likely follow-ups

- Have you used OwnBackup or Salesforce Backup & Recover? Answer truthfully; if not, say you understand the concepts and describe how you would adopt it.
- What happens to record IDs when you restore? Restored records may receive new IDs, so relationships and external references must be handled; Recycle Bin undelete preserves IDs.
- How often should backups run? Driven by RPO: daily for most objects, more often for critical ones, plus a backup before major releases or data operations.
- What if the backup itself is corrupted or out of date? Hence monitoring, integrity checks and periodic restore tests.
- How do you restore metadata versus data? Metadata from Git through the pipeline (rollback), data from the backup tool.

## Be honest about limits

Position yourself as someone who owns release, rollback and environment processes and understands where a data backup tool fits, rather than as an OwnBackup administrator. Recycle Bin limits and product features change, so check Salesforce Help and the current product pages before quoting specifics.