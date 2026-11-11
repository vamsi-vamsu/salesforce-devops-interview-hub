# Sandboxes & Environments

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.
>
> Tags: **[Experience]** = work described in the Copado and GitLab 2GP projects (sandbox refreshes, templates, Data Loader seeding, SSO config, GDPR anonymization Apex, Dev/QA/UAT/Prod + Hotfix pipeline, scratch orgs from snapshots). **[Approach]** = how I would do it. Edit any [Experience] line that does not match what you actually did.
> Numbers below were cross-checked against Salesforce's sandbox guide and practitioner sources; confirm current entitlements in Setup before quoting.

## Q: What are the different types of Salesforce sandboxes?

Four types: Developer, Developer Pro, Partial Copy and Full. All copy production metadata; only Partial Copy (a sample, via template) and Full (everything) also copy data.

| Type | Data copied | Storage (data / files) | Refresh interval | Template | Typical use |
|---|---|---|---|---|---|
| Developer | None | 200 MB / 200 MB | 1 day | No | Individual development, unit testing |
| Developer Pro | None | 1 GB / 1 GB | 1 day | No | Shared dev, integration, QA with seeded data |
| Partial Copy | Sample via template | 5 GB / same as production | 5 days | Required | UAT, integration, training |
| Full | All production data | Same as production | 29 days | Optional | Performance testing, staging, release rehearsal |

Scratch orgs are not sandboxes: they are disposable, source-driven orgs created from a Dev Hub. Note: some Salesforce marketing pages list Developer file storage as 5 MB, while Salesforce Help and most references say 200 MB; verify in Setup.

## Q: What is the difference between Developer, Developer Pro, Partial Copy, and Full Copy sandboxes?

- **Developer:** metadata only, small storage, quick to create. Best for one person's feature work with a small seeded dataset.
- **Developer Pro:** metadata only with 1 GB. Suits shared integration or QA environments that need larger generated datasets.
- **Partial Copy:** metadata plus a template-defined sample of production data (commonly cited cap of 10,000 records per selected object). Good for UAT and realistic integration testing without production-scale storage.
- **Full:** complete replica including data and files. Use when volume or shape matters: performance testing, migration rehearsal, final release validation.

I choose by purpose: isolation and speed, Developer; more seeded data, Developer Pro; realistic samples, Partial; production scale, Full. Full costs more and refreshes only every 29 days, so it is not for day-to-day development.

## Q: What happens during a sandbox refresh?

**Platform behavior:** the sandbox is rebuilt from a copy of production, replacing its contents. Anything not in production (customizations, test data, unreleased metadata) is lost. The new copy must be activated, and refreshes can queue for hours, so schedule off-peak.

**What commonly needs attention afterward:**

- Org ID and integrations: practitioners report a new Org ID after refresh, so SSO, connected apps and integrations referencing the old one fail until updated (confirm in your org).
- Credentials and endpoints: Named Credentials, OAuth tokens and URLs need re-authorization; never let a sandbox keep calling production systems.
- Email: deliverability resets to a restricted setting (System Email Only). Do not choose No Access, which blocks password-reset and verification emails.
- Scheduled jobs and automation: time-based jobs and flows often need review and reactivation; pending job records do not carry over.
- Users and permissions: usernames get the sandbox suffix; roles and access should be re-checked.
- Data: field history does not carry over; data exists only for Partial and Full. Mask PII before anyone tests.

**[Experience]** In my Copado work I handled sandbox refreshes with sandbox templates, reseeded data with Data Loader, reconfigured SSO, and ran Apex to anonymize user records for GDPR.

## Q: How do you manage Git branches after a sandbox refresh?

**[Approach, anchored on Copado experience]** Git is the source of truth; the sandbox is a disposable copy. After a refresh the org matches production, so unreleased work is gone from the org but should still exist on its branch.

1. Confirm unreleased work is pushed on its feature or promotion branches.
2. Bring the environment branch in line with production by back-promoting or merging main down, so lower branches are not behind (admin edits, hotfixes).
3. Re-deploy unreleased stories from Git instead of rebuilding them by hand.
4. Run an org-to-branch comparison to prove they match before development resumes.

**[Experience]** In Copado, back promotion and org sync were enabled on every environment, which lets lower environments catch up after production changes. The same-Apex-class conflict I resolved with another developer is the kind of case that surfaces during this catch-up.

