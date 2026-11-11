# Flow, Apex & Testing

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.
>
> Tags: **[Experience]** = work you have described: Copado deployments (flows deploying as active through the target org's automation setting, dependency order, the missing-field validation failure, the same-class Apex conflict), the GitLab 2GP pipeline (Apex tests in merge request pipelines, JUnit XML and testError.json, scheduled package-level coverage, package versions with and without coverage), sandbox seeding with Data Loader, scratch orgs from snapshots, and Apex to anonymize user records for GDPR. **[Approach]** = how I would do it. **[Conceptual]** = platform knowledge without documented hands-on delivery.
> You told the Copado panel you have not done much Salesforce development recently, so frame the Apex and Flow *authoring* answers as knowledge and approach, and lead with your release, deployment and testing-in-pipeline experience. Confirm limits and behavior against current Salesforce documentation before quoting.

## Q: What are Salesforce Flows?

**[Conceptual]** Flow is Salesforce's declarative ("clicks, not code") automation tool, built in Flow Builder. A flow can collect input from users, create, update or delete records, call Apex or other flows, send notifications and integrate with external systems. It is now Salesforce's standard automation tool: Workflow Rules and Process Builder reached end of support on December 31, 2025 (they still run but receive no fixes), and Salesforce provides a Migrate to Flow tool for moving them over.

From a release point of view, flows are metadata: they are retrieved, committed to Git, promoted and deployed like other components, but with extra considerations around versions and activation (see the deployment and activation questions).

## Q: What are the different types of Salesforce Flows?

**[Conceptual]**

- **Screen Flow:** guided user interface, launched by a user (button, action, Lightning page, Experience Cloud).
- **Record-Triggered Flow:** runs when a record is created, updated or deleted. It can run *before save* (fast, updates the triggering record without a DML statement) or *after save* (can create or update related records and call actions).
- **Scheduled-Triggered Flow:** runs on a schedule for a set of records.
- **Platform Event-Triggered Flow:** runs when a platform event message is published.
- **Autolaunched Flow (no trigger):** has no UI and is called by another flow, Apex, a REST call or an action.
- **Flow Orchestration:** coordinates multi-step, multi-user processes across flows.

Other options exist (for example, flows triggered from other actions), so confirm the current list in Salesforce documentation.

## Q: How do you deploy Flow changes?

**[Approach, anchored on Copado experience]** Treat a flow like any other source-controlled metadata:

1. Build and test the flow in a development org; make sure it works with realistic data and permissions.
2. Retrieve it into the User Story, review the metadata list and diff, and commit it with its dependencies (custom fields, objects, Apex classes it calls, permission sets, custom labels, email templates).
3. Promote through the pipeline, validating at each stage and confirming the dependency order.
4. In the destination, confirm how the flow arrives (active or inactive) and whether any manual or data steps are needed.
5. After deployment, smoke test the flow and check for errors, and record the result in the release evidence.

Practical points: a flow version number does not need to be deployed; what matters is which version is active in the source and how the destination is configured to receive it. Flow tests (where used) and Apex tests for any invocable Apex should run as part of validation.

**[Experience]** In Copado, I deployed flows with dependency order in mind, and the active flow from the lower org arrived as active when "deploy flows as active" was enabled in the target org's automation settings.

## Q: How do you resolve Flow conflicts?

**[Approach]** Flows are stored as XML and a small change in Flow Builder can reorder many lines, so textual merges are noisy and risky. I reduce conflicts first: one owner per flow where possible, coordination when several stories touch the same flow, and early back promotion so work starts from the current version. When a conflict does occur, I:

1. Identify both changes and the intended final behavior with the developers or business owner.
2. Pick a base version, then re-apply the other change in Flow Builder rather than hand-merging XML where possible (or merge the XML very carefully and open it in Flow Builder to confirm it loads).
3. Validate, test the flow's paths, and run any related Apex tests before promoting.
4. Check whether the pipeline's merge behavior supports semantic merge for flows; if not, treat flows as whole-file components that need manual review.

I avoid letting the last merged story overwrite the other, because that can silently remove logic.

**[Experience]** The same principle applied to the same-class Apex conflict: Copado flagged it, auto-merge is excluded for classes, and the developers and I resolved it together instead of accepting either side.

## Q: How do you handle Flow activation across environments?

**[Approach, anchored on Copado experience]**

- Decide per environment whether a deployed flow should arrive active or inactive, and configure it. In Salesforce this is controlled by the Process Automation Settings option to deploy processes and flows as active in the target org. **[Experience]** With "deploy flows as active" enabled in the target org's automation settings, the active flow from the lower org deployed as active.
- Be careful in lower environments and sandboxes: activating a flow that sends emails or calls integrations after a refresh can cause unwanted side effects, so check settings and masked data first.
- For production, activate through the release process (not manually) and include activation, and a rollback step (reactivate the previous version), in the runbook.
- Keep older versions only as long as needed, since a flow can have a limited number of versions (verify the current limit).
- After deployment, verify which version is active and run a smoke test.
- After a sandbox refresh, review scheduled and time-based flows, which often need reactivation.

## Q: What are Apex test classes?

**[Conceptual]** Apex test classes are classes annotated with `@isTest` that exercise Apex code and assert the expected behavior. Salesforce requires them for deployment to production, and good ones verify behavior, not just coverage. Key points:

- Test methods run in an isolated context and roll back their data, so they do not affect the org's records (by default they do not see existing org data, unless `SeeAllData=true`, which is discouraged).
- Create test data inside the test (a test data factory or `@testSetup`), cover positive, negative and bulk scenarios (200 records), and test as different users with `System.runAs` where permissions matter.
- Use `Test.startTest()` and `Test.stopTest()` to reset governor limits and force asynchronous code to finish, and mock callouts with `HttpCalloutMock`.
- Assert results with `Assert` methods rather than only executing code.

```apex
@isTest
private class AccountServiceTest {
    @testSetup static void setup() {
        insert new Account(Name = 'Test Account');
    }
    @isTest static void updatesAccountRating() {
        Account a = [SELECT Id FROM Account LIMIT 1];
        Test.startTest();
        AccountService.markHot(a.Id);
        Test.stopTest();
        Assert.areEqual('Hot', [SELECT Rating FROM Account WHERE Id = :a.Id].Rating);
    }
}
```

**[Experience]** In the GitLab 2GP pipeline, Apex tests ran in merge request pipelines, produced JUnit XML results and testError.json, and gated the merge.

## Q: What is Salesforce code coverage?

**[Conceptual]** Code coverage is the percentage of executable Apex lines that were executed by tests, calculated per class and trigger and as an overall figure for the org. It is a measure of which lines ran, not of whether the logic is correct, so it must be paired with meaningful assertions. You can view it in Setup (Apex Classes, Apex Test Execution), the Developer Console or through the API and CLI.

**[Experience]** In the GitLab project, scheduled pipelines collected package-level Apex coverage and published it to the GitLab Wiki. A real scenario: a nightly coverage pipeline created multiple package versions from the same commit, and policy-base 1.0.0.15 had coverage while 1.0.0.16 did not, so the latest version could not be used blindly for staging.

## Q: What is the minimum Apex code coverage required for production deployment?

**[Conceptual]** Salesforce requires at least 75% overall Apex code coverage to deploy to production, and every trigger must have some coverage. All tests run in the deployment must pass. Teams often set a higher internal threshold, so I separate Salesforce's minimum from the project's own quality gate and check the project's actual setting before quoting a number.

Related nuances: when a deployment runs only specified tests (`RunSpecifiedTests`), the 75% requirement applies to each class and trigger in the deployment, not just to the org overall; and for unlocked packages, a package version generally needs at least 75% coverage before it can be promoted for production installation. Confirm these against current Salesforce documentation.

**[Experience]** The package version coverage scenario above is why I verify coverage per version rather than assuming the newest version qualifies.

## Q: How do you run specific Apex tests during deployment?

**[Conceptual, with CLI examples]** Choose a test level on the deployment:

- `NoTestRun`: no tests (not allowed for production deployments that include Apex).
- `RunSpecifiedTests`: only the tests you name.
- `RunLocalTests`: all tests except those from installed managed packages (the default for production deployments containing Apex).
- `RunAllTestsInOrg`: every test in the org.

```bash
# Validate (check-only) with specified tests
sf project deploy validate --source-dir force-app --test-level RunSpecifiedTests --tests AccountServiceTest --tests OrderServiceTest

# Deploy with specified tests
sf project deploy start --source-dir force-app --test-level RunSpecifiedTests --tests AccountServiceTest

# Run tests directly and produce JUnit output with coverage
sf apex run test --class-names AccountServiceTest --code-coverage --result-format junit --output-dir tests --wait 20
```

Specifying tests speeds up validation, but with `RunSpecifiedTests` each deployed class and trigger needs 75% coverage from the tests you list, so choose them carefully. A quick-deploy of a validated package can avoid re-running tests at deployment time, subject to Salesforce's time window.

**[Experience]** In the GitLab pipeline, merge request pipelines ran Apex tests and wrote JUnit XML to `tests/*-junit.xml`, with testError.json retained for 15 days. In Copado, test execution and validation were part of the promotion process; confirm the exact test level your pipeline used.

## Q: How do you troubleshoot failing Apex tests?

**[Approach, anchored on pipeline experience]**

1. Read the first failure: class, method, error message and stack trace. Separate real failures from noise.
2. Classify the cause: a real defect; a test data problem (missing required field, validation rule, duplicate rule, record type or permission); an environment difference (missing metadata, different configuration, flow or trigger now firing); a governor limit; mixed DML; time or order dependence; hard-coded IDs; dependence on org data (`SeeAllData`); or an intermittent failure (row locks, asynchronous timing).
3. Reproduce it in the same environment, run that one class (`sf apex run test --class-names ...`) with logs enabled, and inspect the debug log.
4. Fix the root cause in source control (the code or the test), rerun the single class, then the wider set, and re-validate.
5. If it is flaky, make the test deterministic rather than rerunning until it passes.
6. Check whether the failure affects other stories in the release.

**[Experience]** In the GitLab project, failures were visible through JUnit results and testError.json, and Google Chat notifications flagged pipeline failures. A missing field in the destination org caused an Apex class to fail validation; I recommitted the field to the User Story and re-ran validation.

## Q: How do you manage test data?

**[Approach, anchored on experience]**

- **Unit tests:** create their own data (a test data factory or `@testSetup`), never rely on org data, and cover bulk volumes. This keeps tests isolated and repeatable.
- **Developer and QA environments:** repeatable seeding from versioned files or scripts (Data Loader or CLI), with parent-before-child order and External IDs for relationships.
- **Realistic data:** sandbox templates for Partial Copy, with masking of personal data (Salesforce Data Mask or Apex scripts) when production data is used.
- **Scratch orgs:** preloaded from a snapshot, plus a seed step.
- **Hygiene:** small purpose-specific datasets, cleanup scripts, and no sensitive data in repositories.

**[Experience]** I seeded sandboxes with Data Loader and sandbox templates after refreshes, used scratch orgs from a nightly snapshot in the GitLab project, and wrote Apex to anonymize user records for GDPR.

## Q: How do you handle dependencies between Apex classes and Flows?

**[Approach, anchored on experience]**

- **Identify them in development and review:** flows call invocable Apex, Apex can start flows, and both reference fields, objects, custom labels, permission sets and each other. Use the metadata list and Git diff to confirm what each story includes.
- **Sequence deployments** so prerequisites go first: custom fields and objects, then Apex classes, then the flows that call them (and the reverse for removing something).
- **Validate the combined set**, not each item alone, and run the Apex tests that cover the invocable methods the flow uses.
- **Do not promote a dependent story in isolation**, and document dependencies in the User Story or release notes.
- **Watch for behavior changes:** a change to an invocable method's signature can break an active flow at runtime, even if it deploys, so test the flow after deployment.

**[Experience]** In Copado, I checked dependency order for deployments that included flows, and a missing field referenced by an Apex class caused a validation failure that I fixed by recommitting the field to the story. In the GitLab project, the pipeline resolved package dependencies and installed packages in dependency order.

---

## Additional questions

## Q: What is the difference between before-save and after-save record-triggered flows?

**[Conceptual]** A before-save flow runs before the record is written, so it can set fields on the triggering record without an extra DML operation, which makes it fast and efficient. An after-save flow runs after the record is saved, so it has the record ID and can create or update related records, send emails and call actions. Use before-save for simple same-record field updates and after-save when you need related records or actions.

## Q: How do flows and Apex triggers interact, and what causes problems?

**[Conceptual]** Both can run on the same object in the order of execution, so multiple automations can fire on one save and sometimes cause recursion, unexpected updates or governor limit problems. Good practice is one trigger per object with handler logic, a clear decision on whether a given behavior lives in flow or Apex, and testing of the combined behavior. When both exist, Apex tests can fail after a flow is added, so run tests when flows change.

## Q: How do you test flows?

**[Approach]** Use Flow Builder's debug and the flow test feature (for record-triggered flows), test each decision path with positive, negative and bulk data, test with different users and permissions, and cover invocable Apex with Apex tests. Include flow paths in UAT scenarios and smoke tests after deployment. Check whether your organization enforces flow test coverage on deployment, since requirements can differ.

## Q: What is the difference between RunLocalTests and RunSpecifiedTests, and when do you use each?

**[Conceptual]** `RunLocalTests` runs all local tests (excluding managed package tests) and is the safest default for production releases; it takes longer. `RunSpecifiedTests` runs only the named tests, which speeds up validation for small, well-understood changes, but each deployed class and trigger must reach 75% coverage from those tests. I choose based on release risk and the project's policy.

## Q: How do you stop slow Apex tests from slowing releases?

**[Approach]** Measure which tests take longest, remove unnecessary data creation and `SeeAllData`, use `@testSetup` for shared data, avoid redundant callouts and DML, and make tests independent so they can run in parallel. In pipelines, run only relevant tests where policy allows, and use quick deploy after a validated package. **[Experience]** In the GitLab project, changed-package detection and parallel package builds and installs kept pipeline time down, including staging dropping from about 5 hours to about 2.

## Q: How would you migrate Workflow Rules and Process Builder to Flow?

**[Approach]** Inventory active workflow rules and processes by object, prioritize by risk and usage, and use the Migrate to Flow tool for simple cases. Rebuild complex ones manually, because order of execution differs (before-save flows run earlier than Process Builder did) and the tool may not handle every case, such as certain formulas or record type criteria. Test each migrated flow in a sandbox against the original behavior, deploy through the pipeline, and deactivate the old automation only after validation.

## Likely follow-up questions

## Q: Can you deploy a flow without its dependencies?

No. If the flow references a field, Apex class, label or other component that does not exist in the destination, validation fails. Include the dependencies in the same promotion or deploy them first. **[Experience]** The missing-field validation failure is a good example: I recommitted the field to the story and re-validated.

## Q: Why do tests pass in the sandbox but fail in production validation?

Usually environment differences: data, configuration, validation rules, flows or triggers that exist only in production, permissions, or test code that depends on org data. Compare the environments, fix the cause in source control, and re-validate. This is also why validation against production (check-only) is valuable.

## Q: How do you make sure a flow change does not break existing Apex?

Run the full relevant Apex test set after the flow change, because a new or changed record-triggered flow can alter data that existing tests assume. Fix the tests or the flow, and include the combined behavior in regression testing.

## Q: What would you do if coverage drops below the threshold at release time?

Identify which classes lost coverage and why (new untested code, removed tests, a failing test), add or fix tests, and revalidate. If it cannot be fixed in time, defer the offending story through the release governance process rather than lower the gate. **[Experience]** The coverage-by-package-version scenario in the GitLab project shows why coverage is checked per version before use.

## Q: Do you write Apex yourself?

Be straightforward. Your background includes development, but you told the panel you have not done much Salesforce development recently. Say what you have written, describe your approach to testing and review, and emphasize your strength in release engineering, pipelines, validation and test management.

## Be honest about limits

Lead with deployment, pipeline and test-management experience. Present Flow and Apex authoring details, and any numbers (coverage thresholds, version limits, test levels), as knowledge you verify against current Salesforce documentation and the project's own settings.