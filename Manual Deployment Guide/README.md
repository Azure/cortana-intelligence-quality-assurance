# Quality Assurance for Manufacturing - Technical Deployment Guide  

Introduction
===============

The goal of the solution guide presented here is to predict failures in manufacturing pipelines (assembly lines), recognizing that manufacturing data is characterized by Assembly Line Steps (ALS). The idea is to leverage test systems already in place and failures data, specifically looking at returns and functional failures at the end of assembly line. By combining these with domain knowledge and root cause analysis within a modular design that encapsulates main processing steps, we provide a generic advanced analytics solution that uses machine learning to predict failures before they happen. Early prediction of future failures allows for less expensive repairs or even discarding, which are usually more cost efficient than going through recall and warranty cost.

The solution is implemented in the cloud, using the flexible on-line Microsoft Azure platform that decouples infrastructure components (data ingestion, storage, data movement, visualization) from analytics engine that supports modern data science languages like R and Python. The solution modeling component can thus be retrained as needed and be implemented using high performance Azure Machine Learning algorithms, or open source (R/Python) libraries, or from a third-party solution vendor.

This tutorial will guide you through the process creating, from the ground up, a predictive manufacturing solution. Other related resources provided here:
 -  a [solution description](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Manual%20Deployment%20Guide/SolutionDescription.md) that includes more details on the machine learning approach and related resources.
 -  [data flow](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Manual%20Deployment%20Guide/DataFlowReport.md) report demonstrating how post deployment monitoring can be used to visualize the timing and work-load of each key components of the solution.
 
**For technical problems or questions about deploying this solution, please post in the issues tab of the repository.**

