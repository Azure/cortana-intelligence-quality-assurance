# Cortana Intelligence Suite Predictive Manufacturing Solution  

1. Introduction
===============

This tutorial will guide you through the process creating, from the ground up, a predictive manufacturing solution. Other related resources provided here:
 -  an extensive [solution description](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Technical%20Deployment%20Guide/SolutionDescription.md).
 -  [data flow](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Technical%20Deployment%20Guide/DataFlowReport.md) report demonstrating how post deployment monitoring can be used to visualize the timing and work-load of each key components of the solution.
 
**For technical problems or questions about deploying this solution, please post in the issues tab of the repository.**

2. Solution architecture description:
=====================================
 
![Solution Diagram Picture](https://cloud.githubusercontent.com/assets/16708375/20287777/95b6a66a-aa9c-11e6-8bae-c39dc543eceb.png)

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
 - After publishing all machine learning experiments, they should be all visible in the web services tab of Azure Machine Learning [Studio](https://studio.azureml.net/):
   
  ![Published Web Services](https://cloud.githubusercontent.com/assets/16708375/20488955/cfc73996-b000-11e6-8b56-7f56461090a2.png)
  
   <sub>
   Web Services tab in Azure Machine Learning [Studio](https://studio.azureml.net/)  
  </sup>
  
## Create Azure Stream Analytics (ASA) Job
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - In the left hand menu select *Resource groups*
 - Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 - At the top of the Resource Group blade click __+Add__.
 - In the *Search Everything* search box enter ***Stream Analytics job***
 - Choose ***Stream Analytics job*** from the results then click *Create*
 - Enter ***manufactureasa*** as the name and choose the subscritpion, resource group and location using previous choices.
 - Click *Create* and return to the *Resource groups* blade.  
   
 - In [Azure Management Portal](https://ms.portal.azure.com), locate the solution *Resource group*.  
 - Select the ***manufactureasa*** resource to open the Stream Analytics job to modify the job settings.  
   
 - Click *INPUTS*  
    - At the bottom of the *INPUTS* page click ***+ADD***
        - INPUT ALIAS : InputHub
        - Source Type : Data Stream
        - Source : Event hub
        - SUBSCRIPTION: Use event hub from current subscription
        - Service bus namespace: ***manufactureehns*** (or whatever you have chosen for the __Event Hub**s**__ namespace previously)
        - Event hub name: ***manufactureeh*** (or whatever you have chosen for the event hub previously)
        - Event hub policy name: leave unchanged
        - Event hub consumer group: leave empty
        - Event serialization format : remains JSON
        - Encoding: remains UTF-8
        - Click the bottom **Create** button to complete.  
          
 - Click *FUNCTIONS*  
        - Click ***+ADD*** at blade top.  
    - For each of the machine learning experiments we created  
        - Enter the name waypoint**N** where N matches the number from the ml endpoint, for example (w.p. 0), N = 0..4  
        - Function Type: remains **Azure ML**  
	- If you can access the ml points from the same subscription as in following instructions set:  
		- Subscription: remains **Select from the same subscription**  
		- Select the ML workspace created earlier (manufacturemlws or whatever you have chosen for the ***Machine Learning Workspace*** previously).  
		- Choose an ml endpont (for example Prediction Model Mk IIII (w.p. 0) [Predictive Exp.])  
		- Click the check button to complete.  
	- If, however, the ML points created before are not visible under current subscription:  
		- Subscription: set to **Select from a different subscription**  
		- Then go to web services tab of Azure Machine Learning [Studio](https://studio.azureml.net/) (see red arrows in image below):  
   
![Copy Published services credentials](https://cloud.githubusercontent.com/assets/16708375/20528296/927a035c-b0c3-11e6-98da-79276d8c710b.png)

<sub>
Published AML Services credentials
</sup>
  
		- URL: click **REQUEST/RESPONSE** hyperlink and copy the **Request URI** for the **POST Method**, e.g:  
			https://europewest.services.azureml.net/workspaces/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/services/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/execute?api-version=2.0&details=true  
		- Key: Click the copy **APi key icon** to copy the API key and the **REQUEST/RESPONSE** hyperlink  
		- Click **Create** button to complete.  
  
 - Click *OUTPUTS*  
    - For each of the ml models, we will create a PowerBI dataset output. In the following steps, complete one for each value of [N] where [N]= 0..4  
        - Select ASA *OUTPUTS* blade and click ***+ADD***  
            - OUTPUT ALIAS : w[N]pbi  
            - Sink: Power BI, then Click **Authorize Now** to access your Power BI data.  
            - WORKSPACE : My Workspace  
            - DATASET NAME : waypoint[N]  
            - TABLE NAME : waypoint[N]  
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
  
         
## (optional) Set-up Azure SQL Data Warehouse sinks 				
 For large scale applications that require scalable storage and processing of massive volumes of data, the solution uses [Azure SQL Data Warehouse](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-overview-what-is/), where data can be later processed and visualized using massively parallel processing architecture.  
   
### Create a [SQL Data Warehouse](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-get-started-provision/), a cloud-based, scale-out database capable of processing massive volumes of data, both relational and non-relational.       
	- Log into the [Azure Management Portal](https://ms.portal.azure.com), and Click + New > Databases > SQL Data Warehouse.  
	  - In the SQL Data Warehouse blade, fill in the information needed as described below, starting with the **server** first:  
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
	  - After you click **Create** button, you check deploymnet status, in the Azure portal, e.g. by looking at the solution resource group **Overview** blade, under **Deployments** section, we should see a clickable status saying initally **1 deploying, 3 succeeded**.  

	- Create a new Azure SQL server-level firewall:  
	Use the following steps in the Azure portal to create a server-level firewall rule that allows connections from an individual IP address (your client computer) or an entire IP address range to a SQL Database logical server.  
		- Log into the [Azure Management Portal](https://ms.portal.azure.com), 
		- In the left hand menu select *Resource groups*
		- Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
		- You should see the SQL server and the SQL database you just created.
		- Click the SQL server, and then click "Firewall".
		- Make sure the "Allow access to Azure services" is set to ON, and then press **+Add client IP" button at the top of the blade to have Azure create a rule for your client IP address. Optionally, to allow access to a range of IP addresses, click the IP address that was added to edit the firewall address.
		- Click Save to create the server-level firewall rule.
		- You should now be able to [access](https://azure.microsoft.com/en-us/documentation/articles/sql-database-connect-query-ssms/) the SQL server using a tool like [Microsoft SQL Server Management Studio](https://msdn.microsoft.com/en-us/library/ms174173.aspx).  
    
  
#### As a less powerful DW alternative, data can be stored in a [SQL Database](https://azure.microsoft.com/en-us/documentation/articles/sql-database-technical-overview/), a relational database service in the cloud based on Microsoft SQL Server engine:

![create Azure SQL database](https://cloud.githubusercontent.com/assets/16708375/20218783/39817dac-a7f4-11e6-9cc4-0544f5fd7bc4.png)  

<sub>
Create the [Azure SQL](https://azure.microsoft.com/en-us/documentation/articles/sql-database-get-started/#create-an-azure-sql-database-logical-server) database  
</sup>  
  
  	      
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
	  - Create V12 server (Latest update): Yes  
	  - Check __Allow azure services to access server__  
	  - Press __Select__ button at the blade buttom.  
	  - Leave __Want to use SQL elastic pool?__ unchanged ("__Not now__").  
	  - On the SQL Database blade, under __Pricing tier__, choose Basic to choose the least expensive pricing tier for your first database. You can always change the pricing tier later.   
	 Press __Select__ button at the blade buttom.    
	  - On the SQL Database blade, review your selections and then click Create to create the server and database. The values that you provided are validated and deployment starts.  
	  - Similar to DW deployment approach above, create a new Azure SQL server-level firewall using the Azure portal: create a server-level firewall rule that allows connections from an individual IP address (your client computer) or an entire IP address range to a SQL Database logical server.
	  
	  
### After the SQL Data Warehouse is deployed, create empty tables in it (using full data solution schema):  
	- Create empty tables in the SQL WH/DB that will store both the the features (ALS test measurement values) and the predictions. The label column in the input data was used for training and is ignored in the scoring experiment.            
	  - Use your favourite tool to connect to the database created above. A good option is [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/en-us/library/mt238290.aspx).  
	  - run the queries in the SQLDatabaseQueries_InputData_ALS0[**N**].sql (**N** = 0..4) files attached to this solution deployment.  
	  After you run the queries, you should have 5 tables AllDataALS0[**N**], (**N** = 0..4) in the database.  
	  - [OPTIONAL] In general, you can use **SQL Server 2016 Import and Export Data** to create SQL database data tables using schema from csv tables.  
		-  you can install the SQL Server Import and Export Wizard by installing SQL Server Data Tools ([SSDT](https://msdn.microsoft.com/library/mt204009.aspx))  
	    use **SQL Server 2016 Import and Export Data** (you can install the SQL Server Import and Export Wizard by installing SQL Server Data Tools ([SSDT](https://msdn.microsoft.com/library/mt204009.aspx))).   
		- If you have a sample of the data you wish to store in the database (for example obtained by running the solution and saving the full data in a blob storage account as described above), you can run **SQL Server 2016 Import and Export Data**:   
			- Data source: select **Flat File Source** as input and choose the waypoint**N**_training.csv file (**N** = 0..4), then press **Next**  
			- Destination: Select  **Microsoft OLE DB Provider for SQL server**  
			- Select **Use SQL Server Authetication**, and type admin user name and password  
			- Press **Edit Mappings** button  
			- then press **Edit SQL...** button and copy and save the autogenared SQL query.  
			- You can continue to upload the csv file to the database, or just use the autogenerated query to created and empty database table with the csv file schema.   
            
      
### Connect the ASA to the SQL DW by adding **SQL database** outputs for the ASA job (similar to the steps preformed when adding PBI output sinks):  
	- Output Alias: dbsinkALS0[**N**] (**N** = 0..4).  
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
  NOTE: in TSQL, **--** at the begining of a line comments that line. So each of the above lines can be easily commented by adding -- at the begining of the line.  
  Finally, start the  ASA (log into the [Azure Management Portal](https://ms.portal.azure.com), select the solution ASA Job, select **Overview**, then click **Start** button, choose **Now** for the **Job output start time***, and then click **Start** button) and the data generator.     
	   
	   

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
    
  NOTE: in TSQL, **--** at the begining of a line comments that line. So each of the above lines can be easily commented by adding -- at the begining of the line.  
  Finally, start the  ASA (log into the [Azure Management Portal](https://ms.portal.azure.com), select the solution ASA Job, select **Overview**, then click **Start** button, choose **Now** for the **Job output start time***, and then click **Start** button) and the data generator.     
  
### After the solution is deployed and is running, you can see live updates of the PBI dashboards, check the blob files, or connect to the DW and run queris to retrieve data:   
       
       
	      SELECT TABLE_NAME FROM Manufacturesqldb.INFORMATION_SCHEMA.Tables WHERE TABLE_TYPE = 'BASE TABLE'
	      SELECT TOP 3 * FROM AllDataALS00
	      
DW Answer:

TABLE_NAME
AllDataALS00
AllDataALS01
AllDataALS02
AllDataALS04
AllDataALS03

evttime	device_id	conveyor	col106	col11	col12	col137	col168	col182	col183	col198	col21	col210	col219	col222	col232	col240	col241	col245	col253	col256	col274	col278	col28	col282	col29	col292	col3	col304	col308	col319	col326	col341	col347	col351	col352	col356	col357	col362	col364	col373	col382	col385	col392	col404	col415	col423	col431	col433	col57	col71	col79	col86	labels	resultw0
2016-11-23T09:06:47.7604884Z	device_55	1	0.00999733494766	-1.2904526375	0.204316776735	-0.368778876439	0.0520253212335	-0.465586375061	0.0222098860902	-0.0229062387125	-0.0180120676383	0.757616590765	0.170885832618	-0.239037370745	0.570323565695	-0.0226634101919	0.237666373353	1.02048953433	0.000243834400369	0.24164924501	0.00266628566355	0.615921724504	-0.0558100302686	-0.104704482877	-0.223702143986	0.011510315221	-1.00055009855	0.0546819858722	0.00447698196494	-0.0832292520235	0.166671122544	0.399742686835	-0.0781922958215	0.00625963667257	-0.000228008521632	0.317848141357	0.0254309068122	-0.00286124534837	-0.0010561969475	-0.00190154185493	0.86576051645	0.0107683594411	-1.34880068418	-0.00816269636356	-1.82956632792	1.82284331612	-1.98994767214	0.422980104239	0.0922600049726	0.0150213898564	-1.01083725661e-05	0.163313355841	-1	-1
2016-11-23T09:13:28.2651491Z	device_135	1	0.00983559370077	-1.14733845917	0.0378400268284	0.0360682766637	0.0883600541488	0.554904768271	-0.00162810572849	0.0233050777961	-0.00599875837892	-1.2679737398	0.386348680851	-0.0184109355799	0.288047011578	0.0059084001448	-0.27300373006	0.0872870462324	-0.00590691604634	-0.158331185666	-0.018151312731	-0.518488601609	0.841967713637	-0.709701797745	0.443937739641	-0.0434182798854	0.485203285753	1.98334216542	-0.0125008009394	-0.0320867619319	-0.86578779744	-0.71765326375	0.114908456291	-0.0238512315713	-0.000126192404431	-0.19215541874	0.325999253631	0.233330272317	-0.0238066885624	-0.120773004412	-0.536344586541	0.00236076283761	0.0481723300073	0.00508190082219	0.333186037952	1.24390738587	-0.539052025347	0.736051353159	0.00488133147767	-0.0306786566931	0.000201316653995	-1.5792988698	-1	-1
2016-11-23T09:10:37.9897048Z	device_101	1	-0.0111869417915	-2.3330857958	-0.239868245565	0.409914872055	0.0882435084078	-0.137122886792	0.0388608591183	0.0185377724472	-0.0105389886855	0.799882501649	-0.370347397553	0.585043522894	0.140239514527	-0.00418778232559	0.293171760941	-0.00882908288453	0.00695057813298	0.0884266582183	-0.0237358873032	-0.106276396312	-1.42104636868	0.354183693846	0.526641603736	-0.0278131992539	0.0391749217188	2.40055441051	-0.00237924029635	0.0785169021052	-1.31357125983	1.10004293785	-0.233661345159	0.00552666401847	-0.00016718778103	1.19774131349	-0.286048287152	0.00579817998766	0.000768204732404	0.0673671095849	-1.52030211371	0.0245767107921	-0.173480601645	0.012477741376	-0.27161723021	0.0843238078119	0.729281614702	0.162477640605	-0.0882622809857	0.0336024855392	0.000571658455461	1.93267158369	-1	1