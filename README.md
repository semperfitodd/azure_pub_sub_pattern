# Pub/Sub Azure Pattern

## Contents
1. [Pattern Function](#pattern-function)
2. [Pattern Resources](#pattern-resources)
3. [Pattern Requirements](#pattern-requirements)
4. [Pattern Structure](#pattern-structure)
5. [Service Usage Details](#service-usage-details)
6. [Workflow](#workflow)
   - [Data Ingestion](#data-ingestion)
   - [Data Queueing and Publishing](#data-queueing-and-publishing)
   - [Data Subscriptions](#data-subscriptions)
   - [Further Processing](#further-processing)
7. [Application Y](#application-y)
8. [Appendices](#appendices)
   - [Appendix A: Event Driven Architecture](#appendix-a-event-driven-architecture)
   - [Appendix B: Troubleshooting Guide](#appendix-b-troubleshooting-guide)
9. [Glossary](#glossary)

## Pattern Function
To process incoming event traffic. 
Route that traffic that event data based on defined logic workflows.
Make that event data available for other applications and services in a reliable, scalable fashion.

## Pattern Resources
- Azure Kubernetes Service
- Azure Application Gateway
- Azure Kubernetes Service
- Azure Logic Apps
- Azure Service Bus
- Azure Event Grid
- Azure Sql
- Azure Storage Account

## Pattern Requirements
- X number of applications either publishing data or subscribing to it.
- All actions performed by applications must be proven to have completed or retried X number of times before an alert notification is triggered.

## Pattern Structure
*Please refer to the diagram below for the structural overview.*

![Pattern Structure](link-to-diagram.png)

### Service Usage Details
This structure hosts applications on AKS (Azure Kubernetes Service). 
The publishing applications are protected and routed to by an Azure Application Gateway 
Azure Logic Apps are used to manage work by reviewing inputs to determine the proper workflow
Azure Service Bus will use Namespaces and Topics to organize data that can be retrieved from subscribers.
Azure Event grid will be used to determine what actions are taken when the workflow has an error
Azure Sql and Azure Storage Accounts will be used to store data.

## Workflow
### Data Ingestion
Clients will upload data via a frontend that will then pass the data to an Azure Application Gateway. The Application gateway will be configured to scan the data for relevant information to determine the appropriate backend path. The application gateway will then route the data to Application X. Application X is one of any applications hosted on AKS that are coded to process the received data. After the data is processed, the application will POST a request trigger with a schema-appropriate body to the Azure Logic App callable endpoint designated by the application code.
Azure Logic App will be configured with visually configurable workflows. The callable endpoint will route the event data to the workflow, where it will be searched for key:value pairs in its payload to determine the next step of the workflow. Assuming the workflow has two successful paths, 0 and 1, the Logic App will route successful messages to Azure Service Bus Namespace A, configured with Topics A0 and A1. Unsuccessful messages, such as those with missing fields, will be routed to the Azure Event Grid.

### Data Queueing and Publishing
Azure Service Bus will be configured with namespaces appropriate for business operations and topics appropriate for Publish distinct data. Assuming the successful paths from Application X to be 0 and 1, the data from path 0 will be sent by the Logic App to Topic A0 and data from path 1 will be sent by the Logic App to Topic A1.
Azure Service Bus will be configured FIFO(First In First Out) message queuing in Peek Lock mode. This requires the subscribing service to request the data, which will cause a Consumption Lock to be placed on the message, preventing other subscribers from retrieving the message. This Consumption Lock remains in place until the subscribing service releases it, or the TryTimeout is exceeded (default 60 secs).
A message that does not have the lock removed by the subscriber and exceeds the TryTimeout setting, will have its retry counter incremented by 1. If the retry counter exceeds the MaxRetries setting (default 10), the message will be routed to the DLQ (Dead Letter Queue), which is a built-in subqueue for all Azure Service Bus Queues and Topics with dead-lettering enabled. This pattern has dead-lettering enabled by default.

### Data Subscriptions
This pattern is defaulted with two subscribers, Application Y and Application Z. Each application will subscribe to their defined Azure Service Bus Topic and retrieve the published data. Upon completion of processing the data, the application will trigger the Service Bus Topic to remove the consumption lock.
In this pattern Application Y will use Topic A0 and Application Z will use Topic A1.

### Further Processing
#### Application Y
In this pattern, Application Y is designed to subscribe to Azure Service Bus Topic A0 and process the published data.
The data will be loaded into an Azure SQL database to be used by other applications and to provide data for reporting.

## Appendices
### Appendix A: Event Driven Architecture
This pattern is aligned with event-driven architecture principles to allow for scalability and the ability to handle unpredictable loads. This is done by decoupling the individual components and allowing them to communicate through events, which means that each part can be scaled independently without affecting the others.

### Appendix B: Troubleshooting Guide
1. **Problem A**: Solution for problem A.
2. **Problem B**: Solution for problem B.
... (more troubleshooting details)

### Appendix C: Security Measures

### Appendix D: Performance Considerations

### Appendix E: Cost Analysis

### Appendix F: Testing Strategies

## Glossary
- **AKS**: Azure Kubernetes Service
- **DLQ**: Dead Letter Queue
- **FIFO**: First In First Out
... (other terms)
