# GitHub Actions: Salesforce CI/CD Implementation

> **Project context:** Based on the workflow and script review of `vamsi-vamsu/salesforce-devops-2026`. This question bank covers two distinct deployment paths: PR-driven validation and Quick Deploy, and release/tag-driven deployment. It distinguishes current workflow behavior from recommended hardening.

## Q: Explain your GitHub Actions Salesforce CI/CD architecture.

I designed the implementation around reusable workflows and two complementary deployment models.

The first is PR-driven. When a pull request is opened or updated, an environment-specific workflow calls shared workflow logic. The pipeline checks out the source, sets up Salesforce CLI and the required plugins, authenticates to the target org, calculates the metadata delta, runs code-quality checks and performs a Salesforce validation deployment with the selected Apex tests. The validation result is associated with the PR. Once the change is reviewed and merged, a separate workflow can use the successful validation for Quick Deploy when Salesforce confirms it is still eligible.

The second is release-driven. A Git tag or a published GitHub Release starts a release workflow. It resolves the current and previous release references and invokes a reusable delta-deployment workflow. That workflow calculates the change set, creates the deployment manifests, authenticates to the selected GitHub Environment and deploys the release with the configured test level.

The PR flow gives developers feedback before merge. The release flow provides a versioned promotion mechanism and a record of what changed between releases.

## Q: What are the two main deployment scenarios in your GitHub Actions implementation?

There are two independent flows:

1. **PR-driven validation and Quick Deploy:** Validate the changed metadata before merge. When the PR is merged, use the validation job ID for Quick Deploy where eligible, avoiding a second Apex test execution.
2. **Release/tag-driven deployment:** A release tag or published GitHub Release identifies the release boundary. The workflow calculates the delta from the previous release reference and deploys it to the selected environment.

PR validation is change-centric and gives early feedback. The release workflow is release-centric and deploys a versioned set of changes.

## Q: Explain the PR validation workflow step by step.

When a developer opens or updates a pull request, the environment-specific caller workflow invokes shared workflow logic. The pipeline checks out the source revision, installs or configures Salesforce CLI and the required plugins, and authenticates to the target org using JWT-based authentication.

It then calculates the delta between the appropriate base and PR revision. The changed metadata is passed through configured quality checks, including Salesforce Code Analyzer and SonarQube where configured. The pipeline performs a check-only Salesforce deployment and runs the Apex tests selected for that PR. The custom coverage validation script evaluates the deployment and test results against the configured per-class threshold.

The validation result is associated with the PR. Once the change is reviewed and merged, the merge workflow can attempt Quick Deploy with the validation job ID. If that validation is no longer eligible, the workflow should use a fresh normal deployment path rather than bypassing validation.

## Q: How are the GitHub Actions workflows organized?

The repository separates shared logic from event- and environment-specific workflow entry points. The common PR workflow template is in `.github/workflows/template.yaml`. Environment-specific caller workflows provide the context for staging, UAT and production. The reusable delta deployment logic is in `.github/workflows/reusable-delta-deploy.yml`, while release/tag entry points include `release-based-deploy.yaml` and `tag-based-deploy.yaml`.

There are also supporting workflows for CI, validation, dispatch and environment-specific deployment. This structure avoids copying the same Salesforce setup and deployment steps into every workflow. When explaining a particular trigger or environment, I refer to the corresponding caller YAML rather than describing every workflow as if it runs for every event.

## Q: Why did you use reusable workflows instead of duplicating YAML?

I want authentication, CLI setup, delta generation, validation and deployment behavior to remain consistent. Reusable workflows centralize that common logic. The caller workflow supplies environment-specific inputs, such as the target GitHub Environment and deployment configuration.

If I need to change a common validation step or update a Salesforce CLI command, I can make the change in one place and have the callers use the same implementation. It also makes differences between environments explicit instead of hiding them in duplicated YAML.

## Q: How do you authenticate GitHub Actions to Salesforce?

The workflow uses JWT-based authentication with Salesforce CLI. The Salesforce External Client App is configured with the public certificate, while the corresponding private key is stored as a protected GitHub secret. At runtime, the workflow supplies the required username, client ID, key and login URL to the Salesforce CLI JWT login command.

