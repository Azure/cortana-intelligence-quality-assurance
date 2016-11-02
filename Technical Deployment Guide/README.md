# Cortana Intelligence Suite Predictive Manufacturing Solution  

1. Introduction
===============

This tutorial will guide you through the process creating, from the ground up, a predictive manufacturing solution. Other related resources provided here:
 -  an extensive [solution description](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Technical%20Deployment%20Guide/SolutionDescription.md).
 -  [data flow](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Technical%20Deployment%20Guide/DataFlowReport.md) report demonstrating how post deployment monitoring can be used to visualize the timing and work-load of each key components of the solution.

2. Solution architecture description:
=====================================
 
![Solution Diagram Picture](https://cloud.githubusercontent.com/assets/16708375/19811942/4375cbfa-9d3c-11e6-99b8-d953124d9361.png)

 <sub>
 Solution design for Predictive Analytics for Quality Assurance Process in Manufacturing
</sup>


 - Predictions are made for a individual products as they travel down a production line passing through 5 different waypoints. 
 - Event ingestion is performed using an [Azure Event Hub](https://azure.microsoft.com/en-us/documentation/articles/event-hubs-overview/) which receives simulated records  of ALS test measurements sent using an [Azure WebJob](https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/).
 - Event processing is performed using an [Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) job which passes events to the appropriate [Azure Machine Learning](https://azure.microsoft.com/en-us/services/machine-learning/) endpoint and 
 pushes the results to [PowerBI](https://powerbi.microsoft.com/) datasets.  
 - (optional, data processsing "cold path") Azure [Blob Storage](https://azure.microsoft.com/en-gb/documentation/articles/storage-blob-storage-tiers/#quick-start) sinks are used for storage and for off-line post-processing. 
 

3. Deployment Steps:
====================

### Prerequisites

This tutorial will require:

 - An Azure subscription, which will be used to deoploy the IoT suite project 
   (a [one-month free
   trial](https://azure.microsoft.com/en-us/pricing/free-trial/) is
   available for new users)
 - An [Azure Machine Learning Studio](https://studio.azureml.net/) account.
 - A Windows Desktop or as Windows based [Azure Virtual Machine](https://azure.microsoft.com/en-us/services/virtual-machines/) to run a data generation tool.    

# Manual Steps
This section will walk you through the steps to manually create the manufacturing solution in your Azure subscription.

## Create an Azure Resource Group for the solution
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - Click "Resource groups" button on upper left, and then click __+Add__. 
 - Enter in a name for the resource group and choose your subscription.
 - For *Resource Group Location* you should choose one of the following as they are the locations that support machine learning workspaces:
    - South Central US
    - West Europe
    - Southeast Asia  

## Create an Azure Event Hub
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - In the left hand menu select *Resource groups*
 - Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group "Overview" blade.
 - At the top of the Resource Group blade click __+Add__.
 - In the *Search Everything* search box enter **Event Hubs**
 - Choose ***Event Hubs*** from the results, then click *Create*
 - For the name, use ***manufactureehns*** and ensure the subscription, resource group, and location match the selections made previously, then click *Create*.
 - Return back to *Resource groups* and choose the resource group for this solution.
 - Click on ***manufactureehns***, then on the subsequent blade, choose "Overview", and click __+Event Hub__
 - Enter ***manufactureeh*** as the name, move partition count to 4 and click *Create*

## Create Machine Learning Workspace
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - In the left hand menu select *Resource groups*
 - Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 - At the top of the "Overview" blade in Resource Group blade click __+Add__.
 - In the *Search Everything* search box enter **Machine Learning Workspace**
 - Choose ***Machine Learning Workspace*** from the results, then click *Create*
 - Enter ***manufacturemlws*** as the name
 - Ensure the subscription, resource group and location match previous selections.
 - Leave both storage account and web service plan as *Create new* and provide names acceptable to the user interface.
 - Click *Web service plan pricing tier* and choose ***S1 Standard***, and click "Select" button.
 - Click *Create* and return to the *Resource groups* blade. 

## Copy Machine Learning Experiments From Gallery
 - For each of the following experiments:
    - [Waypoint 0 Experiment](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-0-Predictive-Exp-1)
    - [Waypoint 1 Experiment](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-1-Predictive-Exp-1)
    - [Waypoint 2 Experiment](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-2-Predictive-Exp-1)
    - [Waypoint 3 Experiment](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-3-Predictive-Exp-1)
    - [Waypoint 4 Experiment](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-4-Predictive-Exp-1) 
 - Once loaded, click the ***Open in Studio*** button
    - When neccesary, log in to your Machine Learning Studio account
 - Select the region that matches previous selections and choose the worksapce ***manufacturemlws***, then click *Ok*
 - At the bottom of the page click ***Run***
 - Wnen completed, click ***Deploy Web Service*** using the selection ***Deploy Web Service [Classic]***
 
## Create Azure Stream Analytics (ASA) Job
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - In the left hand menu select *Resource groups*
 - Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 - At the top of the Resource Group blade click __+Add__.
 - In the *Search Everything* search box enter ***Stream Analytics job***
 - Choose ***Stream Analytics job*** from the results then click *Create*
 - Enter ***manufactureasa*** as the name and choose the subscritpion, resource group and location using previous choices.
 - Click *Create* and return to the *Resource groups* blade. 
 - Now log into the classic [Azure Managment Portal](https://manage.windowsazure.com)
 - Click ***STREAM ANALYTICS*** from the menu on the left.
 - Select the ***manufactureasa*** resource to open the Stream Analytics job to modify the job settings.
 - Click *INPUTS*  
    - At the bottom of the *INPUTS* page click ***+ADD INPUT***
    - Wizard Page 1 -> Data Stream, click next arrow.
    - Wizard Page 2 -> Event Hub, click next arrow.
    - Wizard Page 3 ->
        - INPUT ALIAS : InputHub
        - SUBSCRIPTION: Use event hub from current subscription
        - CHOOSE A NAMESPACE: ***manufactureehns*** (or whatever you have chosen for the namespace previously)
        - CHOOSE AN EVENTHUB: ***manufactureeh*** (or whatever you have chosen for the event hub previously)
        - Click the next arrow 
    - Wizard Page 4 -> EVENT SERIALIZATION FORMAT remains JSON, click the check box button to complete.
 - Click *FUNCTIONS (preview)*
        - Click ***ADD FUNCTION***
    - For each of the machine learning experiments we created
        - Select the ML workspace created earlier (manufacturemlws or whatever you have chosen for the ***Machine Learning Workspace*** previously).
        - Choose an ml endpont (for example Prediction Model Mk IIII (w.p. 0) [Predictive Exp.])
        - Enter the name waypointN where N matches the number from the ml endpoint, for example (w.p. 0), N = 0
        - Click the check button to complete. 
 - Click *OUTPUTS*
    - For each of the ml models, we will create a PowerBI dataset output. In the following steps, complete one for each value of [N] where [N]= 0->4 
        - At the bottom of the *OUTPUTS* page click ***+ADD OUTPUT***
        - Wizard Page 1 -> Power BI, click the next arrow.
        - Wizard Page 2 -> Click **Authorize Now** to access your Power BI data.
        - Wizard Page 3 ->
            - OUTPUT ALIAS : w[N]pbi
            - DATASET NAME : waypoint[N]
            - TABLE NAME : waypoint[N]
            - WORKSPACE : My Workspace
            - Click the check button to complete
- Click *QUERY*
    - Download the file StreamAnalyticsJob.txt from this repository. Copy and paste the content into the query window. 
    - Click *SAVE*
- When all inputs, functions, outputs and the query have been entered, click *START* at the bottom of the page.   

## Download and configure the data generator
 - Download the file ***ManufacturingGenerator.zip*** from this repository.
 - Unzip this file to the local disk drive of a Windows Machine.  
 - Open the file **ManfuacturingGenerator.exe.config** and modify the following AppSettings
    - EventHubName : ***manufactureeh*** (or whatever you have chosen for the event hub previously).
    - EventHubConnectionString : Find this value with these steps
        - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
        - In the left hand menu select *Resource groups*
        - Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
        - Select the Event Hubs (service bus namespace) created for this project (***manufactureehns*** or whatever you have chosen for the **Event Hubs** previously).
        - From the menu on the namespace blade select *Shared access policies*
        - Select *RootManageSharedAccessKey* 
        - Copy the content of the **CONNECTION STRING - PRIMARY KEY** (Warning: the string, *not* the plain *PRIMARY KEY*)
 - Double click the file **ManfuacturingGenerator.exe** to start data generation. This will open a console and show messages as data are streamed from the local computer into the event hub **manufactureeh**.
    - ***NOTE*** The following PowerBI Dashboards will only be dynamically updated when this generator is running. 


## Configure a Power BI dashboard
 You can go to [Power BI Dashboard](https://powerbi.microsoft.com/) and use a Real-time dataset to build reports and dashboards using your data!
  - On the [Power BI Dashboard](https://powerbi.microsoft.com/)  site locate the datasets produced by this pattern.
    - There are 5 Power BI Datasets generated by the Azure Stream Analytics job named waypoint0-waypoint4 (which you created with the Azure Stream Analytics outputs).
	- For each datasest
		- Click on the dataset name under __Datasets__
		- Under __Fields__ select the checkbox for ***device_id***
		- Under __Visualizations__ select ***Gauge***
		- In __Visualizations__ blade and click on format (shaped like a paint roller)
			- Turn on __Title__
			- Change title text to: ALS*N* Count (N is the value between 0 and 4 that corresponds to current dataset)
		- Under the __File__ menu select ***Save As*** and name it with the name of the dataset (i.e. waypoint0-waypoint4)
		- Under __Reports__ click on the report created in the previous step
		- On the report, click on the Gauge then click the pin shaped icon 
			- If this is the first dataset, select ***New dashboard***, otherwise choose ***Existing dashboard*** and choose the dashboard you are creating.
		- Under __Dashboards__ click on the dashboard created for first dataset (waypoint0).
		- For each query provided in the next step
			- Enter the query, from below, in the **Ask a question about your data** control
			- Expand the __Visualizations__ blade and click on format (shaped like a paint roller)
			- Turn on __Title__
			- Enter the description provided with the query
			- Click __Pin Visual__ in the upper right hand corner
		- Queries (where *N* is the dataset you are working with)
			- How many device_id with label=1 in waypoint *N* in the last 30 seconds 
				- Description: No of ALS*N* fails in last 30 Seconds   
			- How many device_id with label=-1 in waypoint *N* in the last 30 seconds 
				- Description: No of ALS*N* passes in last 30 Seconds 
			- device_id with label=-1 in waypoint *N* in the last 30 seconds
				- Description: ALS*N* passes in last 30 seconds
			- device_id with label=1 in waypoint *N* in the last 30 seconds
				- Description: ALS*N* fails in last 30 seconds


## (optional) Set-up Azure Blob Storage sinks 				
 For any solution output, we can also create a blob storage sink, where data shown in PBI dashboard is also saved in text format (csv file).  
  - For any (or all) solution output, first create  containers in the solution blob storage account:
    - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
    - In the left hand menu select *Resource groups*
    - Locate the resource group you created for this project and click on it to display the resources associated with the solution resource group.
    - Click on the storage account of the solution resource group, then click **Containers**, and then **+Container** to add containers named outputsink0[N] (e.g. from outputsink00 to outputsink04). Set **Access type** property to **private**.

 Then, go to the solution Azure Stream Analytics (ASA) Job created before and connect the AML outputs to blob storage sinks created above:
    - Log into the [Azure Management Portal](https://ms.portal.azure.com) and select Stream Analytics Job was created for this solution (you have to stop the the ASA, if it is running, in order to add outputs as described below).
    - Click *OUTPUTS*
      - For each of the ml models, we will create a blob dataset output. In the following steps, complete one for each value of [N] where [N]= 0->4 
        - On the ASA *OUTPUTS* page click ***+ADD***.
          - OUTPUT ALIAS : w[N]blob (e.g. w0blob)
          - Sink: Blob storage
          - Subscription: Provide Storage account settings manually
          - Storage Account: paste name of Storage Account assigned to this solution resource group (use portal, click storage account and then click Access keys, copy **Storage account name**)
          - Storage Account Key: paste key info of Storage Account assigned to this solution resource group (use portal, click storage account and then click Access keys, copy **key1**)
          - Container: use container name created above (e.g. outputsink01).
          - Path Prefix Pattern: e.g. manufacturingtemplateoutput00/{date}/{time} to manufacturingtemplateoutput04/{date}/{time}
          - Date format: your choice (e.g. MM-DD-YYYY)
          - Time format: HH
          - Event Serialization Format: csv
          - Delimiter: comma 
          - Encoding: UTF8
          - Click **Create** button to complete	        
	      
 Extend the query of ASA Job created before to sent data to the blob sinks:
    - Log into the classic [Azure Managment Portal](https://manage.windowsazure.com) and select the ASA Job  created for this solution (you have to stop the ASA, if it is running, in order to add outputs as described below).
    - Append at the end of the query lines like:
       **SELECT evttime, conveyor as waypoint, device_id, resultw0 as label INTO [w0blob] FROM subqueryw0 WHERE conveyor = '1'**
       
       For example, to store all 5 datasets in blobs, we add 5 more lines, so that the last 10 lines of the query look like this:      
	      SELECT evttime, conveyor as waypoint, device_id, resultw0 as label INTO [w0pbi] FROM subqueryw0 WHERE conveyor = '1'
	      SELECT evttime, conveyor as waypoint, device_id, resultw1 as label INTO [w1pbi] FROM subqueryw1 WHERE conveyor = '2'
	      SELECT evttime, conveyor as waypoint, device_id, resultw2 as label INTO [w2pbi] FROM subqueryw2 WHERE conveyor = '3'
	      SELECT evttime, conveyor as waypoint, device_id, resultw3 as label INTO [w3pbi] FROM subqueryw3 WHERE conveyor = '4'
	      SELECT evttime, conveyor as waypoint, device_id, resultw4 as label INTO [w4pbi] FROM subqueryw4 WHERE conveyor = '5'
	      SELECT evttime, conveyor as waypoint, device_id, resultw0 as label INTO [w0blob] FROM subqueryw0 WHERE conveyor = '1'
	      SELECT evttime, conveyor as waypoint, device_id, resultw1 as label INTO [w1blob] FROM subqueryw1 WHERE conveyor = '2'
	      SELECT evttime, conveyor as waypoint, device_id, resultw2 as label INTO [w2blob] FROM subqueryw2 WHERE conveyor = '3'
	      SELECT evttime, conveyor as waypoint, device_id, resultw3 as label INTO [w3blob] FROM subqueryw3 WHERE conveyor = '4'
	      SELECT evttime, conveyor as waypoint, device_id, resultw4 as label INTO [w4blob] FROM subqueryw4 WHERE conveyor = '5'
  
  Finally, start the  ASA (log into the [Azure Management Portal](https://ms.portal.azure.com), select the solution ASA Job, select **Overview**, then click **Start** button, choose **Now** for the **Job output start time***, and then click **Start** button) and the data generator.     

            
            
	    
 

