# Git & Branching Strategies

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: Explain your branching strategy.

**Basis: Implemented (GitLab 2GP project)**

"We used two long-lived branches, `develop` and `main`, in an SFDX monorepo with 9 unlocked 2GP packages. Developers pulled `develop`, created a scratch org with our `create_dev_scratchorg.sh` script, cut a feature branch from `develop`, and opened an MR back to `develop`.

On release day, everything merged into `develop` ships. We cut a release branch from `develop`, MR it to `main`, trigger the staging pipeline manually with `DEPLOYMENT_TYPE=staging`, then promote to staging and production. Anything not ready is reverted from `develop` with an MR revert before the release branch is cut.

The trade-off is that `develop` is effectively the release candidate, so unready work has to be reverted rather than held back by a flag or a separate branch. That keeps the model simple, but it needs discipline about what merges and when. The MR pipeline compensates: every change is validated in a scratch org with Apex tests before it can merge."

**Likely follow-ups:** Why revert instead of cherry-pick into a release branch? How do you avoid reverting often? How do you track which MRs are release-ready?

## Q: What is GitFlow?

**Basis: Conceptual, with a comparison to your model**

"GitFlow is a branching model with long-lived `main` and `develop` branches plus short-lived feature, release, and hotfix branches. Features branch from `develop`. A release branch is cut from `develop` for stabilization and merged to `main` and back to `develop`. Hotfixes branch from `main` and merge to both.

Our model was a simplified GitFlow: long-lived `develop` and `main`, feature branches off `develop`, and a release branch cut from `develop` and merged to `main`. I'd describe it as GitFlow-influenced rather than strict GitFlow."

**Caveat:** Don't claim you ran formal hotfix or support branches unless you did. Those aren't in your documented process.

## Q: What is trunk-based development?

**Basis: Conceptual**

"Trunk-based development means everyone integrates into one main branch in very small, frequent merges. Branches are short-lived, typically a day or two. Unfinished work is hidden behind feature flags, and the trunk is always releasable, which depends heavily on strong automated testing.

Compared to our model: we kept feature branches short-lived and validated each MR automatically, which is trunk-like. But we had a separate `develop`/`main` split and a release-day cut, so we weren't pure trunk-based. In Salesforce, trunk-based is harder because metadata changes can't always be hidden behind flags the way application code can."

**Caveat:** Say "I understand the model" rather than "I ran it."

## Q: What is the difference between merge and rebase?

**Basis: Conceptual**

"Merge combines two branches and preserves history exactly as it happened, with a merge commit joining them. Rebase replays your commits on top of another branch, producing a linear history but rewriting the commit SHAs.

My rule of thumb is to rebase local, unshared feature work to stay current and keep history clean, and never rebase shared or already-pushed branches others depend on. Merge is the safer default for shared branches. In an MR-driven flow, the MR itself is the integration point, so the main concern is keeping the feature branch current with `develop` before the pipeline validates it."

```bash
# Illustrative
git fetch origin
git rebase origin/develop      # local feature branch only
git merge origin/develop       # safer if the branch is shared
```

## Q: How do you resolve merge conflicts?

**Basis: Approach**

"First I identify the cause: is it two people editing the same lines, or a stale branch? Then:
1. Update my branch with the target (`develop`) and see exactly which files conflict.
2. Resolve each file with understanding of both intents, talking to the other author if it's not obvious.
3. For Salesforce metadata (profiles, permission sets, flows, layouts), I check carefully because XML conflicts can silently drop or duplicate entries.
4. Re-run validation (deploy plus Apex tests in a scratch org) after resolving, since a textually clean merge can still break behavior.

Prevention matters too: small MRs, frequent syncs with `develop`, and clear package ownership reduce conflicts."

```bash
# Illustrative
git fetch origin && git merge origin/develop
git status                    # list conflicted files
# edit, then:
git add <file> && git commit
```

## Q: How do you manage long-lived branches?

**Basis: Implemented for `develop`/`main`; Approach for others**

"In our project the long-lived branches were `develop` and `main`. We kept them healthy by requiring MR pipeline success and approvals to merge, so neither could take unvalidated changes. `main` represented what was cut for release.

For any other long-lived branch, my approach is to avoid them where possible, because divergence is what creates painful merges. If one is unavoidable, I sync it from the integration branch frequently, keep it protected, and time-box it."

## Q: How do you handle hotfix branches?

**Basis: Approach (not documented as part of your process)**