## Q: How do you maintain consistency between Git and Salesforce environments?

- Git is the single source of truth; no direct changes in higher orgs except controlled hotfixes.
- Promote only through the pipeline (Dev, QA, UAT, Production) with PR quality gates and approvals.
- Back-promote production and hotfix changes into lower environments and branches.
- Run periodic org-vs-repo comparisons to catch untracked changes.
- Track what is deployed where (version or story IDs) so "what is in QA?" has an instant answer.

**[Experience]** Copado pipeline: Dev, QA, UAT, Production plus a Hotfix org tied to production, a PR quality gate needing 2 approvals, and rollback, org sync and back promotion on every environment. The earlier process (changesets/sfdx without full source control) was the drift problem this replaced.

## Q: How do you manage sandbox-specific configurations?

**[Approach]** Keep everything environment-specific out of the code:

- Named Credentials / External Credentials for endpoints and auth, set per environment.
- Custom Metadata and Custom Settings for values that differ by environment.
- `replacements` in sfdx-project.json (or pipeline variable substitution) for values that change at deploy time.
- SandboxPostCopy Apex for repeatable post-refresh steps.
- A short post-refresh runbook for manual items (SSO, connected app re-authorization, email deliverability).
- Secrets in a vault or protected CI variables, never in the repo.

**[Experience]** In the GitLab project, secrets and auth lived in protected GitLab variables and GCP Secret Manager, with JWT auth through a Connected App.

## Q: How do you handle data templates during sandbox refreshes?

- Partial Copy requires a template; Full is optional. Templates choose which objects (and related objects) are copied and how much.
- Design templates by purpose: a UAT template includes the objects and relationships business scenarios need, and stays small to protect storage and refresh time.
- Maintain templates like code: clear names, owners, review when the data model changes, and test a refresh against them.
- Mind relationships: excluding a parent object can break child data.

**[Experience]** I used sandbox templates during refreshes in the Copado environment and topped up with Data Loader seeding for records the template did not cover.

## Q: How do you manage parallel development environments?

- One environment per developer or story (Developer sandboxes or scratch orgs) so work does not collide.
- Feature branches off the integration branch, promoted or merged into a shared environment with automated tests.
- A shared integration/QA environment where merged work is validated together.
- Clear ownership and naming for every environment.
- Conflict detection at merge or promotion, resolved by the developers involved.

**[Experience]** In Copado, multiple dev orgs connected to a shared QA first. Copado flagged two stories that changed the same Apex class (auto-merge excludes classes) and both developers resolved it together. In GitLab, each developer used a scratch org from develop, a feature branch and an MR to develop.

## Q: How do you coordinate releases across multiple environments?

**[Approach, anchored on both projects]**

- Release calendar with code freeze, regression window and UAT sign-off before promotion.
- Promote a validated, versioned set of changes in order: Dev, QA, UAT, Production.
- Pre-deployment validation (check-only with tests), dependency-order check, rollback plan, post-deployment verification.
- CAB or release approvals for production; communication to business and offshore teams.
- Anything not ready is removed before the release cut rather than shipped half-done.

**[Experience]** GitLab: release branch cut from develop, MR to main, staging pipeline triggered manually, then promote to staging and production, with unready work reverted from develop beforehand. Copado: PR quality gate with 2 approvals and rollback available. Staging install time dropped from about 5 hours to about 2 by parallelizing package installs with `needs` and downstream pipelines.

## Q: How do you prevent environment drift?

Drift comes from direct edits in orgs, hotfixes not back-ported, partial deployments and refreshes. Prevention:

- Restrict direct setup changes in QA, UAT and Production (permissions, change control).
- Always back-promote hotfixes and production-only config.
- Scheduled comparisons (org vs branch, org vs org) with alerts on differences.
- Refresh lower sandboxes on a cadence so they re-converge to production.
- Deploy the same artifact through every stage so environments differ only in config, not code.

**[Experience]** In the GitLab 2GP project, installing the same package versions through QA and staging in dependency order, with version IDs passed via artifacts, kept environments aligned.

## Q: What are the standard Salesforce sandbox storage capacities?

