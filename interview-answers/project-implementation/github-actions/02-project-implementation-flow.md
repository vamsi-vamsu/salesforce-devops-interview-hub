# GitHub Actions Salesforce CI/CD Project Implementation Flow

> Step-by-step reference for explaining the GitHub Actions implementation in `vamsi-vamsu/salesforce-devops-2026`. It documents two separate deployment models: PR-driven validation and Quick Deploy, and release/tag-driven deployment. Keep implemented behavior separate from recommended hardening.

## Architecture overview

The implementation uses GitHub Actions with reusable workflows to avoid duplicating Salesforce setup and deployment logic.

- **PR workflow template:** `.github/workflows/template.yaml`
- **Environment-specific PR callers:** staging, UAT and production workflow files
- **Reusable release delta workflow:** `.github/workflows/reusable-delta-deploy.yml`
- **Release entry points:** `.github/workflows/release-based-deploy.yaml` and `.github/workflows/tag-based-deploy.yaml`
- **Supporting automation:** Salesforce CLI setup, Python helpers, delta generation, test selection, coverage evaluation and deployment summaries

The two main flows serve different purposes. PR validation checks a proposed change before merge and can use Quick Deploy after merge. Release/tag deployment calculates the delta between release references and deploys that release to the selected GitHub Environment.

## Branch and pull request workflow

1. **Create a feature branch:** Developers create a feature branch from the target integration or release branch according to the repository branching policy.
2. **Make Salesforce changes:** Developers update Salesforce DX source and commit the metadata changes.
3. **Open or update a pull request:** The PR triggers the environment-specific caller workflow, which invokes the shared template.
4. **Select the target environment:** The caller supplies the target environment and configuration. The same reusable workflow logic is shared while the target-specific context remains explicit.
5. **Prepare the runner:** The workflow checks out the source revision, sets up Salesforce CLI and the required plugins, and prepares the scripts used by the pipeline.
6. **Authenticate to Salesforce:** The workflow uses JWT-based Salesforce CLI authentication. The External Client App has the public certificate, and the private key is supplied through protected GitHub secrets.
7. **Calculate the metadata delta:** SFDX Git Delta compares the appropriate base and PR references and creates the package manifest for the changed metadata. The comparison must represent the complete PR change relative to its target branch.
8. **Run quality checks:** The PR path runs configured static analysis, including Salesforce Code Analyzer and SonarQube checks.
9. **Determine Apex test selection:** The PR body can specify test classes using the documented marker, request relevant tests, or use the configured default test level. `READ_PRBODY.py` extracts the requested class names for the workflow.
10. **Run check-only deployment:** Salesforce validates the changed metadata against the target org and executes the selected Apex tests. No metadata is committed during this validation step.
11. **Evaluate test and coverage results:** `CODE_COVERAGE.py` evaluates the deployment/test result and the configured per-class coverage policy. Confirm the threshold in the active script and documentation before quoting a fixed percentage, because the reviewed source and documentation have shown inconsistent values.
12. **Publish validation evidence:** After successful validation, the workflow associates the Salesforce validation job ID with the PR using a hidden PR comment. The record should include the validated revision and target environment so a later workflow can verify the association.
13. **Complete PR review:** Reviewers inspect the code and validation checks. Branch protection or repository rules should require the appropriate checks before merge.

## Merge and Quick Deploy workflow

1. **Merge the approved PR:** A separate workflow is triggered when the PR is closed and merged.
2. **Read the PR validation record:** The merge workflow reads the hidden validation comment associated with that PR.
3. **Verify the validation context:** Before using the ID, verify that it was posted by the expected Actions identity, belongs to the PR, records the revision that was actually validated, and names the same target environment.
4. **Check Salesforce eligibility:** Salesforce must still consider the validation successful and eligible for Quick Deploy. The required tests must have run, and the validation must be within Salesforce's validity window.
5. **Run Quick Deploy:** When eligible, use `sf project deploy quick` with the validation job ID. Quick Deploy commits the validated metadata without running the Apex tests a second time.
6. **Handle an ineligible or expired validation:** If Quick Deploy cannot be used, fall back to a normal deployment with the same intended test selection. The normal deployment must still satisfy the target org's test and coverage requirements.
7. **Keep PR records isolated:** Each PR has its own validation record. Older validation comments should be superseded or cleaned up so that the merge workflow does not accidentally select an obsolete record.
8. **Record the deployment outcome:** Preserve the deployment result and communicate success or failure through the workflow's reporting mechanism.

### Quick Deploy controls and concurrency considerations

- A validation is specific to a particular metadata delta, source revision and target org. The job ID must not be treated as a generic approval token.
- A later commit to the PR requires a new validation. The merge workflow must reject a validation record whose SHA does not match the revision it is intended to deploy.
- Two PRs can each have valid validation records but still interfere if one changes the org before the other deploys. Requiring the branch to be up to date or using a merge queue helps ensure revalidation against the latest base.
- A GitHub Actions concurrency group per target org can serialize deployments. This is recommended hardening unless it is explicitly configured in the workflow.
- Branch protection, required checks and production environment approvals must be verified in repository settings. Do not claim them as implemented based only on workflow YAML.

