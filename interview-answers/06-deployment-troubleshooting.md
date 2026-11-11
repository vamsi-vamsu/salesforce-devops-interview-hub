# Deployment Troubleshooting & Scenarios

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: A deployment works in development but fails in QA. How would you troubleshoot it?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you troubleshoot Apex test failures during deployment?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What happens when a deployment contains missing dependencies?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you resolve metadata conflicts?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle conflicting changes to the same Flow?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you deploy Flow changes safely across environments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle a production hotfix?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you back-promote a production hotfix to lower environments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is your rollback strategy for Salesforce deployments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you recover from a partially failed deployment?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle destructive changes in production?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you troubleshoot deployment failures caused by managed packages?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle a component unexpectedly missing from a Copado promotion?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage dependencies when User Stories contain related metadata?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle a sandbox refresh when the Git branch is outdated?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you ensure only approved changes reach production?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you troubleshoot a JWT authentication failure?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage expired certificates?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle API version differences?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you prevent environment drift?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._


## Q: A production deployment succeeds, but business functionality is broken. What do you do?

I treat it as a production incident, not as a successful release simply because the deployment job completed. I identify the affected functionality, business impact and scope, then review deployment history and recent metadata changes with the relevant technical owners. I decide whether a targeted correction or rollback is safer, based on dependencies and impact. After recovery, I run regression and smoke tests, reconcile the fix in source control, and document the root cause and preventive actions.

## Q: How do you handle an emergency change made directly in production outside CI/CD?

I do not overwrite a working emergency fix without understanding its impact. I capture the exact production change, confirm the business outcome, and compare production with the source-controlled production baseline. I then create a reviewed corrective or hotfix change in Git, validate and test it, and reconcile it into the production, release and development branches as appropriate. This prevents a later deployment from removing the emergency fix and restores traceability.

## Q: How do you prevent an emergency production fix from being overwritten by the next release?

I make sure the fix is represented in source control and reconciled with the production baseline first. I then propagate it into the active release and development lines using the established merge or back-promotion process. Before the next deployment, I compare the target environment with the intended source state and confirm that the fix remains in scope. I also run the relevant regression tests.

## Q: What if business stakeholders request an immediate deployment while required checks are failing?

I explain the specific failure, business risk and fastest safe options. I first look for a way to isolate the problem, correct it, or reduce the deployment scope without compromising dependencies. If it qualifies as an emergency, I follow the formally approved exception process with documented risk acceptance and a recovery plan. I do not silently bypass controls, and I complete the missing validation and reconciliation afterward.

## Q: How do you decide between rolling back and deploying a forward fix?

I assess the scope of the defect, the affected metadata, dependencies, data impact and how quickly a safe correction can be delivered. A targeted rollback is appropriate when restoring the previous state is predictable and will not break dependent changes. A forward fix is safer when a rollback would remove valid functionality or cannot reverse data changes cleanly. Whichever approach is selected, I validate it, reconcile Git and verify the business behavior.

## Q: How do you explain a production failure to a nontechnical stakeholder?

I explain which business functionality is affected, the impact, the immediate recovery action and when the next update will be provided. I avoid unnecessary technical detail and distinguish confirmed facts from what is still being investigated. After recovery, I summarize the cause, prevention steps and any business action required.
