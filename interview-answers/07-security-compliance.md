# Security, Authentication & Compliance

> Interview preparation notes. Answers distinguish documented implementation experience from conceptual knowledge. Topics outside direct project experience are framed as an approach rather than claimed hands-on delivery.

## Q: How does JWT-based authentication work in Salesforce CI/CD?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you authenticate Salesforce CLI from a pipeline?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage connected apps?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you manage certificates and private keys?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: Where should Salesforce credentials be stored?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you prevent secrets from appearing in pipeline logs?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you implement security scanning in CI/CD?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you control access to production deployments?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you implement approval workflows?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you maintain deployment audit trails?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you support compliance and release governance?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._


## Q: How do you secure a CI/CD pipeline that deploys to production?

I use least-privilege access, protected branches, required reviews and successful CI checks. Deployment credentials are stored in an approved secret manager or protected pipeline variables, never in source code or logs. Production deployments are restricted to authorized users and require the configured approvals. I also review audit logs, credential rotation and access permissions periodically.

## Q: How do you handle authentication failures in a Salesforce CI/CD pipeline?

I check the Salesforce authentication configuration, connected app or OAuth setup, certificate validity, integration-user permissions and target org identity. I confirm that the pipeline retrieves the correct secret and that credentials or certificates have not expired or been rotated without updating the pipeline. I avoid printing credentials in logs. After correcting the cause, I rerun authentication and validate the target org before deployment.
