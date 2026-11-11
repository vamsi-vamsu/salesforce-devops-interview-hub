# Salesforce CPQ & Conga/Apttus

> Interview preparation notes. Answers reflect the experience and implementation details shared by Vamsi. Where a topic is outside direct project experience, the answer is framed as conceptual knowledge or an approach rather than claimed hands-on delivery.

## Q: What is Salesforce CPQ?

Salesforce CPQ helps sales teams configure products, calculate pricing and generate accurate quotes. It uses product configuration, pricing rules, discounts and quote-line calculations to guide users through the quoting process. From a DevOps perspective, CPQ changes need careful dependency management and functional regression testing.

## Q: Explain the basic Salesforce CPQ architecture.

At a high level, Salesforce CPQ connects products, product options, configuration rules, price rules, discounting and quote records. Users configure products and the Quote Line Editor calculates quote lines and pricing based on the configured rules. The resulting quote can then move through approval and document-generation processes.

## Q: What are Product Rules?

Product Rules control which products or options can be selected during configuration. They can validate a configuration, show or hide options, automatically select products or alert users to incompatible choices. They help enforce product compatibility and business rules.

## Q: What are Price Rules?

Price Rules automate pricing calculations by evaluating conditions and applying actions to quote or quote-line fields. They can calculate discounts, adjust prices or populate values based on product, customer or quote context. They need regression testing because a small rule change can affect many quote scenarios.

## Q: What are Configuration Rules?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What are Discount Schedules?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is a Quote Line?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is the Quote Line Editor?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How does Salesforce CPQ calculate pricing?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is the difference between Salesforce CPQ and Conga CPQ?

Salesforce CPQ is Salesforce's own configure-price-quote solution, while Conga CPQ is a separate commercial CPQ product that integrates with Salesforce. Both support product configuration and pricing, but their data models, rule engines, setup and deployment considerations differ. The implementation architecture determines the specific metadata and package dependencies.

## Q: What is Apttus, and how does it integrate with Salesforce?

Apttus, now part of Conga's product portfolio, provides configure-price-quote and contract lifecycle management capabilities that can run on Salesforce. It uses managed-package components and Salesforce records, with integrations to quoting, approvals and document or contract processes. Customizations must be tested against the installed package version.

## Q: How do you deploy CPQ metadata?

I identify the affected configuration, rules, fields, objects, Apex, Flows and package dependencies, then deploy them in the required order. I validate in a lower environment using representative quote scenarios, including product configuration, pricing, approvals and document generation. I also confirm managed-package versions and post-deployment settings.

## Q: What challenges arise when deploying CPQ customizations?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage CPQ managed packages during deployments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle CPQ dependencies and package upgrades?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you troubleshoot CPQ deployment issues?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage CPQ configuration changes across environments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

