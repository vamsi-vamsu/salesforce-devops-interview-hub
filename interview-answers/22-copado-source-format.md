# Copado Source Format Pipelines

This category focuses on the Source Format Pipeline behavior covered in the supplied technical reference. Product behavior and available settings can vary by Copado package version and pipeline configuration. Confirm version-specific settings before relying on them.

## Q: What is a Copado Source Format Pipeline?

A Source Format Pipeline uses Salesforce DX source format and Git branches to manage metadata changes as they move through Salesforce environments. Environments represent pipeline stages, while User Stories capture units of work and their committed metadata. Promotions merge selected story branches and deploy the resulting changes to the next environment. The exact pipeline behavior depends on its configuration and installed Copado package version.

## Q: How are environments and credentials related in a Source Format Pipeline?

An environment represents a Salesforce org at a particular pipeline stage, such as development, integration, testing, staging, hotfix or production. Creating an environment also creates a related credential, which must be authenticated before the environment can be used. The credential's access is based on the Salesforce user used for authentication, so that user should have only the permissions required for the pipeline activities.

## Q: Which Apex test levels can be configured for an environment?

The reference lists No Test Run, Run Specified Tests, Run Local Tests and Run All Tests in Org. The environment can also provide defaults such as minimum Apex test coverage and the test level used for deployments. I choose the test level according to the environment's purpose and release policy, and verify that production requirements are met.

## Q: At which points can a Git conflict occur in Copado?

The reference identifies three main points. A commit conflict can occur when a feature branch is merged into its source-org branch. A promotion conflict can occur when User Story feature branches are merged into the promotion branch. A deployment-time conflict can occur when the promotion branch is merged with the destination-org branch. Identifying the point matters because the available resolution options differ.

## Q: What is the difference between commit, promotion and deployment conflicts?

A commit conflict occurs while committing a User Story into its source branch. A promotion conflict occurs while combining multiple User Story branches into a promotion branch. A deployment conflict occurs when the promotion branch is merged with the destination environment branch. Promotion conflicts can be resolved using the configured automatic, manual or smart resolution approach. A new deployment-time conflict cannot be resolved inline in the same way; the promotion needs to be re-merged or recreated against the updated destination state.

## Q: How does Copado auto-resolve a conflict?

Auto-resolve is a strategy for handling supported conflicts without requiring a person to edit every file. For a conflicting file that is part of the incoming User Story commit, the incoming feature branch can win; for a file not included in that story, the promotion or destination branch can win. This is a source-level strategy, so it is important to know whether a metadata type is merged semantically or replaced as a whole file.

## Q: What is semantic merge in Copado?

Semantic merge understands selected nested Salesforce metadata structures rather than treating the entire XML file as one text block. For supported types such as profiles, permission sets, labels, sharing rules, workflows and translations, the selected nested component from the User Story can be applied while unrelated nodes from the target branch are preserved. This reduces the risk of replacing unrelated configuration in a large metadata file.

## Q: Which metadata types can be risky with whole-file source override?

The reference highlights monolithic or whole-file types such as Apex classes, triggers, layouts, FlexiPages and Lightning components. If two User Stories modify the same file, a source override can replace the entire destination file and silently remove another developer's changes. For programmatic metadata such as Apex, Visualforce, LWC and Aura, the reference recommends excluding files from automatic resolution so a developer can review the conflict.

## Q: What is the purpose of the .copado_exclude_autoresolve file?

It identifies repository files or patterns that should not be automatically resolved. The file is placed at the repository root and supports .gitignore-style patterns. When a matching file conflicts, the promotion stops for manual review instead of applying the configured automatic resolution. The reference recommends keeping this file consistent across environment branches, because a branch without it may resolve those files automatically.

## Q: Should declarative metadata such as Profiles, Layouts and Flows be added to .copado_exclude_autoresolve?

Not by default. The supplied reference advises against manual merging of complex declarative metadata such as profiles, layouts, flows and permission sets because their XML structures can be difficult to merge safely. Copado's semantic merge is intended to preserve unrelated nested elements for supported metadata types. The exclusion file is more appropriate for programmatic files where a whole-file override could discard another developer's code.

## Q: How do you manually resolve a conflict in a Source Format Pipeline?

