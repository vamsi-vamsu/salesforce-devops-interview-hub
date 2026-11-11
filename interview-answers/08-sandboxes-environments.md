# Sandboxes & Environments

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: What are the different types of Salesforce sandboxes?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is the difference between Developer, Developer Pro, Partial Copy, and Full Copy sandboxes?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What happens during a sandbox refresh?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage Git branches after a sandbox refresh?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you maintain consistency between Git and Salesforce environments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage sandbox-specific configurations?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle data templates during sandbox refreshes?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage parallel development environments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you coordinate releases across multiple environments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you prevent environment drift?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._



## Q: What are the standard Salesforce sandbox storage capacities?

Salesforce's published standard limits are 200 MB data and 200 MB file storage for a Developer sandbox, and 1 GB data and 1 GB file storage for a Developer Pro sandbox. A Partial Copy sandbox has 5 GB data storage and file storage matching Production. A Full sandbox has the same data and file storage allocation as Production. Developer and Developer Pro data storage can be upgraded to 400 MB and 2 GB respectively where an upgrade license is available. Check the current Salesforce Help page and the org's actual license entitlements before quoting these values. <https://help.salesforce.com/s/articleView?id=sf.data_sandbox_environments.htm&language=en_US&type=5>

## Q: What is your strategy for refreshing sandboxes?

I refresh based on purpose and release cadence rather than refreshing every sandbox at the same time. I identify the source org, owners, active development, open User Stories, test data requirements and integrations first. Before refresh, developers commit or preserve work in progress and the release team records the branch and org state. After refresh, I reauthorize integrations and credentials, apply sandbox-specific configuration, load required test data, verify users and permissions, and reconcile the org with Git before development resumes. I avoid refreshing an active development org without coordinating with its users.

## Q: How do you handle storage issues in a Salesforce sandbox?

I check Setup > Storage Usage to identify whether data or file storage is the constraint and which objects or files consume the most space. I remove obsolete test records and files using an approved cleanup process, archive or reduce test data where appropriate, and review logs or attachments that are accumulating. For ongoing needs, I consider a Developer Pro or an available storage upgrade, or use a Partial Copy or Full sandbox when the testing purpose requires more data. I preserve required test scenarios and relationships instead of deleting records indiscriminately.

## Q: What should you check before refreshing a development sandbox?

I check for uncommitted metadata, open User Stories, local developer work, test data, integration endpoints, connected app credentials, scheduled jobs and sandbox-specific settings. I notify the developers and agree a cutoff. After refresh, I reconfigure endpoints and credentials, run any post-copy scripts, restore required test data and verify Git alignment. A sandbox refresh should not become an unplanned way to discard changes that were never committed.

## Q: What is the difference between sandbox refresh interval and sandbox storage capacity?

The refresh interval is the minimum time between refreshes for a sandbox type, while storage capacity defines how much data and file content the sandbox can hold. Standard Salesforce intervals are one day for Developer and Developer Pro, five days for Partial Copy and 29 days for Full. These are platform limits and can be subject to product changes, so I confirm current entitlements in Salesforce Setup and Help. <https://help.salesforce.com/s/articleView?id=sf.data_sandbox_environments.htm&language=en_US&type=5>

## Q: How do you manage sandbox storage when several teams need realistic test data?

I define a data strategy by environment purpose. Developer sandboxes generally use small, targeted datasets, while integration and QA environments use controlled datasets that cover cross-system and regression scenarios. Partial Copy uses a sandbox template to select relevant sample data, and Full Copy is reserved for cases that require a production-scale copy. I use repeatable data seeding and cleanup, avoid copying unnecessary records, and apply masking when sensitive data is involved.