The private key must never be printed, checked into source control or included in an artifact. GitHub Environment scoping and repository permissions should restrict which workflows can access production credentials. I also avoid exposing credential values in debug output.

## Q: Why did you use JWT authentication instead of username and password?

JWT authentication is suitable for unattended CI/CD because it avoids storing an interactive user's password and security token in the workflow. The External Client App establishes trust through the certificate, and the pipeline uses a signed JWT to authenticate.

I still apply least privilege. The integration user should have only the permissions required to validate and deploy metadata, and the private key should be stored in a protected secret with restricted access. JWT does not remove the need for credential rotation, auditing or environment protection.

## Q: How do you handle secrets in GitHub Actions?

Sensitive values such as the Salesforce private key are stored as GitHub secrets rather than hardcoded in YAML. Environment-specific credentials should be stored at the appropriate GitHub Environment scope so production secrets are not available to lower-environment workflows. The workflow references secrets only at runtime.

I also review scripts and sample command files for accidental credential exposure. Any real key or password committed to Git must be considered compromised, removed from the repository history where appropriate, and rotated. A client ID or username is not the same as a private key, but it should still be handled thoughtfully.

## Q: How do you generate a delta deployment in the PR workflow?

The primary PR deployment flow uses SFDX Git Delta to compare the appropriate base and head references. The goal is to produce a manifest containing only the added or modified metadata in the PR, rather than deploying the entire repository.

The chosen references matter. The comparison must represent the PR's complete change relative to the target branch. The workflow should use consistent commit references and ensure that the comparison is performed against the correct base revision. The delta is then used for validation and the corresponding deployment path.

The repository also contains a separate custom Python utility, `sf_delta.py`. I describe that as an additional delta-generation utility, not as the primary implementation used by every PR workflow.

## Q: How does your custom Python delta utility work?

The `sf_delta.py` utility compares two Git commits and classifies Salesforce metadata as added, modified or deleted. It maps source paths to metadata types and members, copies changed components while preserving the Salesforce DX directory structure, and generates a `package.xml` for additions and modifications plus a destructive manifest for deletions.

It also handles bundle-style metadata such as Lightning Web Components, Aura and ExperienceBundle files, including associated companion files. It considers ignore configuration such as `.sfignore`, `.sgdignore` and `.forceignore`. This utility is separate from the SFDX Git Delta action used in the main PR workflow.

## Q: How do you handle deleted metadata in a delta deployment?

Deleted metadata cannot be represented as an ordinary member in the positive deployment manifest. The delta process generates a destructive-changes manifest for the deleted components.

In the PR flow, the deletion set should be included in the check-only validation and then applied during the merge deployment in the correct sequence. The implementation uses separate destructive processing so that destructive changes can be ordered relative to the main deployment. In the release flow, destructive changes are intended to be applied after the main deployment succeeds.

A release containing only deletions still has work to deploy even if its positive package manifest is empty. The release workflow must evaluate both manifests rather than skip the entire deployment just because there are no additions or modifications.

## Q: What is the difference between pre-destructive and post-destructive changes?

Pre-destructive changes are deletions that must happen before the main metadata deployment, often because the target org needs an obsolete component removed before a replacement can be deployed. Post-destructive changes are applied after the main deployment, which is useful when the replacement metadata must exist before the old component is removed.

The correct order depends on the metadata dependency and Salesforce deployment behavior. I do not put every deletion into one bucket automatically. I review the impacted components and use the correct manifest and deployment option for the intended sequence.

## Q: How do you handle companion files and metadata bundles in delta deployment?

I make sure the delta includes the complete metadata component, not just an isolated source file. For example, an Apex class has its companion metadata file, and bundle-based metadata such as LWC and Aura has a directory structure that must be preserved. The delta utility includes companion files and bundle contents when identifying and copying changed components.

This avoids incomplete deployments where the implementation file is included but its required metadata or bundle members are omitted.

## Q: How do you handle metadata dependencies in delta deployment?

A delta identifies what changed. It does not automatically know every dependency that a changed component has on metadata already in the org or on other components in the same change.

For example, an Apex class may reference a newly created custom field. Both components must be available in the target org, and the deployment plan must account for their dependency. I address this through dependency-aware sequencing, metadata grouping and validation in lower environments. When the change is too interconnected or the dependency impact is uncertain, I would use a broader deployment rather than force a narrow delta.