I first identify the conflicting operation and inspect the affected file and User Story scope. When the conflict is configured for manual resolution, I open Review Conflicts, compare the versions, and resolve the intended code or metadata changes. I remove every conflict marker, save the file and check its syntax. For LWC or other code, I use the available two-panel or three-panel view as appropriate. I then continue the promotion and rerun validation or tests to confirm the result.

## Q: What is Smart Conflict Resolution?

Smart Conflict Resolution stores a fingerprint of a manually resolved conflict and can reapply the same resolution when that fingerprint occurs again. The supplied reference says it is enabled by default. It can reduce repeated manual work for recurring conflict patterns. I would leave it enabled unless there is a specific reason to disable it, and still review the outcome where the change has functional risk.

## Q: What is gitresolution.json?

gitresolution.json records conflicts that Copado auto-resolved and the strategy used. It is useful when investigating why a particular version of a file won during a promotion. I review it alongside the affected metadata and the merge history rather than assuming a successful merge means that the intended code was preserved.

## Q: What is Overlap Awareness?

Overlap Awareness helps identify components committed in more than one User Story. The affected metadata can be flagged as Potential Conflict in the User Story Metadata related list. This gives developers and release managers an opportunity to review overlapping work before promotion. It does not prevent two developers from editing the same component, so ownership coordination, early pull requests and branch synchronization are still important.

## Q: How do you prevent conflicts when multiple developers modify the same Apex class?

I identify shared components early through overlap checks and pull-request review. I coordinate ownership with the developers, keep commits small and back-promote stable changes so feature branches do not drift. If two stories must change the same class, I make sure the intended combined version is present in both branches and resolve any conflict deliberately. The repository exclusion rules should also prevent risky whole-file code overrides from happening silently.

## Q: Why does merge order matter in a Copado promotion?

The promotion branch is created or reused from the destination branch, and Copado merges the selected User Story feature branches into it. The supplied reference says forward promotions default to ascending User Story Reference order. If a conflict is resolved by source override, the last merged story can determine the final file content. I review Potential Conflict items and set the User Story merge order deliberately when overlapping changes exist.

## Q: Can the User Story merge order be changed?

Yes. The reference describes an Order User Stories By setting at the pipeline level, with an option to override the order on an individual promotion before merging. This setting applies to forward promotions. Back promotions preserve the order in which the stories were originally promoted. I use a deliberate order when conflicts or dependencies make the sequence important.

## Q: What is Environment Sync?

Environment Sync moves a continuous sequence of User Stories from a higher environment to a lower environment while preserving work in progress by default. It retrieves the affected components from the selected source point, combines those changes with destination work in progress using the configured conflict strategy, deploys the result and updates the destination branch after a successful deployment. It is useful for synchronizing approved changes across lower environments.

## Q: How do you enable and run Environment Sync?

The supplied reference says Environment Sync is enabled at the pipeline level in Pipeline Builder and is available for Source Format Pipelines when the required package versions are installed. In Pipeline Manager, I select the higher source environment, the destination environment and a point in promotion history. I review the continuous range of stories and the Included Metadata view, then run Sync Environments & Deploy. I review the deployment result and confirm that the destination branch is updated.

## Q: Can you skip a User Story in an Environment Sync range?

No. The reference describes Environment Sync selection as a continuous range of User Stories. A story must exist in the source and every intermediate environment. If a story is missing, later stories cannot simply be selected by skipping over it. I investigate the missing story and its promotion history before deciding how to synchronize the environments.

## Q: Does Environment Sync preserve work in progress?

Yes, work in progress is preserved by default. The reference also describes an Ignore Work in Progress option for environments where the goal is to overwrite sandbox work with approved changes. I would use that option only when the environment's purpose and team coordination make it appropriate, because overwriting uncommitted work can cause developers to lose changes.

## Q: What is the difference between Environment Sync and classic back promotion?

Environment Sync is designed for synchronizing a continuous range of stories and preserving or intentionally ignoring destination work in progress. Classic back promotion is useful when the team needs to select individual eligible stories or use the established promotion workflow. The supplied reference notes that the same stories synchronized to a destination through Environment Sync cannot then be back-promoted again to that same destination through manual or automated back promotion.

## Q: What is a classic back promotion?

A back promotion moves User Stories that have already reached a higher environment back to one or more lower environments so their branches remain aligned. Eligible stories generally must have reached a higher environment, must not have originated from the destination and must not be excluded from the pipeline. Back-promoted stories follow their original promotion order. Before running one, I confirm that developers have committed or safely preserved their work in progress.

