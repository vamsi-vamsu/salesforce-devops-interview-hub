# Salesforce Development & Integrations

This category covers Apex, triggers, Lightning Web Components, REST and SOAP integrations, asynchronous processing, data volume, security and production troubleshooting. Present answers as hands-on examples only when they match work actually performed. Otherwise, frame them as the approach you would take.

## Q: How do you explain your Salesforce development experience in a DevOps interview?

My Salesforce development background helps me understand the metadata and application behavior that move through a release pipeline. I can discuss Apex, triggers, Flows, Lightning components, integrations and the dependencies that affect deployments. In a DevOps role, I use that understanding to assess change impact, troubleshoot validation failures, plan deployment order and work with developers and QA to deliver changes safely.

## Q: Explain how you design a REST integration between Salesforce and an external system.

I start by defining the business event, data mapping, direction of data flow, volume, latency and error-handling requirements. On the Salesforce side, I configure required objects, fields, permissions and the integration user. I use secure authentication such as OAuth with Named Credentials, and implement the callout using Apex when that is the right fit. I validate HTTP responses, capture errors safely and design retry behavior for transient failures. I also consider idempotency, monitoring, test coverage and reconciliation.

## Q: What is the role of Named Credentials in Salesforce integrations?

Named Credentials provide a managed way to define the endpoint and authentication configuration for callouts. They help avoid hard-coding URLs, usernames, passwords or tokens in Apex. I configure access for the appropriate users or principals, use the supported authentication mechanism and keep permissions limited to what the integration requires.

## Q: What is the difference between REST and SOAP integrations?

REST commonly uses HTTP methods and lightweight JSON payloads, while SOAP uses a formal XML message structure and service contract such as WSDL. REST is often a natural fit for web APIs, while SOAP can be useful when an enterprise service exposes a strongly defined contract. I choose based on the external system's interface, security, payload, reliability and business requirements.

## Q: How do you handle API failures in Salesforce?

I validate the HTTP status code and response body, distinguish client errors from authentication, authorization, not-found and server errors, and log enough context to troubleshoot without exposing sensitive information. For transient failures, I use bounded retry logic with an appropriate delay or backoff. I also make the operation idempotent where possible so a retry does not create duplicate records. Failed transactions should be visible for monitoring and controlled reprocessing.

## Q: How do you process callouts asynchronously in Apex?

When the work should not run in the user's synchronous transaction, I can enqueue Queueable Apex that implements Database.AllowsCallouts. I pass the required record identifiers or payload context, perform the callout asynchronously and record the response or failure. I consider transaction boundaries, retry limits, monitoring and duplicate prevention. For large volumes, I evaluate Batch Apex or an external integration platform rather than creating an unbounded queue.

## Q: What is the difference between Queueable Apex and Batch Apex?

Queueable Apex is useful for asynchronous work represented as a job and may need job chaining or complex parameters. Batch Apex is designed for processing large record sets in manageable chunks through start, execute and finish. I choose based on volume, transaction limits, need for chunking, callout requirements and operational monitoring.

## Q: How would you process 50,000 or more Salesforce records?

I clarify whether the processing is an import, update, calculation or integration, and whether it must be synchronous. For large data volumes, I consider Batch Apex or Salesforce Bulk API, depending on whether the work is internal Apex processing or an external data load. I use selective queries, process records in chunks, handle partial failures and capture failed record identifiers. I also plan for retries, reconciliation, monitoring and realistic volume testing.

## Q: How do you bulkify an Apex trigger?

I assume the trigger can receive a collection of records. I collect relevant IDs in Sets, query related data once using bulk SOQL, and use Maps for efficient lookups. I perform DML on collections outside loops and avoid repeated queries or updates. I also check recursion and automation interactions, and test with batches of up to 200 records rather than only a single record.

## Q: How do you handle multiple triggers on the same object?

I prefer a single trigger per object with business logic delegated to handler classes or a trigger framework. I organize logic by trigger context and keep the implementation bulk-safe and testable. Salesforce does not guarantee execution order among multiple triggers on the same object, so consolidating the entry point helps make behavior more predictable. I also consider interactions with Flows and other automation.

## Q: What is the difference between @wire and imperative Apex in LWC?

The @wire service provisions data reactively and is useful when the component should respond to changes in reactive parameters. Imperative Apex gives the component explicit control over when the method is called, such as after a user clicks Search or submits a form. For wired Apex methods, the method must be cacheable. I choose based on whether the interaction should be reactive or explicitly initiated.

## Q: How do unrelated Lightning Web Components communicate?

I use Lightning Message Service when components do not have a direct parent-child relationship or are in different regions of a Lightning page. I define a message channel, publish a message from one component and subscribe from the other. I keep the message payload focused and handle subscription lifecycle appropriately.