Code Analyzer, SonarQube and Apex tests reduce risk, but they do not replace dependency analysis.

## Q: What happens if the delta contains only destructive changes?

The positive package manifest may be empty while the destructive manifest contains deletions. The workflow must not interpret an empty positive manifest as an empty release. It should detect whether either manifest contains deployable work and invoke the appropriate Salesforce deployment operation.

During my review, I identified this as an edge case to close in the release flow. I would explicitly test a deletion-only release so the destructive manifest is processed even when there are no additions or modifications.

## Q: Explain the Quick Deploy approach in your PR workflow.

A validation is a check-only deployment of a specific delta, at a specific commit, against a specific org. The validation job ID is only meaningful for that combination.

On a PR, the workflow validates the delta and posts the validation ID as a hidden comment. When a separate merge workflow runs, it reads that PR comment and attempts Quick Deploy with the ID. Before using it, the merge workflow should confirm that the record belongs to the expected PR, commit and target environment, and that Salesforce still considers the validation eligible. The validation must have succeeded, the required tests must have run and it must still be inside Salesforce's validity window.

Quick Deploy relies on the earlier validation test results and does not run those tests again. If Salesforce rejects the validation ID or it is no longer eligible, the workflow should fall back to a normal deployment with the same intended test selection. The normal deployment must still satisfy the target org's deployment test requirements.

## Q: Why did you use a hidden PR comment to pass the validation ID instead of workflow artifacts?

The PR is the natural link between the validation event and the later merge event. A comment can be looked up by PR number, so the merge workflow has a straightforward place to find the validation record.

An artifact is another option, but the merge workflow would need to identify the correct validation run, retrieve its artifact through the API and account for artifact retention. The comment is simpler to associate with the PR. The trade-off is that comments are editable, so the merge workflow must validate that the comment came from the Actions bot and that its recorded ID, commit SHA and target environment match the expected values. A commit status, check run or Deployments API record are other possible handoff mechanisms.

## Q: How do you ensure the Quick Deploy validation belongs to the latest PR commit?

I associate the validation ID with the exact commit SHA that was validated and the target environment. When the merge workflow runs, it checks the recorded SHA against the commit that the validation is intended to cover and verifies the target environment.

I am careful not to assume that a PR head SHA and a synthetic merge commit SHA are always identical. The workflow must record which revision it actually validated and compare that with the final merge context correctly. If the association does not match, I reject the old validation and require a fresh validation or a normal deployment path. Branch protection should require the validation check for the latest commit.

## Q: What happens if a developer pushes another commit after validation succeeds but before merge?

The new commit changes the PR state, so the earlier validation must not automatically be reused. The PR workflow should run again, generate the updated delta and validate the new revision. The validation record is tied to the commit SHA and target org. When the merge workflow runs, it verifies that association. If the SHA does not match, it rejects the old validation and uses a fresh validation or normal deployment path according to the release design.

## Q: What is the difference between a validation deployment and Quick Deploy?

A validation is a check-only deployment. Salesforce evaluates whether the metadata can be deployed and runs the configured tests, but it does not commit the metadata changes to the org.

Quick Deploy takes an eligible successful validation and commits that validated change without running the Apex tests a second time. It is useful for reducing deployment time, but it depends on the earlier validation remaining eligible and matching the intended deployment context.

## Q: Does Quick Deploy execute Apex tests again?

No. Quick Deploy uses the test results from the successful validation. That is why test selection, test execution and coverage checks must be correct during validation. If the validation cannot be reused, a normal deployment is required and the configured tests execute as part of that deployment.

## Q: What happens when a Quick Deploy validation expires or becomes ineligible?

Salesforce may reject Quick Deploy when the validation is outside its validity window or otherwise no longer eligible. The workflow should capture the failure reason and use the normal deployment path with the same intended test selection. That path is slower and performs validation/deployment checks again.

For important releases, I would avoid depending on an old validation. I would schedule validation close enough to the release window and alert when the pipeline has to fall back, so repeated eligibility failures can be investigated.

## Q: How do you handle multiple PRs targeting the same Salesforce org?

