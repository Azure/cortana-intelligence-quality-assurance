# Cortana Intelligence Suite Solution Template
# Predictive Analytics for Quality Assurance Process in Manufacturing

George Iordanescu, Ph.D.<br/>
*Data Scientist at Microsoft*


## Executive Summary


The purpose of the project is a manufacturing solution template for predicting failures in manufacturing pipelines (assembly lines). The idea is to leverage test systems already in place and failures data, specifically looking at returns and functional failures at the end of assembly line. By combining these with domain knowledge and root cause analysis within a modular design that encapsulates main processing steps, we provide a generic advanced analytics solution that uses machine learning to predict failures before they happen. Early prediction of future failures allows for less expensive repairs or even discarding, which are usually more cost efficient than going through recall and warranty cost.

![Assembly Line](https://cloud.githubusercontent.com/assets/16708375/19886616/f5a621d0-a01b-11e6-84c1-39d10d5c8941.png)

#### TAGS -  Azure Machine Learning, Azure Stream Analytics, PowerBI, Azure Event Hub, Azure WebJob, IoT, Predictive Maintenance, Predictive Analytics, QA

## Business Audiences

In this repository you will find a folder labeled [*Solution Overview for Business Audience*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Solution%20Overview%20for%20Business%20Audience) which contains:
- Infographic: covers the benefits of using advanced analytics for Quality Assurance Process in Manufacturing
- SolutionDescription: a detailed desxription of the [Cortana Intelligence Suite](https://www.microsoft.com/en-us/server-cloud/cortana-intelligence-suite/Overview.aspx) solution for Quality Assurance Process in Manufacturing
- Walking Deck: a  presentation covering this solution and benefits of using the Cortana Intelligence Suite

For more information on how to tailor Cortana Intelligence to your needs [connect with one of our partners](http://aka.ms/CISFindPartner).

## Technical Audiences

### Automated Deployment

We provide two deployment guidelines:

-  [step by step] (https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/StandAlone/README.md) deployment
-  one-button [click](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/CIQSDeployment/caqs-patterns/manufacturing/ManufacturingTemplateInstructions.md) experience. 

For technical problems or questions about deploying this solution, please post in the issues tab of the repository.


### Solution Architecture
![Solution Diagram](https://cloud.githubusercontent.com/assets/16708375/19811942/4375cbfa-9d3c-11e6-99b8-d953124d9361.png)

 <sub><sup>
 Solution design for Predictive Analytics for Quality Assurance Process in Manufacturing
</sup></sub>


 - Predictions are made for a individual products as they travel down a production line passing through 5 different waypoints. 
 - Event ingestion is performed using an [Azure Event Hub](https://azure.microsoft.com/en-us/documentation/articles/event-hubs-overview/) which receives simulated records  of ALS test measurements sent using an [Azure WebJob](https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/).
 - Event processing is performed using an [Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) job which passes events to the appropriate [Azure Machine Learning](https://azure.microsoft.com/en-us/services/machine-learning/) endpoint and 
 pushes the results to [PowerBI](https://powerbi.microsoft.com/) datasets.  
 - (optional, data processsing "cold path") Azure [Blob Storage](https://azure.microsoft.com/en-gb/documentation/articles/storage-blob-storage-tiers/#quick-start) sinks are used for storage and for off-line post-processing.
 
 



