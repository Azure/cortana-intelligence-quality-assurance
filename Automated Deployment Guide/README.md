# [Quality Assurance for Manufacturing Solution](https://go.microsoft.com/fwlink/?linkid=831187)

This folder contains the post-deployment instructions for the deployable Quality Assurance for Manufacturing solution in the Cortana Intelligence Gallery. 

To start a new solution deployment, visit the gallery page [here](https://go.microsoft.com/fwlink/?linkid=831187).

<Guide type="PostDeploymentGuidance" url="https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Technical%20Deployment%20Guide/SolutionDescription.md"/>

## <a name="Summary"></a>Summary
<Guide type="Summary">
In digital manufacturing, quality assurance across the assembly line is imperative. Identifying errors, slowdowns and potential failures before they occur rather than after they are detected can help companies be more proactive and improve productivity. The goal of the solution guide presented here is to predict failures in manufacturing pipelines (assembly lines). This is done by leveraging test systems already in place and failure data, specifically looking at returns and functional failures at the end of assembly line. By combining these with domain knowledge and root cause analysis within a modular design that encapsulates main processing steps, we provide a generic advanced analytics solution that uses machine learning to predict failures before they happen. Early prediction of future failures allows for less expensive repairs or even discarding, which are usually more cost efficient than going through recall and warranty cost.

The solution pattern uses several Azure services:
* Azure Web jobs
* Event Hub
* Stream Analytics
* Azure Storage
* Azure ML
* Power BI

</Guide>

## <a name="Description"></a>Description

#### Estimated Provisioning Time: <Guide type="EstimatedTime">30 Minutes</Guide>
<Guide type="Description">
The Cortana Intelligence Suite provides advanced analytics tools through Microsoft Azure — data ingestion, data storage, data processing and advanced analytics components — all of the essential elements for building a Quality Assurance for Manufacturing solution.

The solution is implemented in the cloud, using the flexible on-line Microsoft Azure platform that decouples infrastructure components (data ingestion, storage, data movement, visualization) from analytics engine that supports modern DS languages like R and Python. The solution modeling component can thus be retrained as needed and be implemented using high performance Azure Machine Learning algorithms, or open source (R/Python) libraries, or from a third-party solution vendor.

The 'Deploy' button will launch a workflow that will deploy an instance of the solution within a Resource Group in the Azure subscription you specify. The solution includes multiple Azure services (described below) along with a web job that simulates data so that immediately after deployment you have a working end-to-end solution.

For post deployment instructions, please see the [instructions here](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md).

## Solution Diagram
![Solution Diagram](https://cloud.githubusercontent.com/assets/16708375/20932195/acb87330-bbcb-11e6-8a89-27d8b6e17bdf.png)

## Technical details and workflow
1.	The Manufacturing Assembly Line simulation data is streamed by the newly deployed **Azure Web Jobs**. 

2.	This synthetic data feeds into the **Azure Event Hubs** as data points/events, that will be consumed in the rest of the solution flow and stored in **Azure SQL Data Warehouse**.

3.	There are 2 **Azure Stream Analytics** jobs used in this pattern to provide near real-time analytics on the input stream from the **Azure Event Hub**. Both jobs filter through the input data and pass the data points along to a **Azure Machine Learning** endpoint sending the results to a **Power BI Dashboard**.

4.	Finally, **Power BI** is used for results visualization.
</Guide>

#### Disclaimer

©2017 Microsoft Corporation. All rights reserved.  This information is provided "as-is" and may change without notice. Microsoft makes no warranties, express or implied, with respect to the information provided here.  Third party data was used to generate the solution.  You are responsible for respecting the rights of others, including procuring and complying with relevant licenses in order to create similar datasets.