## Delta generation and destructive changes

1. **Generate the positive package manifest:** SFDX Git Delta identifies added and modified metadata for the PR or release comparison.
2. **Generate destructive manifests:** Deleted metadata is represented separately because it cannot be deployed as an ordinary positive package member.
3. **Include complete metadata components:** Companion files and bundle structures must travel with the changed component, including Apex metadata files and Lightning bundle contents.
4. **Validate destructive changes:** In the PR flow, destructive changes are included in the check-only validation and are applied in the merge deployment in the intended sequence.
5. **Sequence the deployment:** Use pre-destructive or post-destructive processing based on component dependencies. Do not assume every deletion belongs after the main package.
6. **Skip empty operations safely:** Skip a destructive pass only when its manifest is empty. In the release flow, a deletion-only release must still deploy even when the positive package manifest is empty.

The repository also contains `sf_delta.py`, a separate custom utility that compares Git commits, classifies metadata changes, preserves Salesforce DX paths and generates positive and destructive manifests. It supports bundle-style metadata and ignore files. The main PR path uses SFDX Git Delta; do not describe the Python utility as the primary delta engine for every workflow.

## Release/tag-based deployment flow

1. **Create a release reference:** A release can be initiated by pushing a tag matching `releases/X.Y.Z` or by publishing a non-prerelease GitHub Release with a `v` tag.
2. **Trigger the release entry workflow:** The matching tag or published release starts its corresponding workflow. The tag-push entry point filters to the `releases/*` pattern.
3. **Resolve release references:** The entry workflow identifies the current release tag and a previous reachable tag, then passes both references to the reusable delta deployment workflow.
4. **Calculate the release delta:** The reusable workflow compares the two references and creates the positive package and destructive manifests.
5. **Select the deployment environment:** The workflow targets the configured GitHub Environment, which supplies the environment-specific deployment context and any configured protection rules.
6. **Authenticate to Salesforce:** The reusable workflow establishes Salesforce CLI JWT authentication using the protected credentials for that environment.
7. **Deploy the release delta:** The workflow deploys the generated metadata delta using the configured Apex test level and handles destructive changes after the main deployment according to the release workflow.
8. **Publish a deployment summary:** The workflow writes a summary of the components included in the release and the deployment outcome.

### Release governance and hardening

The current release entry points have different tag-selection behavior. The tag-push flow filters to `releases/*`, while the release-based caller should apply the same release-only filtering. A reachable previous tag is not necessarily the last release successfully deployed to production.

For a stronger production release process, I would:

- Use one consistent release-only lookup strategy in both entry points.
- Resolve tags to immutable commit SHAs and verify that the previous baseline is an ancestor of the release commit.
- Record and verify the last successfully deployed production baseline.
- Fail closed when a valid previous baseline cannot be resolved rather than silently treating it as a first deployment.
- Use one canonical release trigger or a shared concurrency group so a tag push and GitHub Release cannot deploy the same release concurrently.
- Split release planning from deployment so reviewers can inspect the package and destructive manifests before approving production.
- Ensure deletion-only releases are processed even when the positive package manifest is empty.
- Require release commits to have passed the appropriate validation before production deployment. The current tag/release path should not be described as having the same PR validation-to-Quick-Deploy handoff unless that control is added and verified.

## Supporting scripts and workflow responsibilities

| File or workflow | Responsibility |
|---|---|
| `.github/workflows/template.yaml` | Shared PR validation and deployment workflow logic |
| Environment-specific PR workflow callers | Supply the target environment and invoke shared workflow logic |
| `.github/workflows/reusable-delta-deploy.yml` | Reusable release delta deployment logic |
| `.github/workflows/release-based-deploy.yaml` | Published GitHub Release entry point |
| `.github/workflows/tag-based-deploy.yaml` | Release tag entry point |
| `READ_PRBODY.py` | Extracts Apex test class selection from the PR body |
| `CODE_COVERAGE.py` | Evaluates deployment/test output and per-class coverage |
| `sf_delta.py` | Separate custom Git-diff-based Salesforce metadata delta utility |
| `scripts/shell/setup-scratch-org.sh` | Automates Salesforce scratch-org setup |

## Key points to remember in an interview

- There are two distinct deployment models: PR validation followed by Quick Deploy, and release/tag-driven delta deployment.
- Reusable workflows centralize common Salesforce setup and deployment logic; caller workflows provide the target environment context.
- PR validation uses a metadata delta, static analysis, Apex tests and a coverage evaluation before merge.
- The validation job ID is handed to the merge workflow through a PR comment because the validation and merge are separate workflow runs.
- Quick Deploy reuses successful validation results and does not run Apex tests again. An ineligible validation requires a normal deployment.
- The release workflow calculates a delta between release references and deploys it through the selected GitHub Environment.
- Destructive changes need their own manifests and correct sequencing. A deletion-only release is still a real deployment.
- Clearly distinguish implemented controls from hardening recommendations, especially concurrency, merge queues, branch protection, production validation and release-baseline verification.