## Q: What is a mass back promotion?

A mass back promotion allows multiple eligible stories or sources to be synchronized through a bulk operation. The reference describes bulk and multiple-source mass back promotions. I review the eligible story set, destination environments and promotion order before executing it. I also confirm that the operation will not overwrite uncommitted sandbox work.

## Q: Why should production changes be back-promoted carefully?

The Git branch is the source of truth for the release process. If a production change is not synchronized to lower development environments, a later commit or promotion can reintroduce an older version and overwrite the production fix. I synchronize approved production changes down through the pipeline and verify that lower branches contain the intended state before new development commits begin.

## Q: What causes a Missing Commit error during Environment Sync?

The reference says this can occur when the source branch does not contain a required first or last commit for the selected range. Possible causes include a recreated branch or a story that was excluded before reaching production. I inspect branch history and the affected User Stories first, then follow the documented remediation for the specific branch state rather than applying a blind workaround.

## Q: What is the difference between Merge Changes, Deploy Changes, Merge & Deploy Changes and Validate Changes?

Merge Changes creates the promotion branch and merges the selected User Story feature branches. Deploy Changes deploys the changes already included in the promotion branch. Merge & Deploy Changes performs both operations. Validate Changes runs a validation deployment to check whether the changes can be deployed without completing the actual deployment. I choose the action based on whether the promotion branch is already prepared and whether the team needs a validation-only run.

## Q: What does Create Job Execution Only do?

It allows a user to create a draft job execution so the job steps and quality gates can be reviewed or edited before the execution is started. The supplied reference says the draft execution must then be started from its Job Execution record. This is useful when the release manager needs to inspect or adjust the execution plan before running it.

## Q: Why might a deployment step not run during validation?

The reference notes that validation can skip User Story deployment steps unless Run on Validation Deployments is selected. Whether a step supports validation also depends on the extension and its configuration. I check the job step settings and extension support before assuming that a successful validation exercised every deployment action.

## Q: What is Global Find & Replace in Source Format Pipelines?

Global Find & Replace uses YAML rules to replace or remove matching content in files during commits or deployments. Rules can be configured at pipeline or environment level, with pipeline-level YAML taking precedence when both exist. I use explicit file paths or patterns to limit the scope and avoid unintended changes to unrelated metadata.

## Q: What should you know about regular expressions in Global Find & Replace?

The supplied reference says regex expressions in replace_values are evaluated against the entire file rather than line by line. As a result, line anchors such as ^ and $ may not behave as expected. I prefer carefully scoped, non-anchored patterns or word boundaries and test the rule against representative metadata before applying it broadly.

## Q: How do you troubleshoot a Global Find & Replace rule that did not run?

I inspect the deployment Result logs. A successful rule reports replacement counts. A No yaml file found message indicates that the configured file was not found, while YAML parse errors indicate invalid configuration. I verify the YAML structure, file path or pattern, rule scope and whether the rule is configured at the pipeline or environment level.

## Q: How can a Git merge succeed but Salesforce deployment fail because of duplicate XML elements?

Git performs text-based merging and does not understand Salesforce XML semantics. A merge can therefore produce duplicate XML elements in a metadata file such as a layout, even though Git reports no conflict. I inspect the deployment parser error and the resulting XML, then use the duplicate-handling capability described in the Source Format reference and validate the corrected metadata.

## Q: How are data templates deployed with User Stories?

Data templates can be included as User Story deployment steps and configured to run before or after metadata deployment. The execution order matters when records depend on metadata or metadata depends on records. I review the template configuration and job-step sequence, then inspect the promotion result, data deployment summary, execution details, logs and record-level CSV results.

## Q: What does the newer Data Commit experience provide?

The supplied reference describes a preview of up to 1,000 records, source and destination value comparison, search and filtering, record selection, and the ability to commit a full template. Data and metadata can be committed in the same User Story, but dependency analysis does not apply to data. A data commit creates a snapshot, so later source changes do not modify the already committed dataset.

## Q: What should you consider when multiple data commits contain the same records?

The deployment-step order determines the final values when multiple data sets contain the same records with different values. The reference says the latest committed data set wins conflicts by default. It also lists a maximum payload size of 5 MB. I review record overlap and execution order before promoting data, and avoid assuming metadata dependency analysis will resolve data conflicts.

