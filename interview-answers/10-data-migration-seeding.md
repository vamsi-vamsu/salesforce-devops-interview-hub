# Data Migration & Data Seeding

> Interview preparation notes. Answers reflect the experience and implementation details shared by Vamsi. Where a topic is outside direct project experience, the answer is framed as conceptual knowledge or an approach rather than claimed hands-on delivery.

## Q: What is the difference between data migration and data seeding?

Data migration moves business data from one system or environment to another, often as part of an implementation or transition. Data seeding loads controlled sample or reference data into an environment so developers and testers can work with realistic scenarios. Migration emphasizes completeness and reconciliation; seeding emphasizes repeatability and fit for testing.

## Q: How do you migrate data between Salesforce environments?

I begin with source-to-target mapping, field validation and a migration sequence based on object relationships. I load parent records first, use External IDs to maintain relationships, and then load dependent records. After loading, I reconcile record counts, validate key fields and relationships, review rejected records and obtain business sign-off.

## Q: Which tools have you used for Salesforce data migration?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is Salesforce Data Loader?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is the Bulk API?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What is the difference between Bulk API 1.0 and Bulk API 2.0?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you migrate parent-child records?

I load parent records first and capture or map their Salesforce IDs using stable External IDs. Then I load child records and reference the parent through the External ID relationship mapping. I validate relationship counts and sample records after the load to ensure the hierarchy is preserved.

## Q: How do you maintain relationships between records during migration?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: What are External IDs, and why are they important?

An External ID is a field used to identify a Salesforce record using a value from an external system. It supports upsert operations and helps preserve relationships during migration without relying on Salesforce-generated record IDs. It is especially useful for repeatable loads and cross-system reconciliation.

## Q: How do you handle data mapping?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle duplicate records during migration?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you validate data after migration?

I compare source and target record counts, validate required fields and key business values, and check parent-child relationships. I also review rejected records and transformation exceptions, run business reports or queries, and ask data owners to validate representative scenarios. Validation should be documented before the migration is signed off.

## Q: How do you handle failed records?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you migrate large volumes of Salesforce data?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle data transformation?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you seed test data into scratch orgs?

I use repeatable data templates or scripts to create the minimum representative records needed for tests. The process respects object dependencies, uses stable reference values and avoids production-sensitive data. For automated tests, I prefer test factories or generated data where possible so tests remain isolated and repeatable.

## Q: How do you create reusable test data templates?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you maintain referential integrity?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you handle sensitive data and data masking?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

## Q: How do you reconcile source and target record counts?

_Sample answer to be completed. Prepare this using the project context, technical decisions, troubleshooting steps and outcomes relevant to the question. Do not claim experience that you have not personally had._