Solution architecture description:
=================================
![Solution Diagram Picture](https://cloud.githubusercontent.com/assets/16708375/20932195/acb87330-bbcb-11e6-8a89-27d8b6e17bdf.png)

Solution design for Predictive Analytics for Quality Assurance Process in Manufacturing


  
Solution [Lambda](http://social.technet.microsoft.com/wiki/contents/articles/33626.lambda-architecture-implementation-using-microsoft-azure.aspx) architecture uses the hot (upper) path for real time processing and cold (lower) path for distributed processing that can handle complex queries on very large quantities of historical data.  
   
Speed Layer:  
 - Predictions are made for a individual products as they travel down a production line passing through 5 different ALS's.  
 - Event ingestion is performed using an [Azure Event Hub](https://azure.microsoft.com/en-us/documentation/articles/event-hubs-overview/) which receives simulated records  of ALS test measurements sent using an [Azure WebJob](https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/).  
 - Event processing is performed using an [Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) job which passes events to the appropriate [Azure Machine Learning](https://azure.microsoft.com/en-us/services/machine-learning/) endpoint and 
 pushes the results to [PowerBI](https://powerbi.microsoft.com/) datasets.  
  
Batch Layer:  
 - For large scale applications, solution scales out easily for complex processing and detailed visualizations of historical data by using Azure [SQL Data Warehouse](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-get-started-provision/), or [SQL Database](https://azure.microsoft.com/en-us/documentation/articles/sql-database-technical-overview/).  
 - Azure [Blob Storage](https://azure.microsoft.com/en-gb/documentation/articles/storage-blob-storage-tiers/#quick-start) sinks are used for storage.  
 

Deployment Steps:
====================

### Prerequisites

This tutorial will require:

 - An Azure subscription, which will be used to deoploy the project 
   (a [one-month free
   trial](https://azure.microsoft.com/en-us/pricing/free-trial/) is
   available for new users)
 - An [Azure Machine Learning Studio](https://studio.azureml.net/) account.
 - A Windows Desktop or as Windows based [Azure Virtual Machine](https://azure.microsoft.com/en-us/services/virtual-machines/) to run a data generation tool.    
 
### Naming Convention  

This deployment guide walks the readers through the creation of each of the Cortana Intelligence Suite services in the solution architecture defined in Section 2. As there are usually many interdependent components in a solution, [Azure Resource Manager](https://azure.microsoft.com/en-gb/features/resource-manager/) enables you to group all Azure services in one solution into a resource group. Each component in the resource group is called a resource. We want to use a common name for the different services we are creating. However, several services, such as Azure Storage, require a unique name for the storage account across a region and hence a naming convention is needed that should provide the user with a unique identifier. To address this, we suggest employing a base service name based on solution scope (manufacturing) and user's specific details like name and/or a custom numeric ID:  
 **manufact[UI][N]**  
  
where [UI] is the user's initials, N is a random integer that you choose and characters must be entered in in lowercase.  
  
To achieve this, all names used in this guide that contain string **manufacture** should be actually spelled as manufact[UI][N]. So for example, user Steven **X. Smith** might use a base service name of manufact**xs01**, and all services names below should follow the same naming pattern. For example, in the section "Create an Azure Event Hub" below: 
 - manufact***ureehns*** should actually be spelled manufact***xs01ehns*** 
 - manufact***ureeh*** should actually be spelled manufact***xs01eh***  
  
While in general the users can choose to follow this naming scheme or use their own naming, in the section "Create Azure Stream Analytics (ASA) Job", we note the functions of the Azure Stream Analytics (ASA) Job MUST be named waypoint**N** (i.e. waypoint0), where N = 0..4  matches the number from the ml endpoint. Changing the ASA function names is certainly possible, but it requires name changing for the ML services as well, which is beyond the scope of this project. 


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
 - Return back to *Resource groups* and choose the resource group for this solution.
 - Click on ***manufactureehns***, then on the subsequent blade, choose "Shared access policies", and click  __+Add__
 - Enter any appropriate name (e.g. ***myASAPolicy***), then select ***Manage***, ***Send*** and ***Listen***, and then click *Create*  

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
There are 5 waypoints which are the different stations in the assembly line with a predictive model developed for each one corresponding to the experiments below.  
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
 - When completed, click ***Deploy Web Service*** using the selection ***Deploy Web Service [Classic]***
 - After publishing all machine learning experiments, they should be all visible in the web services tab of Azure Machine Learning [Studio](https://studio.azureml.net/):
   
  ![Published Web Services](https://cloud.githubusercontent.com/assets/16708375/20488955/cfc73996-b000-11e6-8b56-7f56461090a2.png)
  
   
   Web Services tab in Azure Machine Learning [Studio](https://studio.azureml.net/)  

  
## Create Azure Stream Analytics (ASA) Job
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - In the left hand menu select *Resource groups*
 - Locate the resource group you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 - At the top of the Resource Group blade click __+Add__.
 - In the *Search Everything* search box enter ***Stream Analytics job***
 - Choose ***Stream Analytics job*** from the results then click *Create*
 - Enter ***manufactureasa*** as the name and choose the subscritpion, resource group and location using previous choices.
 - Click *Create* and return to the *Resource groups* blade.  
   
 - In [Azure Management Portal](https://ms.portal.azure.com), locate the solution *Resource group*.  
 - Select the ***manufactureasa*** resource to open the Stream Analytics job to modify the job settings.  
   
 - Click *INPUTS*  
    - At the top of the *INPUTS* page click ***+ADD***
        - INPUT ALIAS : InputHub
        - Source Type : Data Stream
        - Source : Event hub
        - SUBSCRIPTION: Use event hub from current subscription
        - Service bus namespace: ***manufactureehns*** (or whatever you have chosen for the __Event Hub**s**__ namespace previously)
        - Event hub name: ***manufactureeh*** (or whatever you have chosen for the event hub previously)
        - Event hub policy name: leave unchanged, or choose the policy name (e.g. ***myASAPolicy***) created above in the "Shared access policies" for the solution Event Hub.
        - Event hub consumer group: leave empty
        - Event serialization format : remains JSON
        - Encoding: remains UTF-8
        - Click the bottom **Create** button to complete.  
          
 - Click *FUNCTIONS*  
	 - Click ***+ADD*** at blade top.  
   - For each of the machine learning experiments we created  
	 - Enter the name waypoint**N** where N matches the number from the ml endpoint, for example waypoint0 for Prediction Model ALS01 (Prediction-Model-Mk-IIII-w-p-0-Predictive-Exp-1).    
	 - Function Type: remains **Azure ML**  
	 - If you can access the ml points from the same subscription as in following instructions set:  
		 - Subscription: remains **Select from the same subscription**  
		 - Select the ML workspace created earlier (manufacturemlws or whatever you have chosen for the ***Machine Learning Workspace*** previously).  
	 - Choose an ml endpont (for example Prediction Model Mk IIII (w.p. 0) [Predictive Exp.])  
		 - Click the check button to complete.  
	 - If, however, the ML points created before are not visible under current subscription:  
		 - Subscription: set to **Import from a different subscription**  
		 - Then go to web services tab of Azure Machine Learning [Studio](https://studio.azureml.net/) (see red arrows in image below):  
 
![Copy Published services credentials](https://cloud.githubusercontent.com/assets/16708375/20528296/927a035c-b0c3-11e6-98da-79276d8c710b.png)


Published AML Services credentials

   

  - URL: click **REQUEST/RESPONSE** hyperlink and copy the **Request URI** for the **POST Method**, e.g:  
    https://europewest.services.azureml.net/workspaces/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/services/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/execute?api-version=2.0&details=true  
  - Key: Click the copy **APi key icon** to copy the API key and the **REQUEST/RESPONSE** hyperlink  
  - Click **Create** button to complete.
  
The final list of ASA functions show the increasing number of features as we move along ALS line, where for example ALS1 model (waypoint1) uses features (total 102) from both ALS1 (50) and ALS0 (52), but not from later ALS-s like ALS2, or ALS4.
 
![ASA Functions](https://cloud.githubusercontent.com/assets/16708375/22430020/9c92bd4e-e704-11e6-9dcd-98756bc0865c.png)


ASA functions properties show the increasing number of features as we move along ALS line.

  
  
 - Click *OUTPUTS*  
   - For each of the ml models, we will create a PowerBI dataset output. In the following steps, complete one for each value of [N] where [N]= 0,1,2,3,4  
	   - Select ASA *OUTPUTS* blade and click ***+ADD***  
	     - OUTPUT ALIAS : w[N]pbi  
	     - Sink: Power BI, then Click **Authorize Now** to access your Power BI data.  
	     - WORKSPACE : My Workspace  
	     - DATASET NAME : waypoint[N]  
	     - TABLE NAME : waypoint[N]  
	     - Click the **Create** button to complete  
  
- Click *QUERY*  
    - Download the file StreamAnalyticsJob.txt from the [resources folder](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Manual%20Deployment%20Guide/resources) of this repository. Copy and paste the content into the query window.  
    - Click *SAVE*  
- When all inputs, functions, outputs and the query have been entered, click *START* at the top of the Overview page.   
  
  
## Download and configure the data generator  
 - Download the file ***ManufacturingGenerator.zip*** from the [resources folder](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Manual%20Deployment%20Guide/resources) of this repository.  
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
    ***NOTE:*** The following PowerBI Dashboards will only be dynamically updated when this generator is running.  
    ***NOTE:*** Data generator can also be run in the cloud, using an Azure [Virtual Machine](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-hero-tutorial). For some of the snapshots we show here, a Windows Server 2008 R2 SP1 [Virtual Machine](https://azure.microsoft.com/en-us/marketplace/virtual-machines/) was used with A4 Basic (8 Cores, 14 GB, 16 Data disks, 16x300 Max IOPS) configuration.


## Configure a Power BI dashboard
 You can go to [Power BI Dashboard](https://powerbi.microsoft.com/) and use a Real-time dataset to build reports and dashboards using your data! This section is about creating a real-time dashboard through connecting Azure Stream Analytics queries to Power BI. There is another example of a Power BI Dashboard below in the SQL Data Warehouse section, where it is described how one can build dashboards using Power BI Desktop on top of data sitting in a database.
 
  - On the [Power BI Dashboard](https://powerbi.microsoft.com/)  site locate the datasets produced by this pattern.
    - There are 5 Power BI Datasets generated by the Azure Stream Analytics job named waypoint0-waypoint4 (which you created with the Azure Stream Analytics outputs).
	- For each datasest
		- Under __Datasets -> Streaming datasets__, click on __Create report__ under __Actions__ across the dataset name 
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
			- How many device_id with label=1 in waypoint*N* in the last 30 seconds 
				- Description: No of ALS*N* fails in last 30 Seconds  
			- How many device_id with label=-1 in waypoint*N* in thelast 30 seconds 
				- Description: No of ALS*N* passes in last 30 Seconds 
			- device_id with label=-1 in waypoint *N* in the last 30 seconds
				- Description: ALS*N* passes in last 30 seconds
			- device_id with label=1 in waypoint *N* in the last 30 seconds
				- Description: ALS*N* fails in last 30 seconds

![PBIASA Dashboard](https://cloud.githubusercontent.com/assets/16708375/19904139/5cacc6e8-a069-11e6-830c-d42c3a94e678.png)

         
## Set-up Azure SQL Data Warehouse sinks 				
 For large scale OLAP applications that require scalable storage and processing of massive volumes of data via complex aggregation  queries, the solution uses [Azure SQL Data Warehouse](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-overview-what-is/), where data can be later processed and visualized using massively parallel processing architecture.  
   
### Create a [SQL Data Warehouse](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-get-started-provision/), a cloud-based, scale-out database capable of processing massive volumes of data, both relational and non-relational.  
- Log into the [Azure Management Portal](https://ms.portal.azure.com), and Click + New > Databases > SQL Data Warehouse.  
- In the SQL Data Warehouse blade, fill in the information needed as described below, starting with the **Server** first:  
- Server: click **Create a new server**  
  - Server name: e.g. __manufacturesqldbserver02__ . A green check mark indicates that you have provided a valid name.    
  - Server admin login:  A green check mark indicates that you have provided a valid name.  
  - Password:  A green check mark indicates that you have provided a valid password.  
  - Location: choose the same location as the other solution resources  
- Database name (this will be a **SQL data warehouse** type of db): e.g. Manufacturesqldb.  
- Subscription: Select the subscription that this SQL Data Warehouse will bill to.  
- Resource group: select the reource group created for this solution.  
- Select source: leave it **blank database**.  
- Collation: remains SQL_Latin1_General_CP1_CI_AS  
- Performance: We recommend starting with 400 DWUs. You can move the slider to the left or right to adjust the performance of your data warehouse, or scale up or down after creation. To learn more about DWUs, see our documentation on scaling or our pricing page.  

After you click **Create** button, you check deploymnet status, in the Azure portal, e.g. by looking at the solution resource group **Overview** blade, under **Deployments** section, we should see a clickable status saying initally **1 deploying, 3 succeeded**.  
  
### Create a new Azure SQL server-level firewall:  
  Use the following steps in the Azure portal to create a server-level firewall rule that allows connections from an individual IP address (your client computer) or an entire IP address range to a SQL Database logical server.
  - Log into the [Azure Management Portal](https://ms.portal.azure.com), 
  - In the left hand menu select *Resource groups*
  - Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the  resource group blade.
  - You should see the SQL server and the SQL database you just created.
  - Click the SQL server, and then click "Firewall".
  - Make sure the "Allow access to Azure services" is set to ON, and then press **+Add client IP** button at the top of the blade to have Azure create a rule for your client IP address. Optionally, to allow access to a range of IP addresses, click the IP address that was added to edit the firewall address.
  - Click Save to create the server-level firewall rule.
  - You should now be able to [access](https://azure.microsoft.com/en-us/documentation/articles/sql-database-connect-query-ssms/) the SQL server using a tool like [Microsoft SQL Server Management Studio](https://msdn.microsoft.com/en-us/library/ms174173.aspx).  
  
  
#### As an OLTP alternative to SQL DW, if processing requirements are focused not on complex aggregations but on a large number of short on-line transactions, data can be stored in a [SQL Database](https://azure.microsoft.com/en-us/documentation/articles/sql-database-technical-overview/), a relational database service in the cloud based on Microsoft SQL Server engine:

![create Azure SQL database](https://cloud.githubusercontent.com/assets/16708375/20218783/39817dac-a7f4-11e6-9cc4-0544f5fd7bc4.png)  


Create the [Azure SQL](https://azure.microsoft.com/en-us/documentation/articles/sql-database-get-started/#create-an-azure-sql-database-logical-server) database  
 
  
  	      
 Log into the [Azure Management Portal](https://ms.portal.azure.com), click __+(New)__ button, click databases and then select __SQL Database__.  
  - Database name: provide a name for database - such as __manufacturesqldb__  
  - Subscription: use your subscription  
  - Resource group(select __Use existing__): Locate the resource group  you created for this project  
  - Select source: Blank database  
  - Under __Server__, click Configure required settings:  
	  - Server name: provide a name for server - such as __manufacturesqldbserver__. A green check mark indicates that you have provided a valid name  
	  - Server admin login: A green check mark indicates that you have provided a valid name.   
	  - Password: A green check mark indicates that you have provided a valid password.   
	  - Location: choose same location you used for other solution resources. 
	  - Check __Allow azure services to access server__  
	  - Press __Select__ button at the blade buttom.  
	  - Leave __Want to use SQL elastic pool?__ unchanged ("__Not now__").  
	  - On the SQL Database blade, under __Pricing tier__, choose Basic to choose the least expensive pricing tier for your first database. You can always change the pricing tier later.   
	 Press __Select__ button at the blade buttom.    
	  - On the SQL Database blade, review your selections and then click Create to create the server and database. The values that you provided are validated and deployment starts.  
	  - Similar to DW deployment approach above, create a new Azure SQL server-level firewall using the Azure portal: create a server-level firewall rule that allows connections from an individual IP address (your client computer) or an entire IP address range to a SQL Database logical server.   


### After the SQL Data Warehouse is deployed, create empty tables in it (using full data solution schema)
 Create empty tables in the SQL WH/DB that will store both the the features (ALS test measurement values) and the predictions. The label column in the input data was used for training and is ignored in the scoring experiment.
  - Use your favourite tool to connect to the database created above. A good option is [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/en-us/library/mt238290.aspx).
  - Run the queries in the SQLDatabaseQueries_InputData_ALS0[**N**].sql (**N** = 0..4) files attached to this solution deployment.
  After you run the queries, you should have 5 tables AllDataALS0[**N**], (**N** = 0..4) in the database.
  - [OPTIONAL] In general, you can use **SQL Server 2016 Import and Export Data** tool to create SQL database data tables using schema from sample csv files.  
            - Use **SQL Server 2016 Import and Export Data** (you can install the SQL Server Import and Export Wizard by installing SQL Server Data Tools ([SSDT](https://msdn.microsoft.com/library/mt204009.aspx))).
            - Use the traing data sample (in the solution [resources directory](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Technical%20Deployment%20Guide/resources)), you can run **SQL Server 2016 Import and Export Data**:
            	- Data source: select **Flat File Source** as input and choose the waypoint**N**_training.csv file (**N** = 0..4), then press **Next**  
            	- Destination: Select  **Microsoft OLE DB Provider for SQL server**  
            	- Select **Use SQL Server Authetication**, and type admin user name and password  
            	- Press **Edit Mappings** button  
            	- Press **Edit SQL...** button and copy and save the autogenerated SQL query.  
            	- You can continue to upload the csv file to the database, or just use the autogenerated SQL query in SSMS to create an empty database table with the csv file schema.  
            	NOTE: The automatically generated SQL queries will generally use non-specific varchar data type for all fields in the created tables. For data fields that require non-generic types, the SQL query autogenerated above should be edited to specify the desired type for each field.
  
  
### Connect the ASA to the SQL DW by adding **SQL database** outputs for the ASA job (similar to the steps preformed when adding PBI output sinks):  
  - Log into the [Azure Management Portal](https://ms.portal.azure.com) and select the Stream Analytics Job created for this solution (you have to stop the the ASA, if it is running, in order to add outputs as described below).  
  - Click *OUTPUTS* and then click ***+ADD***  
    - Output Alias: dbsinkALS0**N** (**N** = 0..4).  
    - Sink: **SQL database**  
    - Subscription: choose solution subscription  
    - Database: choose the database created for this solution  
    - Fill Server name, Username, Password
    - Table: choose AllDataALS0[**N**], (**N** = 0..4) or the table names you created in the SQL db/wh before  
    - Click **Create** button.  
  - Extend the query of ASA Job created before to sent data to the DW sinks:  
 	 - Log into the classic [Azure Managment Portal](https://manage.windowsazure.com) and select the ASA Job  created for this solution (you have to stop the ASA, if it is running, in order to add outputs as described below).   
 	 - Append at the end of the query lines like:  
  
       **SELECT * INTO [dbsinkALS00] FROM subqueryw0 WHERE conveyor = '1'**  
   
	For example, to store all 5 datasets in the DW, we add 5 more lines, so that the last 10 lines of the ASA query look like this:   
       
       
	      SELECT evttime, conveyor as waypoint, device_id, resultw0 as label INTO [w0pbi] FROM subqueryw0 WHERE conveyor = '1'
	      SELECT evttime, conveyor as waypoint, device_id, resultw1 as label INTO [w1pbi] FROM subqueryw1 WHERE conveyor = '2'
	      SELECT evttime, conveyor as waypoint, device_id, resultw2 as label INTO [w2pbi] FROM subqueryw2 WHERE conveyor = '3'
	      SELECT evttime, conveyor as waypoint, device_id, resultw3 as label INTO [w3pbi] FROM subqueryw3 WHERE conveyor = '4'
	      SELECT evttime, conveyor as waypoint, device_id, resultw4 as label INTO [w4pbi] FROM subqueryw4 WHERE conveyor = '5'
	      SELECT * INTO [dbsinkALS00] FROM subqueryw0 WHERE conveyor = '1'
	      SELECT * INTO [dbsinkALS01] FROM subqueryw1 WHERE conveyor = '2'
	      SELECT * INTO [dbsinkALS02] FROM subqueryw2 WHERE conveyor = '3'
	      SELECT * INTO [dbsinkALS03] FROM subqueryw3 WHERE conveyor = '4'
	      SELECT * INTO [dbsinkALS04] FROM subqueryw4 WHERE conveyor = '5'  
  
	
  
  
  **NOTE**: Each of the above TSQL lines can be commented if needed by adding **--** at the begining of the line.  
  
  - Finally, start the  ASA (log into the [Azure Management Portal](https://ms.portal.azure.com), select the solution ASA Job, select **Overview**, then click **Start** button, choose **Now** for the **Job output start time***, and then click **Start** button) and the data generator.     
  
Once data has been flowing into the data warehouse, [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) can be used to build visualizations of the historical data (and a tool such as SQL Server Management Studio can be used to query and understand the data).

## **Visualization**
Power BI dashboard can be used to visualize the real-time failure predictions. The following instructions will guide you to build a dashboard to visualize data from database and from real-time data stream.

### Visualize Data from Database
The essential goal of this part is to get the failure predictions and visualize it. Power BI can directly connect to an Azure SQL DW as its data source, where the prediction results are stored. Note that this is mainly for visualizing historical predictions rather than acting upon predictions in real time. For real-time visualizations on the predicted failures as they pass through the assembly line, further instructions are given below on visualizing streaming datasets.

> Note: 1) In this step, the prerequisite is to download and install the free software [Power BI desktop](https://powerbi.microsoft.com/desktop). 2) We recommend you start this process 2-3 hours after you deploy the solution so that you have more data points to visualize.

1. Get the database credentials. 
You can find your database and server name on the page when you finish your deployment. The SQL username and password will be the ones you choose in the beginning of the deployment.

2. Update the data source of the Power BI file

- Make sure you have installed the latest version of [Power BI desktop](https://powerbi.microsoft.com/desktop). 
 
- In this GitHub repository, you can download the **'ManuQualityAssurance.pbix'** file under the folder **'Automated Deployment Guide'** and then open it. **Note:** If you see an error massage, please make sure you have installed the latest version of Power BI Desktop.
 
- On the top of the file, click **‘Edit Queries’** drop down menu. Then choose **'Data Source Settings'**. 
 
- In the pop out window, click **'Change Source'**, then replace the **"Server"** and **"Database"** with your own server and database names and click **"OK"**. For server name, make sure you specify the port 1433 in the end of your server string (**YourSolutionName.database.windows.net, 1433**). After you finish editing, close the 'Data Source Settings' window.
 
- On the top of the screen, you will see a message. Click **'Apply Changes'**. A new window will pop out and ask for database credentials. Click **'Database'** on the left of the window, enter your SQL credentials. For ***'Select which level to apply these settings to'***, choose the second one with database name. Then click ***'Connect'***.
 
- Now the dashboard is updated to connect to your database. In the backend, model is scheduled to be refreshed every 1 hour. You can click **'Refresh'** button on the top to get the latest visualization as time moving forward.

3. (Optional) Publish the dashboard to [Power BI online](http://www.powerbi.com/). 
Note that this step needs a Power BI account (or Office 365 account).
	- Click **"Publish"** on the top panel. Choose **'My Workspace'** and few seconds later a window appears displaying "Publishing succeeded". 
	- Click the link on the screen to open it in a browser. On the left panel, go to the **Dataset** section, right click the appropriate dataset, choose **Dataset Settings**. In the pop out window, click **Enter credentials** and enter your database credentials by following the instructions. To find detailed instructions, please see [Publish from Power BI Desktop](https://support.powerbi.com/knowledgebase/articles/461278-publish-from-power-bi-desktop).
	- Now you can see new items showing under 'Reports' and 'Datasets'. To create a new dashboard: click the **'+'** sign next to the **Dashboards** section on the left pane. Enter the name "Energy Demand Forecasting Demo" for this new dashboard. 
	- Once you open the report, click ![Pin](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Automated%20Deployment%20Guide/Figures/PowerBI-4.png?raw=true) to pin all the visualizations to your dashboard. To find detailed instructions, see [Pin a tile to a Power BI dashboard from a report](https://support.powerbi.com/knowledgebase/articles/430323-pin-a-tile-to-a-power-bi-dashboard-from-a-report). 

An example visualization is below, and the [Power BI Desktop file](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Technical%20Deployment%20Guide/resources/ManuQualityAssurance.pbix) that was used to create this is included in the [resources folder](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Technical%20Deployment%20Guide/resources/) of this repository.
![PBI](https://cloud.githubusercontent.com/assets/9042064/20733041/891bb5f2-b65f-11e6-8362-068b9ba92698.PNG)
	   
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
    - For each of the ml models, we will create a blob dataset output. In the following steps, complete one for each value of [N] where [N]= 0->4 (you can use a new web browser tab to see the storage account stting in [Azure Management Portal](https://ms.portal.azure.com))  
      - On the ASA *OUTPUTS* page click ***+ADD***.  
        - OUTPUT ALIAS : w[N]blob (e.g. w0blob)  
        - Sink: Blob storage  
        - Subscription: Provide Storage account settings manually  
        - Storage Account: paste name of Storage Account assigned to this solution resource group (use portal, click storage account and then click Access keys, copy **Storage account name**)  
        - Storage Account Key: paste key info of Storage Account assigned to this solution resource group (use portal, click storage account and then click Access keys, copy **key1**)  
        - Container: use container name created above (e.g. outputsink01).  
        - Path Prefix Pattern: manufacturingtemplateoutput0[N]/{date}/{time}, N = 0..4 (e.g. manufacturingtemplateoutput00/{date}/{time} to manufacturingtemplateoutput04/{date}/{time})  
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
       
       For example, to store all 5 datasets in blobs, we add 5 more lines, so that the last 15 lines of the query look like this:   
       
       
	      SELECT evttime, conveyor as waypoint, device_id, resultw0 as label INTO [w0pbi] FROM subqueryw0 WHERE conveyor = '1'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw1 as label INTO [w1pbi] FROM subqueryw1 WHERE conveyor = '2'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw2 as label INTO [w2pbi] FROM subqueryw2 WHERE conveyor = '3'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw3 as label INTO [w3pbi] FROM subqueryw3 WHERE conveyor = '4'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw4 as label INTO [w4pbi] FROM subqueryw4 WHERE conveyor = '5'  
	      SELECT * INTO [dbsinkALS00] FROM subqueryw0 WHERE conveyor = '1'
	      SELECT * INTO [dbsinkALS01] FROM subqueryw1 WHERE conveyor = '2'
	      SELECT * INTO [dbsinkALS02] FROM subqueryw2 WHERE conveyor = '3'
	      SELECT * INTO [dbsinkALS03] FROM subqueryw3 WHERE conveyor = '4'
	      SELECT * INTO [dbsinkALS04] FROM subqueryw4 WHERE conveyor = '5'
	      SELECT evttime, conveyor as waypoint, device_id, resultw0 as label INTO [w0blob] FROM subqueryw0 WHERE conveyor = '1'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw1 as label INTO [w1blob] FROM subqueryw1 WHERE conveyor = '2'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw2 as label INTO [w2blob] FROM subqueryw2 WHERE conveyor = '3'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw3 as label INTO [w3blob] FROM subqueryw3 WHERE conveyor = '4'  
	      SELECT evttime, conveyor as waypoint, device_id, resultw4 as label INTO [w4blob] FROM subqueryw4 WHERE conveyor = '5'  
    
  **NOTE**: Each of the above TSQL lines can be commented if needed by adding **--** at the begining of the line.    
  
  Finally, start the  ASA (log into the [Azure Management Portal](https://ms.portal.azure.com), select the solution ASA Job, select **Overview**, then click **Start** button, choose **Now** for the **Job output start time***, and then click **Start** button) and the data generator.     
  
### After the solution is deployed and is running, you can see live updates of the PBI dashboards, connect to the DW and run queris to retrieve data, or check the blob files. Queries like the ones below can be run within [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/en-us/library/mt238290.aspx):    
       
       
	      SELECT TABLE_NAME FROM Manufacturesqldb.INFORMATION_SCHEMA.Tables WHERE TABLE_TYPE = 'BASE TABLE'
	      SELECT TOP 3 * FROM AllDataALS00
	      
DW Answer:  
  
TABLE_NAME  
AllDataALS00  
AllDataALS01  
AllDataALS02  
AllDataALS04  
AllDataALS03

evttime	device_id	conveyor	col106	...  
2016-11-23T09:06:47.7604884Z	device_55	1	0.00999733494766	...  
2016-11-23T09:13:28.2651491Z	device_135	1	0.00983559370077	...  
2016-11-23T09:10:37.9897048Z	device_101	1	-0.0111869417915	...  


  **NOTE**: These queries will work in older versions of [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/en-us/library/mt238290.aspx), even if the WS tables may not be visible. For best performance and experience (including being able to see the tables in SSMS GUI, we recommed the latest [SSMS version](https://msdn.microsoft.com/en-us/library/mt238290.aspx)).