Each PR has its own validation record, so the validation IDs do not collide. The bigger risk is that one PR can change the org after another PR has been validated. The second validation may no longer represent the combined state that will be deployed.

I would address this with a merge queue or an up-to-date branch requirement so the PR is revalidated against the latest target branch. I would also serialize deployments to the same target org with a GitHub Actions concurrency group. These controls complement each other: concurrency serializes deployment activity, while revalidation ensures the change was tested against the current base.

## Q: How do you prevent an unvalidated PR deployment from reaching production?

The PR flow should require successful validation on the latest commit before merge. I would enforce that through branch protection or repository rulesets, with required status checks and required reviews. Production GitHub Environment protection adds an approval gate before the production job can access its credentials and deploy.

I distinguish this from the release/tag flow. The current release/tag workflow is a direct release deployment path and does not have the same PR validation-to-Quick-Deploy handoff. I would close that gap by requiring the release commit to have a successful validation, or by adding a validate-then-deploy sequence to the release workflow.

## Q: Explain your release/tag-based deployment workflow.

A release can be initiated by pushing a tag that follows the `releases/X.Y.Z` pattern or by publishing a non-prerelease GitHub Release with a `v` tag. The entry workflow resolves the current release tag and the previous release reference, then passes those references to the shared reusable delta deployment workflow.

The reusable workflow calculates the difference between the two refs, creates the package and destructive manifests, authenticates to the selected GitHub Environment and deploys the changes with the configured Apex test level. It then writes a deployment summary so the release has a record of the components included.

This release path is separate from the PR validation and Quick Deploy path. It is intended to deploy a defined release boundary rather than an individual PR delta.

## Q: How do you identify the previous release tag?

The release workflow uses Git history and `git describe` to find a reachable tag while walking back from the parent of the release commit. In the tag-push workflow, the tag pattern is restricted to `releases/*`, which avoids unrelated tags being considered by that caller.

However, finding a reachable tag does not prove that it was the last version successfully deployed to production. For a stronger release baseline, I would use a release-only tag lookup, verify ancestry and record the last successfully deployed production tag. The workflow should fail closed if it cannot resolve a valid baseline.

## Q: How do you prevent the wrong tag from being selected as the release baseline?

I would apply the same release-only selection logic in both release entry points and exclude prerelease or unrelated tags. I would verify that the baseline tag is an ancestor of the release commit and corresponds to the last successfully deployed production version. I would also fail the workflow if the baseline is missing rather than silently treating the release as a first deployment.

The repository review identified that the release-based caller needs the same filtering discipline as the tag-push caller. This is a hardening item, not something I would claim is already enforced in both paths.

## Q: How do you prevent a tag push and a published GitHub Release from deploying the same release twice?

I would use one canonical release trigger, or configure a shared concurrency group for the target environment and release identity. That prevents two workflows from attempting the same production deployment concurrently.

I would also make the deployment idempotent where possible and record the commit SHA and release tag associated with a successful deployment. That way, a retry or duplicate trigger can be recognized rather than silently deploying the same release twice.

## Q: How do GitHub Environments and approvals protect production?

A GitHub Environment gives a workflow a named deployment target and provides a place to configure environment-specific secrets and protection rules. For production, I would require authorized reviewers, restrict which branches or tags can deploy and keep production credentials scoped to that environment.

The workflow must reference the production Environment so that the protection rules are actually evaluated. An approval gate is not effective if the deployment job does not target the protected Environment.

## Q: How would you improve release visibility before production approval?

I would split the release workflow into a plan phase and a gated deployment phase. The plan phase would resolve immutable commit references, generate the delta and both manifests, and publish a summary of the components and destructive changes. Approvers could review that plan before the deployment job requests access to production secrets and starts the deployment.

This improves change visibility and makes the approval meaningful because reviewers can see what will be deployed, not only the release tag.

## Q: How do you handle release tags and immutable references?

I resolve a release tag to its commit SHA and use that immutable SHA for the deployment and audit record. The previous baseline should also resolve to a specific commit. This avoids ambiguity if a tag is moved or a branch advances after the release starts.

For production governance, I would protect release tags from being moved or deleted and record the deployed tag, commit SHA, target org, workflow run and Salesforce deployment result.

