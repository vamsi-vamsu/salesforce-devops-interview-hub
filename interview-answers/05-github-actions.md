# Salesforce DX, Metadata & Packaging

> Interview preparation notes. Answers reflect the experience and implementation details shared by Vamsi. Where a topic is outside direct project experience, the answer is framed as conceptual knowledge or an approach rather than claimed hands-on delivery.

## Q: What is Salesforce DX?

Salesforce DX is Salesforce's modern development model and toolset for source-driven development. It supports project-based metadata, Salesforce CLI, scratch orgs, source tracking and package-based development. It allows teams to manage Salesforce metadata in Git and automate validation and deployment through CI/CD.

## Q: What is the difference between source format and metadata format?

Salesforce DX source format is optimized for development and source control, with decomposed and project-organized metadata. Metadata API format is the deployment-oriented structure used by Metadata API tools and package manifests. Salesforce CLI can convert or deploy between supported formats as part of the workflow.

## Q: What is the purpose of sfdx-project.json?

sfdx-project.json is the central Salesforce DX project configuration file. It defines package directories, namespace information, package dependencies and other project settings. In a multi-package repository, it helps Salesforce CLI understand the package structure and relationships.

## Q: What is project-scratch-def.json?

project-scratch-def.json defines the characteristics of a scratch org, including edition, features and settings. The Dev Hub uses this definition when creating the org. Teams can maintain different definitions for different development or validation needs.

## Q: What are scratch orgs?

Scratch orgs are temporary, source-driven Salesforce environments created from a Dev Hub. They provide isolated environments for development, automated testing and validation. They can be created consistently through Salesforce CLI and removed when no longer needed.

## Q: What are unlocked packages?

Unlocked packages are a Salesforce packaging model used to modularize custom metadata and distribute it across orgs. They support versioning and dependency management and are useful for teams adopting package-based development. They are different from managed packages, which are generally used for distributing protected commercial functionality.

## Q: What are second-generation packages (2GP)?

Second-generation packaging is Salesforce's modern package development model, supporting source-driven package creation and versioning. It includes managed and unlocked package types. In an unlocked-package model, teams can modularize an application, define dependencies and promote tested package versions through environments.

## Q: How do you create and install a 2GP package?

The team defines the package directory and package configuration in sfdx-project.json, then creates the package through Salesforce CLI using the Dev Hub. A package version is created and promoted when it has passed the required validation. The version is then installed into the target org in dependency order and validated with tests.

## Q: How do you manage package dependencies?

We maintain dependencies and ancestor tracking across the nine unlocked packages. The pipeline uses that dependency information and an explicit sequence to avoid building or installing a dependent package before its prerequisite. In the QA deployment flow, changed packages are processed sequentially. During staging, independent package builds and deployments can run in parallel, but packages with dependencies follow the required order. The resulting package IDs are recorded in GitLab Wiki, and sfdx-project.json is updated as part of the release workflow. This makes package promotion repeatable and traceable.

## Q: How do you deploy multiple packages in a monorepo?

I keep each package in its own directory and define relationships in the project configuration. The pipeline detects changed package directories, identifies dependent packages and builds or deploys them in the correct order. This allows teams to share one repository while keeping package ownership and release scope clear.

## Q: How do you handle package versioning?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you determine deployment order?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you deploy destructive changes?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle managed package upgrades?

I review the vendor release notes and compatibility requirements, then test the upgrade in a representative lower environment. I validate dependent customizations, Apex behavior, Flows and integrations before scheduling the production upgrade. The deployment plan includes a backup or recovery approach and post-upgrade smoke tests.

## Q: How do you troubleshoot missing metadata dependencies?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

