# Gearset Release Management

Interview answers covering Gearset CI jobs, Git integration, deployment pipelines, release planning, validation, recovery and Salesforce environment promotion.

These answers describe Gearset experience and a practical release-management approach. Keep environment names, branch names, exact pipeline counts, integrations and metrics aligned with the implementation actually used. Do not reuse company or client names from interview reference material.

## Q: Explain your Gearset release process end to end.

I start by understanding the Salesforce change, its dependencies and the target release. Developers work in the agreed Git branching model and raise pull requests for review. Gearset is used to compare and validate metadata changes against the target environment, identify missing dependencies and run configured checks. Once the pull request is approved and the release scope is confirmed, changes are promoted through the required environments, such as QA and UAT, before production. After deployment, I verify results, run smoke tests with QA or business owners, check critical automation and integrations, and communicate the release outcome.

## Q: What is the difference between a Gearset CI job and a deployment pipeline?

A CI job validates changes when a configured source-control event occurs. It can compare metadata, run validation and provide feedback before changes are merged or promoted. A deployment pipeline provides a controlled path for moving approved changes between Salesforce environments, with visibility into release stages and deployment status. CI helps identify problems early; the pipeline governs promotion of approved changes.

## Q: How do you connect Gearset with Git?

I connect the relevant Git provider and repository to Gearset using an authorized integration. I select the repository, branches and Salesforce environments that participate in the workflow. I verify that Gearset can read the intended source branch and compare it with the correct target org. I test the connection with a small, controlled change and confirm that the comparison includes the expected metadata. Repository permissions and credentials should follow least-privilege principles.

## Q: How do you configure a Gearset CI job?

I define the source branch, target Salesforce environment, trigger and metadata scope. I configure validation and test options required by the team's release policy, then connect the job to the relevant pull-request or branch event. I test both a successful change and an intentionally invalid change so the team can confirm that feedback is generated and failures are visible. Exact settings depend on the repository strategy and Gearset features enabled.

## Q: How do you use webhooks between GitHub and Gearset?

I confirm whether the integration uses Gearset's supported repository connection and event mechanism. Where a webhook is required, I configure the endpoint and event subscriptions according to the product setup, then verify delivery and permissions. I test a pull request or push event and confirm that the expected CI job starts. If it does not, I check webhook delivery status, repository permissions, branch filters and job configuration. I avoid exposing webhook secrets or tokens in logs.

## Q: What quality gates do you apply before production?

I use the controls defined by release governance. Typical gates include peer review, successful CI validation, required Apex tests, static analysis where configured, resolution of deployment dependencies, QA and UAT sign-off, change approval and a confirmed production deployment window. The gate should reflect the team's actual policy rather than checks that are not configured.

## Q: How do you manage a release containing more than 100 pull requests?

I treat it as a planned release train. I maintain a release inventory with each pull request, owner, work item, approval, test status, dependencies and intended release. I group related changes into logical batches and identify shared metadata or dependencies before deciding deployment order. QA and UAT sign-off are tracked at the appropriate level. Before production, I confirm the final scope, deployment sequence, runbook, owners and recovery plan. A large number of pull requests does not mean they should all be deployed as one unreviewed batch.

## Q: Would you combine all pull requests into one production deployment?

Not automatically. I check whether the changes are independent, whether they share metadata and whether dependencies require a particular order. If they form one tested release unit, they can be deployed together. If a change is not ready or introduces unacceptable risk, I isolate or defer it through the release process. I verify that excluding a pull request does not leave another approved change incomplete.

## Q: How do you decide deployment order when multiple pull requests have dependencies?

I identify metadata and functional dependencies, then build a deployment sequence that satisfies them. Foundational objects and fields may need to be available before automation, Apex classes or components that reference them. I validate the proposed sequence in a lower environment and review comparison results for missing dependencies. The order should be based on the dependency graph and deployment errors, not simply the order in which pull requests were merged.

## Q: A Gearset deployment works in QA but fails in UAT. How do you troubleshoot it?

I review the first meaningful deployment error and compare source and target states. I check missing dependencies, metadata differences, permissions, record types, custom settings or metadata, Named Credentials, environment-specific values and test-data differences. I correct the issue in source control rather than making an untracked change in UAT, then rerun validation and confirm the corrected change can move through the release path.

## Q: What do you do when a Gearset validation fails?

I inspect validation results and identify the first root-cause error rather than treating every downstream error as separate. I determine whether it is a metadata dependency, Apex test failure, permission issue, org difference or configuration problem. I fix the source change, rerun validation and confirm the target org and release scope before proceeding. If the issue cannot be resolved within the release window, I follow the agreed defer or go/no-go process.

## Q: How do you handle rollback after a production deployment containing multiple pull requests?

