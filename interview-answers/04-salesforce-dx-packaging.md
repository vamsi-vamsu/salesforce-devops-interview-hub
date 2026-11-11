# GitHub Actions

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: What is GitHub Actions?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is a workflow, job, and step?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What are GitHub Actions runners?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is the difference between GitHub-hosted and self-hosted runners?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: Where do you store credentials and secrets in GitHub Actions?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you configure environment-specific secrets?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you implement approval gates?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How would you migrate a Salesforce pipeline from GitLab to GitHub Actions?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you deploy Salesforce metadata using Salesforce CLI?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage managed packages and dependencies?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you implement automated tests and security checks?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you troubleshoot a failed GitHub Actions workflow?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._



## Q: How would you architect a CI/CD pipeline for deployments across multiple orgs using scratch and non-scratch orgs?

I separate development validation from environment promotion. Developers use scratch orgs for isolated feature work and automated pull-request validation, while persistent sandbox orgs support integration, QA, UAT and production-like validation. The pipeline authenticates to each org using a dedicated integration identity, detects changed metadata or packages, resolves dependencies, deploys in the required order and runs Apex tests. Promotion to shared environments is controlled by branch policy, approvals and release gates. I keep org-specific configuration and secrets outside the source package and maintain a clear mapping between orgs, branches and release stages.

## Q: What are the limitations of Salesforce DX, and how do you work around them?

Salesforce DX provides a source-driven development model, but not every metadata type or org configuration is equally source-friendly. Some metadata has dependencies, unsupported or incomplete retrieve/deploy behavior, large XML files can create noisy diffs, and org-specific settings or data may need separate handling. I address this with package boundaries, dependency-aware deployment order, source tracking where appropriate, manifest-based retrieval, targeted data seeding and documented manual steps for unsupported metadata. I also validate against the actual target org because a successful source conversion does not guarantee a successful deployment.

## Q: How do you manage destructive changes in an automated deployment?

I generate destructive changes from an explicit, reviewed list of components that should be removed. I separate pre-destructive and post-destructive manifests where ordering matters, check references and dependencies, and require peer review and approval for high-impact removals. I run validation against a representative target org and preserve the previous source version or deployment artifact for recovery. I do not infer deletion solely from a missing file in a partial checkout or an incomplete delta.

## Q: What is the difference between unlocked and managed packages, and when would you use each?

Unlocked packages are commonly used to modularize custom application components that the owning team expects to evolve through source control and package versioning. Managed packages are generally used to distribute protected or vendor-owned functionality with controlled upgrades and namespace boundaries. I choose based on ownership, distribution model, upgrade requirements, dependency management and whether the code must remain editable by the subscriber. I also account for package dependencies and version compatibility in the pipeline.

## Q: What is Org Shape, and how can it help Salesforce DX development?

Org Shape captures the metadata and feature configuration characteristics of a source org and can be used to create scratch orgs that more closely resemble that org's configuration. It helps reduce the gap between a generic scratch org and the features enabled in the target environment. It does not copy production data or replace explicit metadata and permission setup. I use it where supported, and still validate that the scratch org has the required settings and dependencies.

## Q: How do you optimize deployments in a monolithic Salesforce org with large Apex and Flow metadata?

I first measure deployment duration and identify whether the bottleneck is metadata volume, dependency sequencing, Apex tests, validation, or API limits. I use delta detection where safe, deploy only the required components, group related metadata by dependency and avoid unnecessary full deployments. For Apex, I review test selection and execution time without weakening required coverage. For Flows, I coordinate version changes, check references and test entry criteria and fault paths. Longer term, I look for package boundaries and modular ownership to reduce coupling.

## Q: What are best practices for test data management across Salesforce environments?

I use repeatable test data factories for Apex tests and controlled data seeding for integration and user acceptance testing. Test records should have stable external identifiers, known relationships and deterministic values. I keep data sets as small as possible while covering important business scenarios, isolate tests from existing org data, and mask sensitive data copied from production. I also define cleanup and refresh procedures so test environments remain reliable between release cycles.

## Q: How do you manage dependencies in modular Salesforce deployments?

I identify dependencies through package configuration, metadata references and deployment validation. I maintain a dependency map and deploy foundational components before dependent packages or metadata. In a package-based architecture, I build and install package versions in dependency order and record the exact version IDs used by each environment. I fail early when a required dependency is missing rather than allowing a later deployment step to fail with an unclear error.

## Q: How do you ensure reliable deployment of frequently changing Flows?

I treat Flow changes as source-controlled metadata. I review the XML diff, API version, trigger conditions, execution order, referenced fields and subflows. I avoid multiple teams editing the same Flow without coordination, validate the Flow in a representative sandbox and test positive, negative and fault scenarios. If the Flow is high impact, I use a controlled activation strategy and verify the active version after deployment.

## Q: What is a blue-green deployment or feature toggle approach in Salesforce?

A blue-green approach maintains two versions of an application path and switches traffic or user access from the old version to the new one after validation. Salesforce org metadata deployments are not universally atomic, so I would apply this pattern selectively using parallel components, routing or configuration controls where the architecture supports it. Feature toggles can use Custom Metadata or controlled configuration to enable functionality for selected users or groups. I plan the toggle, permissions, testing and rollback behavior before deployment rather than assume a single switch can reverse every metadata change.

## Q: What is source-driven development, and how does it improve release velocity?

Source-driven development makes version control the authoritative record of application changes. Developers work in branches, commit changes, review pull requests and promote tested versions through automated pipelines. This improves traceability, repeatability and collaboration, and reduces manual changes directly in shared orgs. Release velocity improves when the team can detect changes early, validate consistently and deploy smaller, well-understood units of work.

## Q: Which DevOps KPIs do you track for Salesforce delivery?

I track deployment frequency, lead time for changes, change failure rate, mean time to recovery, deployment success rate, validation failure rate, release duration and escaped defects. I also review pipeline duration, test execution time, rework caused by conflicts and the number of emergency changes. I use these metrics to identify bottlenecks and quality trends rather than optimize deployment speed at the expense of reliability.
