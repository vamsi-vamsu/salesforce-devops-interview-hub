# OwnBackup & Recovery

> Interview preparation notes. Answers distinguish the hands-on platform activities you have now described from broader product concepts and recommended operational practices.
>
> **Experience note:** You have described connecting Salesforce orgs to Own, configuring backup selections, working with Find, Compare and Restore, creating data-seeding templates, selecting source and target orgs, anonymizing seeded data, and scheduling operations through Jobs. Use those as your hands-on examples. For deeper implementation details not yet documented here (such as exact permissions, object-selection rules, relationship handling, masking rules and restore execution controls), describe your actual configuration rather than guessing.
> **Naming:** The interface in your screenshot is branded Own from Salesforce. Interviewers may still refer to the platform as OwnBackup. Product names and packaging can change, so use the labels visible in the environment you worked with.

## Own platform: feature overview

The platform brings together Salesforce backup, discovery, comparison, recovery and sandbox data-management capabilities. The main areas visible in the interface are Backup, Find, Compare, Restore, Seeding, Anonymize and Jobs.

### 1. Backup

**[Hands-on experience]**

- Connect the Salesforce orgs that need protection to Own.
- Configure a dedicated integration user in each connected Salesforce environment, with the permissions required by the integration.
- After the org is connected, choose which Salesforce metadata and data should be included in the backup configuration.
- Configure the backup scope and schedule based on the environment's requirements.

**Interview explanation:** "We connected our Salesforce orgs to Own using a dedicated integration user for each environment. Once the connection was established, we configured the metadata and data that needed to be included in the backup."

### 2. Find

Find is used to search for and identify Salesforce records or metadata in the available Own data and backup information. It helps narrow down the item you are investigating before deciding whether further comparison or recovery is needed.

**Interview explanation:** "If we need to locate a particular record or metadata component, we can use Find to search for it and identify the item we need to investigate."

### 3. Compare

Compare supports several comparison scenarios, including:

- Comparing metadata between Salesforce orgs.
- Comparing data between orgs or available backup states.
- Identifying differences that can help with investigation and troubleshooting.

**Interview explanation:** "We use Compare to understand differences between environments. Depending on what we are investigating, we can compare metadata or data and review the differences rather than manually checking each item."

### 4. Restore

Restore is used when Salesforce data or metadata has been deleted or needs to be recovered to an earlier state.

Typical use cases include:

- Recovering records that were accidentally deleted or changed.
- Recovering metadata that was deleted or changed unintentionally.
- Selecting the appropriate backup state and the items that need recovery.

**Interview explanation:** "If data or metadata is accidentally deleted from an org, we can use the Restore capability to recover the required items from the available backup. We first identify what needs to be recovered and then select the appropriate recovery point and scope."

### 5. Seeding

Seeding is used to populate a target Salesforce environment with selected data from a source environment.

**[Hands-on experience]**

1. Create a seeding template.
2. Select the source Salesforce org that contains the data.
3. Select the target Salesforce org that needs to receive the data.
4. Configure the data and objects required for the target environment.
5. Run the template to seed the target org.

For example, a lower environment such as SIT can be populated using selected data from Production. The screenshot provided shows the Seeding area, including the Templates and Scheduled tabs and examples of templates with different sources, including Production Data and SIT Data.

**Interview explanation:** "We created seeding templates in Own. In the template, we selected the source org where the data should come from and the target org where the data was needed. We could then configure the data required for that environment and execute the template. This helped us populate lower environments with relevant data instead of manually recreating records."

### 6. Anonymize

Anonymize is used with data seeding to mask sensitive information when data is copied into another Salesforce environment.

- Configure anonymization for sensitive fields and data that should not be exposed in the target environment.
- Apply masking as part of the seeding process, based on the configured rules.
- Preserve useful test data while reducing exposure of real customer information in non-production environments.

**Interview explanation:** "When we seed data from Production into a lower environment, we use Anonymize to mask sensitive information. That allows the team to work with realistic data for development and testing without exposing the original sensitive values."

Do not claim a particular masking algorithm, field transformation or relationship behavior unless you have confirmed it in the actual template configuration.

### 7. Jobs

Jobs is the area used to schedule and manage supported Own operations.

- Configure schedules based on operational requirements.
- Schedule supported activities such as backups and seeding.
- Review job execution and status to understand whether an operation completed or needs investigation.

**Interview explanation:** "Jobs is where we schedule the different Own operations based on our requirements. It gives us a place to manage scheduled activities and review their execution."

## Q: What is the difference between Backup and Seeding?

Backup protects Salesforce data and metadata by capturing copies that can be used for investigation and recovery. Seeding takes selected data from a source org and populates a target org for development, testing or other environment needs.

A backup is primarily for protection and recovery. Seeding is for making useful data available in another environment. Anonymization can be applied during seeding to mask sensitive values.

## Q: How do Backup, Find, Compare and Restore work together?

A practical workflow is:

1. Backup captures the configured Salesforce data and metadata.
2. Find helps locate the record or metadata component being investigated.
3. Compare helps identify what differs between the relevant states or environments.
4. Restore is used to recover the selected data or metadata when recovery is required.

The exact steps depend on the incident and the available backup state.

## Q: How do you seed a Salesforce sandbox with production-like data?

**[Hands-on experience]**

