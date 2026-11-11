# GitLab CI/CD Project Implementation Flow

> Step-by-step reference for the Salesforce DX monorepo implementation. The repository uses two long-lived branches: `Develop` and `main`.

## Branching and developer workflow

1. Developers create a scratch org using `create_dev_scratchorg.sh`. The scratch org is based on the current `Develop` baseline.
2. Developers create feature branches from `Develop`.
3. They make and test their Salesforce changes in the scratch org, then commit the changes to the feature branch.
4. They open a merge request (MR) from the feature branch to `Develop`.
5. The MR pipeline validates the proposed changes before they are merged.

## Merge request validation pipeline

1. **Security:** Run the configured security checks, including the secret-detection checkpoint.
2. **Provision scratch org:** Use a nightly snapshot when one is available.
3. **Snapshot bypass:** If the MR title contains `skip-snapshot`, bypass the snapshot path and create a regular scratch org.
4. **Fallback:** If snapshot creation or use fails, fall back to regular scratch-org creation so validation can continue.
5. **Deploy:** Deploy the required metadata and packages in dependency order.
6. **Test:** Run the required Apex unit tests.
7. **Review:** Provide a Review job so QA can log in to the scratch org and review the change.
8. **Notify:** Send Google Chat notifications when the pipeline fails.
9. **Merge controls:** The MR can be merged only after the pipeline succeeds and the required approvals are present.

### Snapshot considerations

- A nightly pipeline creates scratch-org snapshots to speed up MR validation.
- Snapshot-backed scratch orgs use the prepared baseline.
- The `skip-snapshot` title condition deliberately selects the regular scratch-org path.
- Regular scratch-org creation is also the fallback when a snapshot is unavailable or cannot be used.
- The snapshot-based path has a limitation for deleting components. Use the regular path when destructive changes need to be processed.

## QA deployment after merge to Develop

1. Merging an approved MR into `Develop` triggers the QA deployment pipeline.
2. The pipeline runs its **Security → Build → Deploy** stages.
3. It detects which packages have changed.
4. It builds and deploys the relevant packages to QA in dependency order.
5. Pipeline failures are reported to the relevant Google Chat channels.
6. Code coverage is not part of this QA deployment pipeline. Coverage is tracked separately by the nightly coverage job.

## Staging release process

1. **Confirm release scope:** On the staging release day, remind developers and teams about the cutoff and join release calls to confirm the tickets included in the release.
2. **Remove work that is not ready:** If a ticket is not ready for QA, revert its commit from `Develop` before creating the release branch.
3. **Create release branch:** Create the release branch from `Develop`.
4. **Open release MR:** Raise a merge request from the release branch to `main`.
5. **Start staging:** Manually trigger the staging pipeline with `package_list=all`.
6. **Run security and package discovery:** The staging flow runs **Security → Get package list → Build → Deploy**.
7. **Check package coverage:** For each package, check whether the latest package version has code coverage enabled.
8. **Reuse or create package version:** Reuse the latest coverage-enabled package version when available. If it is not coverage-enabled, create a new package version and deploy that version.
9. **Optimize package execution:** Build and deploy independent packages in parallel where dependencies allow. Parent/child pipelines and conditional jobs coordinate the package work.
10. **Publish package details:** Each package deployment sends a Google Chat notification and updates the GitLab Wiki with package and version IDs.
11. **Prepare the next development cycle:** Bump the package versions in `sfdx-project.json` for the next release in `Develop`.
12. Developers can continue their work for the next release while the release candidate proceeds through the release process.

## Production release process

1. On production release day, resolve outstanding conflicts and merge the release MR.
2. The manual production pipeline becomes available after the release MR is merged.
3. Retrieve the approved package names and package IDs recorded in the GitLab Wiki.
4. Supply the package names and IDs to the production job and run the pipeline.
5. The pipeline promotes the selected package versions and deploys them to Production.
6. Complete post-deployment steps, validate the deployment and communicate the outcome to business owners.
7. Tag the release branch to identify the production release.
8. Delete the release branch after the release is complete.

## Hotfix process

1. Create a feature branch from the normal development line, implement the fix and validate it in QA.
2. Create a release branch from the tag corresponding to the previous production release.
3. Cherry-pick the approved fix commit onto that release branch.
4. Follow the normal staging and production release process.
5. Ensure the fix is also represented in the active development line so it is not lost in a future release.

## Scheduled and supporting workflows

- **Nightly snapshot pipeline:** Creates snapshots used by MR scratch-org provisioning.
- **Nightly code coverage pipeline:** Checks package coverage against Dev Hub and posts the results to Google Chat.
- **Review scratch-org lifecycle:** Supports QA review of the scratch org created for an MR.
- **Release notifications and tracking:** Google Chat communicates pipeline and package deployment status; GitLab Wiki records package/version IDs.
- **Release notes support:** A Google Gem is used as part of the release-notes workflow.

## Key points to remember in an interview

- Two long-lived branches: `Develop` and `main`.
- Feature branches are created from `Develop`; MRs target `Develop`.
- MR validation uses snapshot-backed scratch orgs where possible, with `skip-snapshot` and regular-org fallback behavior.
- After merge, the QA pipeline deploys changed packages in dependency order.
- Staging is manually triggered with `package_list=all`.
- Staging reuses a coverage-enabled package version or creates a new one, and uses dependency-aware parallelism.
- Package/version IDs are recorded in GitLab Wiki and communicated through Google Chat.
- Production deployment is manually triggered with the approved package names and IDs.
- Hotfixes branch from the previous production release tag and cherry-pick the approved fix.