Salesforce's published standard limits are 200 MB data and 200 MB file storage for a Developer sandbox, and 1 GB data and 1 GB file storage for a Developer Pro sandbox. A Partial Copy sandbox has 5 GB data storage and file storage matching Production. A Full sandbox has the same data and file storage allocation as Production. Developer and Developer Pro data storage can be upgraded to 400 MB and 2 GB respectively where an upgrade license is available. Check the current Salesforce Help page and the org's actual license entitlements before quoting these values. <https://help.salesforce.com/s/articleView?id=sf.data_sandbox_environments.htm&language=en_US&type=5>

## Q: What is your strategy for refreshing sandboxes?

I refresh based on purpose and release cadence rather than refreshing every sandbox at the same time.

1. **Plan:** identify source org, owners, active development, open User Stories, test data and integration needs; pick a window and agree a cutoff.
2. **Before:** developers commit or promote work in progress; the release team records branch and org state; document Named Credentials, connected apps, endpoints and sandbox-specific settings.
3. **Refresh:** use the right template for Partial or Full, and attach a SandboxPostCopy Apex class so common fixes run on every create or refresh.
4. **After:** activate, set email deliverability, reauthorize integrations, reactivate jobs, load or mask data, verify users and permissions, and reconcile the org with Git before development resumes.
5. **Verify:** smoke test key flows before handing the sandbox back.

I avoid refreshing an active development org without coordinating with its users.

## Q: How do you handle storage issues in a Salesforce sandbox?

I check Setup > Storage Usage to identify whether data or file storage is the constraint and which objects or files consume the most space. I remove obsolete test records and files using an approved cleanup process, archive or reduce test data where appropriate, and review logs or attachments that are accumulating. For ongoing needs, I consider a Developer Pro or an available storage upgrade, or use a Partial Copy or Full sandbox when the testing purpose requires more data. When storage is full, new data cannot be saved, so I check usage proactively rather than waiting. I preserve required test scenarios and relationships instead of deleting records indiscriminately.

## Q: What should you check before refreshing a development sandbox?

I check for uncommitted metadata, open User Stories, local developer work, test data, integration endpoints, connected app credentials, scheduled jobs and sandbox-specific settings. I notify the developers and agree a cutoff. After refresh, I reconfigure endpoints and credentials, run any post-copy scripts, restore required test data and verify Git alignment. A sandbox refresh should not become an unplanned way to discard changes that were never committed.

## Q: What is the difference between sandbox refresh interval and sandbox storage capacity?

The refresh interval is the minimum time between refreshes for a sandbox type, while storage capacity defines how much data and file content the sandbox can hold. Standard Salesforce intervals are one day for Developer and Developer Pro, five days for Partial Copy and 29 days for Full. These are platform limits and can be subject to product changes, so I confirm current entitlements in Salesforce Setup and Help. <https://help.salesforce.com/s/articleView?id=sf.data_sandbox_environments.htm&language=en_US&type=5>

## Q: How do you manage sandbox storage when several teams need realistic test data?

I define a data strategy by environment purpose. Developer sandboxes generally use small, targeted datasets, while integration and QA environments use controlled datasets that cover cross-system and regression scenarios. Partial Copy uses a sandbox template to select relevant sample data, and Full Copy is reserved for cases that require a production-scale copy. I use repeatable data seeding and cleanup, avoid copying unnecessary records, and apply masking when sensitive data is involved. Giving each team a defined dataset stops them overwriting each other.

---

## Additional questions

## Q: How do you protect sensitive data in sandboxes (GDPR)?

Sandbox copies of production contain real personal data, and restricting email only stops delivery; it does not anonymize stored data. Controls: mask or anonymize PII after every refresh (Salesforce Data Mask, an equivalent tool, or custom Apex/scripts), scramble emails so automations cannot reach real customers, limit access by least privilege, prefer synthetic data in Developer and Developer Pro, and keep an audit trail of the masking step.

**[Experience]** I wrote Apex to anonymize user records for GDPR, and I work in a GDPR-regulated environment with production approval controls, privileged access management and segregation of duties.

## Q: How do you seed test data repeatably?

**[Approach]** Scripted loads (Data Loader or `sf data` commands) driven by versioned data files or data plans, parent-before-child load order, and external IDs for relationships. The same process for every refresh gives QA the same baseline; cleanup scripts handle reset. **[Experience]** Data Loader seeding after refreshes; scratch orgs from a nightly snapshot in the GitLab project give developers a preloaded starting point.