## Q: How do you prevent duplicate records created by Salesforce users and integrations?

I define what makes a record unique, such as an external customer identifier or a business-approved combination of fields. I use Matching Rules and Duplicate Rules for supported user-entry scenarios. For integrations, I prefer an External ID field and upsert operations so the same external record can be matched reliably. I also consider concurrency, data normalization and duplicate handling in the integration logic.

## Q: An Account's Billing Address changes and related Contacts must be updated. How would you design it?

I detect whether the relevant Billing Address fields actually changed, collect the affected Account IDs and retrieve related Contacts in bulk. For a manageable number of records, I can update the Contacts in a bulk-safe transaction. If an Account can have a very large number of related Contacts, I would move the work to an asynchronous process such as Batch Apex. I would also consider whether the business rule should be implemented through Flow or Apex and test bulk updates.

## Q: An Order becomes Activated and its details must be sent to an ERP system. How would you implement it?

I detect the transition to Activated, not simply every update to an already activated Order. I capture the required Order details and enqueue asynchronous processing for the callout. The integration uses Named Credentials for authentication, serializes the request, validates the response and records the integration status. I design retries for transient failures and use an idempotency key or external reference to avoid sending duplicate transactions.

## Q: An external application sends 10,000 customer records every day. How would you design the integration?

I clarify the delivery window, data size, latency and error-recovery requirements. For a large-volume load, I consider Salesforce Bulk API 2.0 rather than making an individual synchronous REST request for each record. I use an External ID for upsert, validate incoming data, capture rejected records and provide a reconciliation report so source and Salesforce counts can be compared. I also plan for authentication, monitoring, retries and duplicate prevention.

## Q: A Batch Apex job fails after processing many records. How do you troubleshoot and recover?

I check the Apex Jobs page or AsyncApexJob details, identify the failed batch execution and review the exception and logs. I determine whether the cause is a governor limit, validation rule, record lock, data-quality issue or external dependency. I capture failed record IDs, correct the root cause and rerun only the records that need processing where the design permits. The batch should be restartable and should not unnecessarily repeat successful work.

## Q: A production transaction hits the Apex CPU time limit. How do you investigate?

I review debug logs and identify expensive logic, nested loops, repeated processing and recursive automation. I check how triggers, Flows and managed-package automation interact in the same transaction. I optimize data structures and queries, reduce unnecessary work and move suitable large-volume processing to asynchronous Apex. I then test with representative data and monitor the result.

## Q: How do you test Apex callouts?

I use Apex test classes with HttpCalloutMock to provide controlled responses. I cover successful responses, expected client and server errors, malformed payloads, authentication-related failures where applicable and retry behavior. I also test bulk behavior and verify that Salesforce records are updated correctly. The tests should not depend on a live external service.

## Q: What is the difference between Apex tests and browser end-to-end tests?

Apex tests validate server-side logic such as classes, triggers and business rules. Browser end-to-end tests validate user journeys through the Salesforce UI, such as navigating a record page, entering data and submitting a transaction. They complement each other: one checks application logic, while the other checks behavior from the user's perspective.

## Q: A trigger works for one record but fails when Data Loader inserts 200 records. What do you check?

I inspect debug logs and test results for SOQL, DML, CPU and recursion problems. I look for queries or DML inside loops, assumptions that Trigger.new contains only one record, and automation that executes repeatedly. I refactor using Sets, Maps and collection-based DML, then test with 200 records and realistic related data.

## Q: How do you prevent a retry from creating duplicate integration records?

I design the operation to be idempotent. The external system supplies a stable unique identifier, which I store in a Salesforce External ID field. I use upsert or another controlled matching strategy rather than blindly inserting on every retry. I also record request status and correlation identifiers so a repeated message can be recognized and safely handled.

## Q: What security checks do you consider when writing Apex?

I check object permissions and field-level security, and I consider record sharing behavior. I use sharing declarations deliberately and enforce CRUD/FLS where required by the execution context. I avoid exposing sensitive values in error messages or logs, validate incoming data and ensure integrations run with least-privilege access. I also review whether UI and API entry points enforce the same business rules.

## Q: A user can see records belonging to another department. How do you investigate?

I review the object's Organization-Wide Defaults, role hierarchy, sharing rules, teams, manual sharing and permission assignments. I inspect Apex classes and custom components for sharing behavior and verify object and field permissions. I reproduce the issue with representative users and correct the underlying access model, then retest both allowed and denied access.

## Q: After a deployment, thousands of records have incorrect field values. What do you do?

I assess the impact and identify affected records and fields. I review recent code, Flows, triggers, data loads and deployment tasks to find the cause. I check available field history, audit records and backups or exports to determine the correct prior values. I validate a recovery plan in a sandbox, restore affected data carefully and verify the result. I then add preventive checks and regression coverage.
