# Copado Project Implementation Flow

> Step-by-step reference for explaining the configured Copado release process. This page captures the specific environment topology and workflow described for this implementation.

## Environment architecture

**Multiple Developer Orgs → QA → UAT → Production**

- Multiple developer orgs feed into one shared QA environment.
- A separate Hotfix org is connected to Production and is used for urgent production fixes.
- Rollback, org sync and back-promotion capabilities are enabled for the environments.
- The shared QA environment is the first common integration point for changes coming from the developer orgs.

## Standard development-to-production release

1. **Create the User Story:** Each developer creates a Copado User Story for the assigned work and makes the change in their development org.
2. **Commit the changes:** The developer commits the required metadata to the Copado User Story.
3. **Run developer checks:** Execute the required test coverage and static code analysis checks before requesting promotion.
4. **Create or trigger the pull request:** A pull request is required before deployment. If the developer has not created one, the configured Copado quality gate raises or triggers a pull request between the promotion branch and the destination branch before the deployment can proceed.
5. **Complete peer review:** The pull request requires two approvals. After the approvals are received, it is merged and the changes are deployed to QA.
6. **Complete QA testing:** The QA team tests the changes in the shared QA environment. Successful User Stories are marked as ready for UAT.
7. **Prepare the UAT promotion:** The release owner selects the User Stories marked ready for UAT and creates a promotion containing the approved changes.
8. **Run release quality gates:** Execute Apex test coverage, static code analysis and deployment validation for the promotion.
9. **Promote to UAT:** Confirm tester availability, promote the changes to UAT, complete any required manual steps and communicate the deployment to the team.
10. **Complete UAT:** Business or designated UAT testers validate the changes and approve the stories for production.
11. **Confirm production scope:** On the planned production release day, collect the User Stories approved in UAT and confirm the final release scope.
12. **Complete CAB approval:** Review the release in the CAB meeting and obtain the required change approval.
13. **Submit the production promotion:** Create the production promotion and submit it to the Release Manager for review and approval.
14. **Validate and deploy:** After the Release Manager approves, run the production validation and deploy the approved changes to Production.
15. **Perform post-deployment checks:** Monitor the deployment, complete post-deployment activities, run smoke tests, verify functionality and communicate the successful deployment to the business.

## Back-promotion and environment synchronization

1. After User Stories reach Production, run the appropriate back-promotion or org-sync promotion to keep the lower environments aligned.
2. Preserve work-in-progress items when synchronizing. Do not overwrite active development changes that have not yet been released.
3. Back-promote production changes through the relevant lower environments so the next development or release cycle contains the production version.
4. Because multiple developer orgs feed the same QA org, use the pull-request quality gate and QA integration point to identify and resolve conflicts before changes progress further.
5. Confirm the target org and branch state before synchronization. The back-promotion destination depends on where development is active and which environments have already received the change.

## Hotfix process

1. Create the urgent fix in the dedicated Hotfix org connected to Production.
2. Follow the expedited hotfix process while retaining the required review and approvals.
3. Validate the fix and deploy it to Production through the approved hotfix path.
4. After deployment, back-promote the hotfix changes to the relevant lower environments and development lines.
5. Verify that the fix is present in the active development work so a future release does not overwrite the production correction.

## Key points to remember in an interview

- The standard path is **multiple Developer Orgs → shared QA → UAT → Production**.
- A separate Hotfix org is connected to Production.
- The pull request is a deployment prerequisite and requires **two approvals**.
- The configured quality gate can raise or trigger the pull request when one has not already been created.
- QA-approved stories are selected by the release owner for a UAT promotion.
- Production requires UAT approval, CAB approval and Release Manager approval.
- Back-promotion and org synchronization keep lower environments aligned while preserving work in progress.
- Rollback, org sync and back-promotion are enabled for the environments.