## Q: Sandbox or scratch org: when do you use which?

| | Sandbox | Scratch org |
|---|---|---|
| Source | Copy of production | Built from source and a definition file (Dev Hub) |
| Lifespan | Persistent | Short-lived (up to 30 days, default 7) |
| Data | Depends on type | Seeded by script |
| Best for | UAT, integration, staging, org-based dev | Package/source-driven dev, CI validation, review orgs |

Scratch orgs suit 2GP and CI because they are clean and reproducible; sandboxes are needed where production-like config, data and integrations matter. **[Experience]** In the GitLab project, MR pipelines created scratch orgs from a nightly snapshot, deployed, ran Apex tests and left the org for QA review, with fallback to a full deploy if the snapshot was skipped.

## Q: How do Salesforce seasonal releases affect sandboxes?

Before each major release (three a year), Salesforce offers a sandbox preview: some sandboxes upgrade early while production stays on the prior version, giving teams a regression window. Plan refreshes around it, because a refresh during preview can land the sandbox on the preview or current version depending on the instance. Check the release schedule and decide per environment which version you want.

## Q: What do you do after a production hotfix?

**[Approach, anchored on Copado]** Fix in a Hotfix environment tied to production, test, deploy, then back-promote the change to all lower environments and branches so the next release does not overwrite it, and confirm with a comparison. **[Experience]** My Copado pipeline had a Hotfix org tied to production with back promotion enabled.

## Q: What is SandboxPostCopy and what would you automate with it?

An Apex interface (`SandboxPostCopy`) whose class runs automatically after a sandbox is created or refreshed. **[Approach]** I would use it to scramble or mask emails, set Custom Settings and Custom Metadata, update endpoints, create or adjust test users, and enqueue seed jobs. It cannot do everything: email deliverability resets to a restricted default, and SSO or connected app re-authorization is often manual, so keep a runbook alongside it. (If you have not used it directly, say so and describe your manual or Apex-based post-refresh steps instead.)

## Q: A developer says their work disappeared after a refresh. What do you do?

Check the branch first. If the work was committed or promoted, redeploy from Git. If it only existed in the org, it is gone, which is why the pre-refresh cutoff and commit rule exist. Then add the missing step to the runbook.

## Q: After a refresh, an integration is calling production. What do you do?

Stop it first: disable the integration or point it to a safe endpoint. Then fix Named Credentials and endpoints, rotate any copied secrets and review what was called. Prevent recurrence with a post-copy script and checklist.

## Q: Customers received emails from a sandbox. What do you do?

Contain by setting deliverability to System Email Only and pausing jobs and flows. Scramble email fields, find the sending automation (triggers, flows, batch jobs, marketing integrations) and add masking to the post-refresh routine. In a GDPR environment, escalate to privacy/compliance.

## Q: A deployment passes in QA but fails in UAT. What do you check?

Compare the environments for a missing dependency (for example a field present in QA but not UAT), config differences, data differences or drift. Fix at the source and redeploy through the pipeline rather than editing UAT. **[Experience]** An Apex class failed validation because a referenced field was missing in the destination org; I recommitted the field to the user story, re-ran validation and deployed.

## Q: You need fresh data but a Full sandbox is not due to refresh for 29 days. What do you do?

Use a Partial Copy or Developer Pro for the immediate need, use scripted seeding, or, if a Full refresh is justified, escalate for approval. Do not block the team on the interval; plan Full refreshes around release milestones.

## Likely follow-ups

- Why Partial rather than Full for UAT? Cost, refresh frequency and data protection; Full only when volume or shape matters.
- How do you make sure a refresh does not break SSO? Document and reconfigure SSO in the runbook, and test login before announcing the sandbox.
- Who is allowed to refresh? Restricted by the Manage Sandbox permission and a request process.
- How do you coordinate offshore teams (Slovakia, India) around refreshes? Shared calendar, announced cutoff, post-refresh notice with status and access details.
- What would you improve? Automate masking and post-copy configuration, and schedule drift comparisons.

## Be honest about limits

You told the Copado panel you have not worked directly on SOX and have not done much Salesforce development recently. Lead with release, environment and pipeline ownership, and describe anything outside it as your approach.