## Q: What are Source Format Environment Variables?

Environment Variables, or EnVars, provide environment-specific values that can be applied to metadata files during commit and deployment. They can be scoped to specific files, folders or filename and folder patterns using .gitignore-style syntax. I keep the scope narrow and verify which files receive the substitutions so that environment-specific values are not applied unintentionally.

## Q: Do EnVars apply to every type of deployment step?

No. The supplied reference says EnVars apply during commit to committed metadata files and during deployment. Apex script and custom-setting steps only receive EnVars without a scope statement. I review the step type and scope configuration before relying on a variable being available to a particular action.

## Q: What is a Clean Pull Request in Copado?

Clean Pull Request creates a temporary isolation branch to present a more focused view of the changes introduced by a User Story. Copado creates the isolation branch from the destination branch, squashes commits from the feature branch, cherry-picks the resulting commit into the isolation branch and opens the pull request against the destination branch. The isolation branch is for review; actual promotion and deployment continue to use the original feature branch.

## Q: Does a Clean Pull Request show the exact final state that will be deployed?

No. The supplied reference warns that neither Clean PR nor a standard PR guarantees a complete preview of the final post-merge destination state. A Clean PR can show unrelated destination metadata as removed because of how the isolation branch resolves conflicts. Those apparent removals are not necessarily deletions in the actual deployment. I use the PR for review, but still inspect the promotion and deployment results.

## Q: Should conflicts be resolved directly in GitHub or another Git provider?

For a Copado-managed promotion, I follow the Copado promotion and conflict-resolution process rather than merging or resolving conflicts directly in the Git provider. The reference warns that direct merges, conflict resolution or commits in the Git provider can break Copado's promotion process. I resolve the issue through the supported Copado workflow and verify the resulting branch state.

## Q: How do Copado quality gates work in Source Format Pipelines?

Quality gates run configured automated tests at defined points, such as after commit or before deployment. They can be scoped to a pipeline, environment or stage and use a configured test tool and extension. A rule can include run options, target environments or stages and optional filters based on metadata type, extension, directory or name. I confirm the gate's scope and execution point before relying on it as a release control.

## Q: Can quality gates block a promotion?

Yes. An After Commit gate can block promotion when tests fail or remain in progress. The appropriate response is to investigate the failure, correct the issue, recommit if required and rerun the tests until the required status is passing. The reference notes that quality-gate execution differs for validation promotions, validation promotion-deployments and back promotions, so I verify which operation actually runs the configured gate.

## Q: Should IgnoreTestStatusForPromotion be used to bypass failed quality gates?

The reference lists IgnoreTestStatusForPromotion=true as a setting that can allow promotion when tests are failed or still in progress, but warns that this is not recommended because defective code could be promoted. I would not use it as a routine workaround. Any exception should require explicit approval, documented risk acceptance and an agreed follow-up validation plan.

## Q: What is Reuse Promotion Branch on Conflict?

It is an option described for newer Source Format Pipeline versions that can allow a large promotion to resume from the User Story where the conflict occurred instead of restarting the entire promotion. Changes to the destination branch, selected stories, promotion metadata, merge order or promotion branch can require a restart. I verify that the installed Copado version supports the setting before depending on it.

## Q: What is the correct way to troubleshoot a parser error after a promotion?

I inspect the first meaningful deployment error and identify the affected metadata file. I search the source for unresolved conflict markers such as <<<<<<<, ======= and >>>>>>>, then check whether the file was modified during conflict resolution. I validate the file syntax, confirm the promotion branch reflects the latest destination state, and re-merge or recreate the promotion if the destination changed after the promotion branch was created. I rerun validation before continuing.

## Q: How do you handle a rename/rename conflict or a false-positive conflict?

I confirm the actual Git and metadata state instead of assuming every conflict notification represents the same issue. The reference notes that rename/rename conflicts may not appear in the online conflict panel and can require a different remediation path. Large XML files can also produce false-positive conflicts. I inspect branch history, file paths and component content, then choose a controlled resolution and validate the resulting metadata.

## Q: What should a release manager review before a large Source Format promotion?

I review the User Story scope, dependencies, Potential Conflict items, PR approvals, quality-gate status, merge order, deployment steps, tests and destination environment queue. I confirm that the promotion branch is based on the expected destination state and that the release runbook and owners are ready. After deployment, I review the results and preserve the relevant audit evidence.
