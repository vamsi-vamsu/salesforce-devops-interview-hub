# Certinia PSA

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: What is Certinia PSA?

Certinia PSA is a professional services automation solution built on Salesforce. It helps services organizations manage projects, resources, time and expenses, project financials and delivery operations. It provides a connected view of services work on the Salesforce platform.

## Q: What business problems does Certinia PSA solve?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How does Certinia PSA work within Salesforce?

Certinia PSA uses Salesforce as its platform and adds packaged functionality and data models for professional services operations. Users can manage projects, resource assignments, timecards, expenses and project financial information within Salesforce. It can connect operational services data with other business processes through Salesforce integrations.

## Q: What are the key functionalities of Certinia PSA?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is resource management in PSA?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is project management in PSA?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How does time and expense management work?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How does Certinia PSA integrate with Salesforce?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How are Certinia managed packages deployed?

The managed package itself is installed or upgraded using the vendor-supported package process, rather than deployed as ordinary custom metadata. Custom fields, Apex, Flows, permission sets and other extensions built around the package can be managed through the organization's release process. I would verify the supported upgrade path and dependencies with Certinia documentation before each upgrade.

## Q: How do you handle managed package dependencies?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you deploy customizations built on top of Certinia?

I identify which components are custom extensions and which are owned by the managed package. I deploy the custom metadata and automation through the normal CI/CD process, ensuring the required Certinia package version is already installed. Then I validate key PSA workflows such as project setup, resource assignment, time entry and approvals.

## Q: What challenges can arise during Certinia upgrades?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you validate Certinia functionality after deployment?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How would you manage Certinia PSA changes through CI/CD?

I would treat the Certinia package version as a controlled platform dependency and manage our custom extensions in source control. The pipeline would validate custom metadata and automation against an environment with the correct package version, run relevant tests and deploy approved changes in dependency order. Package upgrades would have a separate compatibility and regression-testing plan.