## Q: How do you handle a hotfix in the GitHub Actions model?

The safe approach is to branch from the commit currently deployed to production, make the smallest corrective change, and run the same security, delta, validation and test checks before production approval. After the fix is deployed, I reconcile it into the active development and release branches so a later release does not reintroduce the defect.

The repository contains a workflow named `sf_hotfix.yaml`, but I would verify its exact trigger and behavior before claiming that every step of this operating model is already automated there. The branching, validation and back-promotion points above describe the release-control approach I would use.

## Q: How do you roll back a failed Salesforce metadata deployment?

I first inspect the Salesforce deployment result to determine whether the deployment failed before commit or whether a successful deployment introduced a runtime issue. For a metadata deployment, I generally prefer a reviewed forward fix or a Git revert that restores the intended metadata state, followed by a new validated deployment.

A rollback is not always a simple reversal. Some metadata changes have dependencies, and data changes or destructive operations may not be reversible by redeploying an older package. I identify the affected components, check dependencies and data impact, choose a safe recovery plan, validate it in a lower environment where possible and then use the approved production process.

## Q: How do you capture deployment traceability?

I want each deployment to be traceable to the repository, commit SHA, PR or release tag, workflow run, target GitHub Environment, Salesforce deployment result and test outcome. The release workflow writes a summary of deployed components. I would also retain the deployment logs and validation identifiers for the required audit period, without storing secrets or sensitive customer data in artifacts.

## Q: How do you troubleshoot a failed GitHub Actions deployment?

I start with the failed job and identify whether the issue is in checkout, Salesforce CLI setup, authentication, delta generation, static analysis, validation, Apex tests, destructive processing or the deployment itself. I inspect the workflow run and job logs, confirm the target org and commit references, and check whether the package and destructive manifests contain the expected components.

For Salesforce errors, I use the deployment ID and test results to identify component failures, dependency issues or test failures. I also check whether the failure is caused by an invalid validation ID, a missing secret, an expired credential or an API/version mismatch. I correct the root cause and rerun the appropriate validation or deployment step rather than repeatedly rerunning the whole workflow without understanding the failure.

## Q: How do you select Apex tests from the pull request?

The PR body can specify Apex test classes using the documented marker format, such as `APEX TEST CLASS TO RUN [RUN:TestClassA,TestClassB]`. The helper script `READ_PRBODY.py` reads the PR body and extracts the specified classes. The workflow can then pass the selection to the Salesforce deployment command.

The PR can also request relevant tests or use the configured default test level. I make sure the selected tests are actually passed to the deployment command and that the chosen Salesforce test level is appropriate for the target org. Test selection is part of the validation contract, not just a text field for documentation.

## Q: What does the custom code coverage script validate?

The `CODE_COVERAGE.py` script evaluates Salesforce deployment and test result output. It checks component failures and test failures and enforces a per-class coverage threshold. The reviewed script uses an 85% threshold, while the repository documentation has referred to 82% in places. I would align the threshold and documentation before describing a single value as the definitive configured gate.

I would also make the script fail closed when the expected deployment result file is missing. A missing result should not be interpreted as a successful coverage check. This is an important reliability improvement for a quality gate.

## Q: How do you integrate Salesforce Code Analyzer and SonarQube?

The PR workflow includes static analysis stages so developers receive feedback before their changes are merged. Salesforce Code Analyzer focuses on Salesforce-specific code quality and security rules, while SonarQube provides broader static analysis and quality reporting. I treat these checks as complementary to Apex tests: static analysis detects code patterns and potential defects, while tests validate behavior.

The pipeline's exit behavior must match the desired policy. A finding should fail the pipeline only when it violates the configured blocking rules. I verify the actual severity threshold and exit code behavior in YAML rather than assuming every finding is a blocking failure.

## Q: What would you improve in the current GitHub Actions implementation?

I would prioritize these improvements:

1. Bind every Quick Deploy validation record to the exact validated commit SHA, PR and target org, and verify the association before use.
2. Require latest-commit validation through branch protection or a merge queue, and serialize deployments per target org.
3. Standardize release tag resolution, fail closed when the previous production baseline is missing, and verify that it is an ancestor of the release commit.
4. Prevent duplicate deployment when both tag-push and GitHub Release triggers refer to the same release.
5. Make release plans visible before production approval, including positive and destructive manifests.
6. Correct the deletion-only release edge case and verify the ordering and command options for destructive deployments.
7. Make quality gates fail closed when expected result files are missing and align the coverage threshold across scripts and documentation.
8. Verify that static-analysis failures actually return a non-zero exit status when configured as blocking gates.

