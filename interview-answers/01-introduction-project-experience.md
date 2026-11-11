# Introduction & Project Experience

> Interview preparation notes. Answers reflect the experience and implementation details shared by Vamsi. Where a topic is outside direct project experience, the answer is framed as conceptual knowledge or an approach rather than claimed hands-on delivery.

## Q: Tell me about yourself.

I have around 10 years of Salesforce experience, including about 7 years focused on DevOps and release engineering. I have worked across Copado, GitLab CI/CD, Azure DevOps, Jenkins, Tekton, Gearset and Salesforce DX. I built a Copado release framework supporting four production orgs, then helped modernize delivery through GitLab pipelines and a nine-package 2GP monorepo. My focus is reliable releases, automation, governance and helping development teams deliver safely.

## Q: Walk me through your current project.

I support a Salesforce DX monorepo spanning four Salesforce orgs and containing nine second-generation unlocked packages. GitLab CI/CD uses reusable YAML templates, Bash, Python and Salesforce CLI for authentication, source-difference detection, delta calculation, manifest generation, metadata validation/deployment and package installation. We have two long-lived branches, Develop and main. Developers create feature branches from Develop and raise merge requests to Develop. The MR pipeline runs Security, Deploy, Test and Review stages. It provisions a snapshot-backed scratch org, deploys metadata and dependencies in reference order, runs Apex tests and makes the org available to QA. If the MR title contains skip-snapshot, or the nightly snapshot is unavailable, it uses a regular scratch org. Failures are reported to Google Chat. After merge to Develop, changed packages are built and deployed sequentially to QA. For our biweekly release, covering around 60 to 90 tickets across four squads, we cut a release branch from Develop, raise an MR to main and manually run staging with package_list=all. The pipeline reuses the latest coverage-enabled package version where possible, otherwise creates one, and builds/deploys packages with dependency-aware parallelism. Package IDs are recorded in GitLab Wiki, Google Chat sends notifications, and sfdx-project.json is updated. Production promotion is a manual pipeline using the approved package names and IDs, followed by post-deployment validation, business-owner communication, tagging and release-branch cleanup.

## Q: Explain your Salesforce DevOps architecture.

We use Git as the source of truth for metadata and automation. Developers work in feature branches and raise merge requests. The CI pipeline validates changes, performs security and quality checks, creates or uses a scratch org, deploys metadata in dependency order and runs Apex tests. Approved changes are merged and promoted through controlled release stages, with production approvals, deployment evidence and post-deployment validation.

## Q: What are your responsibilities as a Salesforce DevOps Lead?

My responsibilities include defining branching and release standards, maintaining CI/CD pipelines, managing Salesforce org connectivity and credentials, automating validation and deployments, resolving deployment failures and coordinating releases. I also work with developers and QA on metadata dependencies, package versioning, test strategy and environment readiness. As a lead, I focus on governance, repeatability, mentoring and measurable improvements.

## Q: Describe your experience implementing Copado from scratch.

I helped implement the Copado release framework from the ground up. The setup included preparing and refreshing the sandboxes, registering Salesforce orgs, connecting Git, configuring environment connections and credentials, defining branch mappings and promotion paths, and establishing quality gates and approval controls. We also integrated supporting tools such as Jira and OwnBackup. After the framework was configured, we used Copado User Stories to track metadata changes through the release lifecycle. Stories were validated and promoted through lower environments, including SIT and UAT, with dependency and conflict checks at each stage. We grouped approved stories into production releases, performed pre-deployment readiness checks, deployed during the release window, completed post-deployment validation and communicated results. Production fixes were back-promoted so lower environments and branches remained aligned. The framework supported four production orgs and approximately 20 to 25 production releases per year.

## Q: Why did you move from Copado to GitLab CI/CD?

The two projects had different delivery models, so I would not describe it as simply replacing Copado with GitLab in the same implementation. Copado was used to establish a governed release-management framework with User Stories, promotion paths, quality gates and controlled multi-org releases. The GitLab project uses a source-driven Salesforce DX monorepo and 2GP packages, where we needed more control over delta detection, package versioning, scratch-org validation and dependency-aware automation. GitLab gives us reusable pipeline-as-code and flexibility to integrate Bash, Python, Salesforce CLI, security checks and package workflows. The choice of tooling follows the architecture and delivery requirements of each team.

## Q: What challenges did you face during the migration?

The main challenges were translating existing release governance into pipeline rules, handling metadata dependencies, reproducing environment promotion behavior and making authentication reliable. We addressed these by standardizing branch policies, creating reusable YAML and scripts, generating deployment manifests, ordering package and metadata deployments, and adding validation and security checkpoints. We migrated incrementally and used pipeline results to refine the process.

## Q: How many Salesforce orgs and environments do you manage?

My release-management experience includes supporting four production Salesforce orgs, with corresponding lower environments for development, QA and release validation. The exact environment topology varies by application, but the important controls are clear ownership, consistent branching, environment-specific configuration and a repeatable promotion path.

## Q: Explain your end-to-end Salesforce release management process.

For the Copado project, the process begins with a Copado User Story tied to the business work and its metadata changes. The developer commits the change to Git, and the story is validated in its source environment. We promote stories along the configured path through SIT and UAT, resolving missing dependencies or Git conflicts and checking quality gates at each stage. For production, we consolidate the approved stories, validate the complete release set, obtain approvals, confirm environment readiness and execute the deployment in the agreed window. We then perform post-deployment and business validation, communicate the outcome and retain release records. Any production-only fix is back-promoted to lower environments and active branches. For the GitLab project, developers branch from Develop and submit an MR. Security, Deploy, Test and Review stages validate the change in a snapshot-backed scratch org, with a regular-org path when skip-snapshot is specified or the nightly snapshot is unavailable. After merge, changed packages deploy to QA. A biweekly release branch is staged using package_list=all, then approved package names and IDs are promoted through a manual production pipeline.

## Q: What is your experience with Salesforce DX and 2GP packages?

I have worked with a Salesforce DX monorepo containing nine second-generation unlocked packages deployed across four Salesforce orgs. We use package versioning, dependencies and ancestor tracking to manage modular delivery. GitLab automation identifies changed source, calculates deltas, generates manifests, validates metadata and builds or installs packages in dependency-aware order. During staging, the workflow uses the latest coverage-enabled package version when possible or creates a new version. We capture package IDs in GitLab Wiki and update sfdx-project.json as part of release management.

## Q: How do you manage multiple Salesforce projects in a monorepo?

I organize the repository into clear package or project directories and define dependencies explicitly. The pipeline identifies changed files, maps them to the affected packages and required dependents, then builds or deploys in dependency order. Shared scripts and reusable pipeline templates reduce duplication. I also keep package ownership, versioning and release notes clear so changes remain traceable.

## Q: Which DevOps tools have you used, and where does each fit?

I have worked with Copado and Gearset for Salesforce release management, GitLab CI/CD and Azure DevOps for pipeline automation, Jenkins and Tekton for CI orchestration, and GitHub and Bitbucket for source control. Salesforce CLI, Bash and Python support metadata operations and automation. The tool is only one part of the solution; source control, testing, security, approvals and deployment governance must work together.