"I create a seeding template, select Production as the source org and the required sandbox as the target org, then configure the data needed by that environment. If the data contains sensitive information, I configure anonymization so those values are masked when the data is seeded. Once the template is ready, I execute it or schedule it through Jobs, depending on the requirement."

## Q: Why is anonymization important when seeding production data?

Production data can contain personally identifiable information and confidential customer or business details. Anonymization helps reduce the risk of exposing that information in lower environments while retaining realistic data structures for development and testing. The specific fields and masking behavior should follow the organization's security and compliance requirements.

## Q: How do you schedule Own operations?

Use Jobs to configure supported operations to run according to the team's requirements. The schedule should reflect the purpose of the activity, the environment, operational windows and the team's data-refresh needs. Monitor execution status and investigate failures rather than assuming a scheduled job completed successfully.

## Q: How does Own fit into a Salesforce DevOps strategy?

Own complements the source-control and deployment toolchain:

- Git and CI/CD manage versioned code and metadata changes and move them through the release process.
- Backup and Restore provide a recovery path for Salesforce data and metadata.
- Compare helps investigate differences between orgs or backup states.
- Seeding helps prepare lower environments with relevant data.
- Anonymization helps protect sensitive values when production-like data is used outside Production.
- Jobs supports scheduling and operational management of these activities.

The key distinction is that a deployment pipeline delivers changes, while backup and recovery protect the org if data or metadata needs to be recovered.

## Q: What would you do if a seeding job failed?

**[Approach]**

1. Open the job details and identify the failed step and error message.
2. Check that the source and target org connections are healthy.
3. Verify the integration user's access and the template's selected objects and data scope.
4. Check whether the failure is related to data dependencies, required fields, validation rules, automation or target-org constraints.
5. Correct the underlying issue and rerun the job in accordance with the team's change and data-handling procedures.
6. Validate that the expected records were created and that sensitive fields were anonymized as intended.

## Q: How do you validate data after seeding?

**[Approach]**

1. Confirm the job completed successfully.
2. Check record counts for the selected objects.
3. Validate representative records and important field values.
4. Check parent-child and lookup relationships.
5. Confirm that anonymization rules were applied to the intended sensitive fields.
6. Have the QA or business owner verify that the seeded dataset supports the required test scenarios.

## Q: How does Own compare with a Git-based deployment pipeline?

They solve different problems. Git and CI/CD provide source control, review, validation and controlled deployment of metadata. Own provides backup, recovery, comparison and data-seeding capabilities. A mature Salesforce DevOps process can use both: the pipeline manages delivery, and Own provides data protection and environment data-management capabilities.

---

## Broader backup and recovery concepts

The following are useful operational concepts. Treat them as recommended practices unless you have personally configured them in your Own environment.

### Q: How do you approach a Salesforce data restore?

1. Scope the incident: identify the affected objects, records, fields, time window and cause.
2. Stop the process causing the issue, such as a faulty integration, batch job or data load.
3. Identify the appropriate backup point and compare it with the current state.
4. Select the smallest sensible recovery scope to avoid overwriting valid later changes.
5. Consider parent-child relationships, lookups, record ownership and references.
6. Consider automation side effects such as triggers, flows, validation rules and integrations.
7. Validate restored record counts, field values, relationships and business outcomes.
8. Document the recovery and obtain the required owner sign-off.

### Q: How do you recover from accidental record deletion?

1. Check whether the records are still available in Salesforce's Recycle Bin and whether they can be undeleted.
2. If they are not available there, identify the appropriate backup and recovery point.
3. Investigate the cause of deletion using available audit information, logs and backup comparison.
4. Restore the required records and consider related records and relationships.
5. Validate the result and introduce safeguards against repeat incidents.

Salesforce Recycle Bin behavior and retention limits can vary by circumstance; verify current Salesforce documentation before quoting exact limits.

### Q: How do you validate data after restoration?

- Compare restored record counts with the expected scope.
- Check key fields on representative records.
- Validate parent-child and lookup relationships.
- Check related files and other dependent data where applicable.
- Run relevant reports or queries and ask data owners to validate business scenarios.
- Confirm that no unintended automation or integration side effects occurred.
- Record the restore point, scope, operator, validation results and exceptions.

### Q: How do you handle sensitive data in backups and lower environments?

Use least-privilege access, appropriate retention and audit controls, and the organization's encryption and compliance policies. When production-like data is seeded into non-production environments, configure anonymization for sensitive fields and validate the masking results before making the environment available to developers or testers.

### Q: Can Own replace a Git-based deployment pipeline?

No. A backup product helps recover a prior state; it does not replace source control, code review, automated validation or controlled release promotion. Git and the CI/CD pipeline remain responsible for delivering code and metadata changes.

## Likely interview follow-ups

- How do you connect Salesforce orgs to Own?
- Why use a dedicated integration user for each org?
- What is the difference between Backup and Seeding?
- How do you choose the source and target orgs in a seeding template?
- How do you mask sensitive information during seeding?
- What is the role of Find compared with Compare?
- When would you use Restore?
- How do you schedule backups and seeding operations?
- How do you investigate a failed job?
- How do you validate that seeded data is complete and anonymized?

## Interview positioning

Lead with the features you have described using: connecting orgs, configuring backup scope, finding and comparing information, restoring data or metadata, creating source-to-target seeding templates, anonymizing data and scheduling jobs. Then explain how those capabilities support release governance, recovery readiness and safe lower-environment testing. Be precise about which configuration details you personally handled and which are broader operational best practices.