I would deliver these incrementally, with tests for each failure scenario, rather than changing all release controls at once.

## Q: How would you scale this GitHub Actions architecture across multiple Salesforce orgs?

I would keep common Salesforce setup and deployment logic in reusable workflows and use thin caller workflows for each target environment. Each environment would have its own secrets, protection rules, deployment permissions and explicit configuration. I would serialize deployments to the same org while allowing independent org deployments to run in parallel.

I would also standardize artifact and deployment summaries, immutable commit references, release baselines and monitoring. That provides consistent governance without forcing every org to share the same release schedule or approval model.

## Q: How do GitHub Actions concurrency groups help in Salesforce deployments?

A concurrency group controls how many workflow runs or jobs with the same group key can run at the same time. I would create the group from the target org or environment so two deployments cannot modify the same Salesforce org concurrently.

Concurrency prevents overlapping writes, but it does not replace validation. A PR may still have been validated against an older target branch or org state. I pair deployment serialization with a merge queue or latest-commit validation so the deployment is both orderly and based on the current change set.

## Q: What is the difference between GitHub Actions artifacts, caches and deployment environments?

Artifacts are files produced by a workflow run, such as logs, test reports, manifests or deployment summaries. They can be downloaded by later jobs or workflows while they are retained.

Caches are intended to reuse dependencies or other repeatable build inputs to speed up later runs. They should not be used as the authoritative store for deployment state or secrets.

Environments represent deployment targets and can provide environment-scoped secrets and protection rules such as required reviewers. For Salesforce deployments, I use environments to separate target org configuration and production approvals, artifacts for non-secret deployment evidence, and caches only for safe reusable dependencies.

## Q: How do you make a GitHub Actions workflow secure?

I apply least privilege to the workflow's `GITHUB_TOKEN` permissions, avoid granting write access unless required, pin third-party actions to trusted immutable versions or commit SHAs, and protect production secrets with GitHub Environments. I validate all values that cross workflow boundaries, especially PR comments and user-controlled PR body text.

For pull requests from forks, I avoid exposing deployment secrets to untrusted code. I also review shell commands for injection risks, use quoted variables and safe argument handling, keep credentials out of logs and artifacts, and restrict who can modify production workflow files.

## Q: How do you compare GitHub Actions with GitLab CI/CD and Copado?

GitHub Actions and GitLab CI/CD are code-driven automation platforms. Both allow us to define workflows or pipelines in YAML, run validation and deployment jobs, manage secrets and approvals, and integrate with Git branching and pull requests.

Copado is a Salesforce-focused release-management platform with concepts such as User Stories, promotion paths, back promotions and Salesforce release governance built into the product. With GitHub Actions, I implement more of that orchestration directly in workflow YAML and scripts. The trade-off is flexibility and control versus the need to own more of the operational logic, guardrails and maintenance.

## Q: What is your role as the DevOps lead in this GitHub Actions implementation?

My role is to design the workflow architecture, keep the PR and release deployment paths consistent, define authentication and environment boundaries, and make sure validation and deployment are traceable. I review the reusable workflow design, delta-generation approach, testing strategy, quality gates and release controls. I also identify edge cases such as stale validations, concurrent deployments, destructive-only releases and incorrect release baselines.

At a lead level, I focus not only on making the deployment command work, but also on making the process predictable, secure, auditable and supportable by the wider engineering team.

## Q: What should you never claim unless you verify it in the workflow configuration?

I should not claim that concurrency groups, merge queues, branch protection, required status checks, a production validation gate in the tag workflow, or a last-successful-production baseline are implemented unless I can point to their configuration. I should also verify the exact test-level behavior, coverage threshold, Code Analyzer failure condition and destructive-change ordering before describing them as enforced controls.

My interview explanation separates implemented behavior, gaps identified during review and improvements I recommend. That makes the architecture discussion accurate and defensible.
