# Security, Authentication & Compliance

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: How does JWT-based authentication work in Salesforce CI/CD?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you authenticate Salesforce CLI from a pipeline?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage connected apps?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage certificates and private keys?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: Where should Salesforce credentials be stored?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you prevent secrets from appearing in pipeline logs?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you implement security scanning in CI/CD?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you control access to production deployments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you implement approval workflows?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you maintain deployment audit trails?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you support compliance and release governance?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._


## Q: How do you secure a CI/CD pipeline that deploys to production?

I use least-privilege access, protected branches, required reviews and successful CI checks. Deployment credentials are stored in an approved secret manager or protected pipeline variables, never in source code or logs. Production deployments are restricted to authorized users and require the configured approvals. I also review audit logs, credential rotation and access permissions periodically.

## Q: How do you handle authentication failures in a Salesforce CI/CD pipeline?

I check the Salesforce authentication configuration, connected app or OAuth setup, certificate validity, integration-user permissions and target org identity. I confirm that the pipeline retrieves the correct secret and that credentials or certificates have not expired or been rotated without updating the pipeline. I avoid printing credentials in logs. After correcting the cause, I rerun authentication and validate the target org before deployment.


## Q: How do you integrate PMD or CodeScan into GitHub Actions or Azure DevOps?

I add static analysis as a dedicated CI job or pipeline task. The job checks out the repository, installs or invokes a pinned scanner version, runs the Apex analysis against the configured source scope and publishes the report as a build artifact. I configure the pipeline to fail on the agreed severity threshold and make the results visible to reviewers. Secrets are not required for a basic local PMD scan, but any hosted scanner token is stored in the platform's protected secret store. The same ruleset should be version-controlled so GitHub Actions and Azure DevOps apply consistent checks.

## Q: How do you version-control Profiles and Permission Sets when multiple admins are working in parallel?

I retrieve Profiles and Permission Sets into source control and use feature branches and pull requests so changes are reviewed before integration. I prefer permission sets and permission set groups for additive access management rather than repeatedly making broad changes to profiles. I pay particular attention to profile XML because it can contain many unrelated settings and produce noisy conflicts. I coordinate ownership, review the actual diff, resolve conflicts carefully and validate the effective access in a lower environment.

## Q: In how many ways can you provide field-level security in Salesforce?

Field-level security is primarily controlled through Profiles and Permission Sets, including Permission Set Groups. A profile provides a user's baseline field access, while permission sets grant additional access. Permission Set Groups combine permission sets for job-based access, and muting permission sets can remove selected permissions from a group. Page layouts can hide or make fields read-only in the UI, but they are not a replacement for field-level security because they do not enforce access across APIs and other entry points.

## Q: How do you monitor and audit deployments across multiple Salesforce orgs?

I retain the pipeline execution, commit or package reference, source and target org, deployment result, validation and test results, approver, timestamp and release identifier. I use pipeline artifacts, Git history and the release tool's execution history as complementary evidence. I also send failure and completion notifications to the team's agreed channel. For production, I make sure the release record links the change request, approval and post-deployment verification.

## Q: How do you enforce peer review and change tracking for low-code changes such as Flows?

I retrieve Flow metadata into source control and require changes to move through feature branches and pull requests like code changes. Reviewers inspect the Flow XML or metadata diff, confirm entry criteria, execution order, fault paths, permissions and dependencies, and request tests for the affected business scenarios. I include validation and regression testing in the deployment process. Low-code changes still require source control, peer review and traceability because they can have the same production impact as Apex.