"I'd handle a hotfix as an expedited version of the normal path rather than bypassing it. The sequence would be:
1. Branch from `main` so it reflects exactly what's in production.
2. Make the minimal fix and validate it in a scratch org with Apex tests.
3. Get the fix through approvals and the same deployment controls, including CAB or emergency change process in a regulated environment.
4. Merge to `main`, deploy, then merge or cherry-pick back to `develop` so the fix isn't lost in the next release.

The key risk is forgetting the back-merge, which causes the bug to reappear."

**Caveat:** If asked "did you do hotfixes?", answer only from what you actually did on the project.

## Q: How do you promote changes from development to production?

**Basis: Implemented (GitLab 2GP project)**

"Promotion is package-based and pipeline-driven:
1. **MR to `develop`:** changed-package detection, scratch org from the nightly `oneibm` snapshot, deploy, Apex tests, and QA review. Approvals plus a passing pipeline are required to merge.
2. **QA pipeline after merge:** `get_package_list()` uses `git diff` over `packages/` and `legacy/schema/` to find affected packages. We resolve dependencies, build package versions, and install in QA in dependency order via downstream pipelines.
3. **Release branch from `develop`, MR to `main`.**
4. **Staging pipeline (manual, `DEPLOYMENT_TYPE=staging`):** security stage, package list as the `STG_PACKAGE_LIST` artifact, then a package version coverage check. We reuse an eligible version or recreate one, save the version ID artifact, and install in staging.
5. **Production:** after staging validation, release approvals and CAB, deploy the approved changes, then post-deployment validation. Rollback planning is part of the process.

The key idea is that we promote versioned, tested package artifacts rather than re-deploying source at each stage."

**Real scenario to know cold:** nightly coverage created multiple versions from the same commit. policy-base 1.0.0.15 had coverage and 1.0.0.16 didn't, so the latest version couldn't be used blindly. That's why the staging pipeline checks coverage before reusing a version.

## Q: How do you prevent developers from merging unvalidated code?

**Basis: Implemented**

"Merging to `develop` required the MR pipeline to pass plus approvals. The pipeline detected changed packages, deployed to a scratch org, and ran Apex tests, publishing JUnit XML (`tests/*-junit.xml`) and `testError.json` as artifacts with 15-day retention. Failures triggered Google Chat notifications. The scratch org stayed available for QA review before approval, and was cleaned up afterward.

Beyond the pipeline, I'd add branch protection so no one can push directly to `develop` or `main`."

```yaml
# Illustrative: restrict job to MR pipelines
mr-validate:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

**Caveat:** Direct-push protection and the exact branch settings aren't documented in your notes. Confirm before stating them as fact.

## Q: How do you identify changed files between branches?

**Basis: Implemented (concept); snippet is Illustrative**

"We used `git diff` in a Bash function, `get_package_list()`, scoped to `packages/` and `legacy/schema/`. It determined which packages were affected, then we resolved dependencies and passed the list to downstream pipelines. This is what lets us build and deploy only what changed rather than all 9 packages every time."

```bash
# Illustrative only. Replace with your real get_package_list() logic.
git fetch origin develop
git diff --name-only origin/develop...HEAD -- packages/ legacy/schema/
```

`A...B` (three dots) diffs from the merge base, so you see only what your branch changed. A two-dot diff against a moved target can include unrelated changes.

**Caveat:** I don't have your original function. Paste it if you want me to document it exactly, rather than approximating it.

## Q: How do you manage multiple release streams?

**Basis: Approach (not documented as part of your process)**

"Our project had a single release stream: `develop` to `main` on release day. For multiple concurrent streams, my approach would be:
- Keep a clear mapping of branch to environment and release.
- Use release branches per stream, and decide up front which stream receives which changes.
- Use package versioning to make each stream's contents traceable, which fits 2GP well.
- Be disciplined about forward-merging fixes so streams don't drift.

The cost is higher merge overhead and a greater chance of divergence, so I'd only add streams when the business genuinely needs them."

## Q: How do you keep feature branches synchronized with the development branch?

**Basis: Implemented workflow; Approach for the mechanics**

"Developers always started by pulling `develop` and cutting the feature branch from it. For longer-running work I'd regularly bring `develop` into the feature branch, by merge or rebase if the branch is unshared, so conflicts surface early and in small pieces. Before the MR is approved, the pipeline validates the result in a scratch org, so integration problems show up there rather than after merge."

```bash
# Illustrative
git checkout feature/my-change
git fetch origin
git merge origin/develop   # or: git rebase origin/develop (unshared branches only)
```