I assess business impact and identify the exact components and changes associated with the incident. I check whether reverting the whole release would remove unrelated functionality or affect dependent metadata and data. If a targeted rollback is safe, I prepare and validate the reversal. If rollback is risky or cannot reverse data changes cleanly, I consider a forward fix. I use Git history and deployment records to preserve traceability, then reconcile the final state into the appropriate branches and environments.

## Q: Would you perform rollback through Gearset or Git?

I use Git as the source of truth for the intended metadata state and Gearset to compare, validate and deploy the corrective change where appropriate. The recovery method depends on the change and release setup. Reverting a Git commit does not automatically restore production, and redeploying an earlier metadata version may not safely reverse data changes. I validate the recovery plan and reconcile the result in source control.

## Q: Can you revert one component without reverting an entire pull request?

Yes, when the component is independent and its dependencies are understood. I identify the exact component and the commit or metadata change that introduced it, then prepare a focused corrective change. I check whether other components in the same pull request depend on it, validate the resulting metadata comparison and test the correction before deployment. I avoid reverting a shared file or component without checking its impact on other changes.

## Q: How do you protect branches and secure the Gearset release process?

I protect critical branches with pull-request reviews and required status checks. I restrict who can merge to release and production branches, and limit production deployment permissions to authorized roles. Credentials and integration tokens are stored using approved secure mechanisms, not committed to Git or printed in pipeline logs. I also review access periodically and retain deployment and approval records for auditability.

## Q: How do you identify which components changed between two releases?

I use Git history, pull requests and work-item references to establish approved source changes, then compare the relevant source and target states in Gearset. I review the component list and dependencies to distinguish intended release changes from unrelated differences or environment drift. This confirms release scope and supports deployment planning and recovery.

## Q: How do you verify a successful production deployment?

I review the Gearset deployment result and validation or Apex test output. I confirm that intended metadata was deployed to the correct production org, then run smoke tests for critical business journeys. I check relevant permissions, Flows, integrations and post-deployment tasks, and ask QA or the business owner to confirm expected behavior. I communicate completion and record follow-up issues.

## Q: What do you include in a production release runbook?

I include release scope, pull requests and work items, target org, deployment order, dependencies, owners, approvals, validation results, pre-deployment tasks and execution steps. I also document smoke tests, rollback or forward-fix criteria, recovery steps, escalation contacts and communication checkpoints. After the release, I record the outcome, issues, verification results and follow-up actions.

## Q: How do you manage weekly fast-track releases alongside a regular release cycle?

I maintain a release calendar with clear cutoffs, testing windows and approval checkpoints. Fast-track changes are kept small and limited to approved, time-sensitive work. The regular release carries the broader scope that has completed the agreed QA and UAT process. Both tracks follow required production controls, and any emergency exception is documented and approved.

## Q: What do you do if an emergency production change was made outside Gearset or CI/CD?

I do not overwrite a working emergency fix without understanding its impact. I capture the exact production change, confirm the business outcome and compare production with the source-controlled baseline. I then create a reviewed corrective or hotfix change in Git, validate and test it, and reconcile it into production, release and development branches as appropriate. This prevents a later deployment from removing the emergency fix.

## Q: How do you prevent a production hotfix from being overwritten by the next release?

I make sure the hotfix is represented in source control and reconciled with the production baseline. I propagate it into the active release and development lines using the agreed merge or back-promotion process. Before the next deployment, I verify that the fix remains in the intended source state and run relevant regression tests.

## Q: How do you manage sandbox drift?

I compare affected environments with the source-controlled baseline and identify whether the difference is an intended configuration change, an emergency fix or an untracked manual change. I confirm ownership and business impact, then reconcile the approved state through the normal Git and deployment process. I avoid blindly overwriting environment differences because they may contain important fixes or configuration.

## Q: What is your approach to deploying Salesforce CPQ changes?

I identify metadata changes and related data or configuration dependencies. I review deployment order, required records, package dependencies and post-deployment steps. I validate the change in a lower environment using representative test data, then coordinate QA and business verification before production. The exact approach depends on the CPQ implementation and whether the change involves managed-package metadata, custom metadata or data records.

## Q: How do you use Gearset when several teams share the same Salesforce environments?

I keep release scope traceable to each team's pull requests and work items. I identify shared components and dependencies before promoting changes, and coordinate deployment windows so one team's release does not unintentionally overwrite another team's work. Where the branching and environment strategy supports it, I use separate release scopes or controlled batches. I make sure QA and business owners know which changes are included in each deployment.

## Q: How do you explain a release delay to business stakeholders?

I explain what is blocked, the business impact, the risk of proceeding without required checks and the options available. I give a realistic next update time and identify whether the affected change can be isolated from the rest of the release. I avoid promising a deployment time until technical and approval blockers are understood.
