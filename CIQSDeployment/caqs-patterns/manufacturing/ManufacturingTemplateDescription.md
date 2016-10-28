***Engaging Introduction On Manufacturing***

This Manufacturing pattern monitors units as they travel down a production line. It is an end-to-end solution that includes data ingestion, data storage,
data processing and advanced analytics — all essential for building a predictive machine Learning solution.  

Data source of this pattern is comprised of or derived from publicly available data from the ***WHERE***  data repository using the ***DESCRIPTION***  Data Set.

## Ingredients
The solution pattern uses several Azure services:
* Azure Web jobs
* Event Hub
* Stream Analytics
* Azure Storage
* Azure ML
* Power BI

## Estimated Provisioning Time: 30 Minutes
	

### How the building blocks work
1. The Manufacturing Production Line simulation data is streamed by newly deployed web job ManfuacturingGenerator.
2. This synthetic data feeds into the Azure Event Hub service as data points/events, that will be consumed in the rest of the solution flow
3. There are 2 Stream Analytics Jobs used in this pattern to provide near real-time analytics on the input stream from the Event Hub. Both jobs filter through the input data and pass the data points along to a Machine Learning endpoint sending the results to a Power BI Dashboard.
4. Finally, Power BI is used for results visualization.
		
***FIX IMAGE TO SOME DASHBOARD VIEW***
<img src="{PatternAssetBaseUrl}/manufacturing.png" width="100%">
	
		
For questions on this pattern, get in touch with us at <CAQSDISC@microsoft.com>.
