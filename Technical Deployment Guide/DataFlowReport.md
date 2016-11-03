# Cortana Intelligence Suite Solution Template
# Predictive Analytics for Quality Assurance Process in Manufacturing


## Solution Post-Deployment monitoring
======================================

Solution status can be monitored using the solution components in [Azure Management Portal](https://ms.portal.azure.com). 

![Solution Components in Azure portal](https://cloud.githubusercontent.com/assets/16708375/19970093/e0a6a49a-a1d2-11e6-8f29-d5974d92480c.png)

 <sub>
 Solution Components in Azure Management Portal
 </sup>


We will focus on the solution end (Event Hub as the beginning and Blob Storage sinks at the end), and present 2 test scenarios explaining the solution data flow, and how the ASA job uses the buffering capabilities to process messages received in the past.

###Test Scenario 1 (process **only current messages**)

We start with the ASA job stopped and empty containers (all blobs deleted) in solution blob storage account.

![Empty Containers](https://cloud.githubusercontent.com/assets/16708375/19970112/f324153a-a1d2-11e6-8437-c79a12ceaa42.png)

 <sub>
 Empty Containers (all blobs deleted) in solution blob storage account
 </sup>
 
 
Next actions were performed:
-   11:59:34 GMT+0000: started Simulated Data Generator
-   12:11:02 GMT+0000: started ASA
-   12:19:46 GMT+0000: stopped Simulated Data Generator

The local/on-prem solution data generator:

![local/on-prem solution data generator](https://cloud.githubusercontent.com/assets/16708375/19970423/d5464b2c-a1d3-11e6-96cc-c7e63a7d022d.png)


was used to send to event hub a data point for each ALS every 5 seconds:

![local/on-prem solution data generator running](https://cloud.githubusercontent.com/assets/16708375/19970574/469e1282-a1d4-11e6-9b12-ef5327c01335.png)


The ASA job start time was set to **"Now"**:

![ASA job start time](https://cloud.githubusercontent.com/assets/16708375/19970440/e0162b4e-a1d3-11e6-89d0-76dc1dc68e25.png)


Using the solution Event Hub Metrics blade on [Azure Management Portal](https://ms.portal.azure.com), we can monitor metrics like (Archive backlog messages, Archive message throughput, Archive messages, Failed Requests, Incoming Messages, Incoming Requests, Incoming bytes per second, Internal Server Errors, Other Errors, Outgoing Messages, Outgoing bytes per second, Server Busy Errors, Successful Requests):

![Data flow](https://cloud.githubusercontent.com/assets/16708375/19895489/e31c0f16-a048-11e6-830d-b579a1e1168d.png)

Data flow visualization


![Data flow](https://cloud.githubusercontent.com/assets/16708375/19895542/1160112e-a049-11e6-84e2-063a508969ae.png)

Data flow visualization (Incoming Bytes/second detail)

Monitoring data flow.
- [ ] Simulated Data Generator was started about 10 minutes before the ASA job 
- [ ] When the ASA starts (start time was set to "Now"), it looks for messages in Event hub buffer that are received after it strats, and ignores previous messages. 
- [ ] After Simulated Data Generator is stopped, no more data enters or leaves the event hub.
- [ ] Content of blob sinks (shown below) shows the device messages moving along the assembly line starting at and after the ASA job starting time. As the ASA starts, we see an initial spike in outgoing messages, corresponding to data already in the Event Hub buffer that is being processed almost instantaneously (e.g. *device_119*). After the Hub buffer is emptied, the devices are processed every 5 seconds, as they enter the asembly line (e.g. *device_239*).

Blob storage sinks content (.csv files) has this header:

   recNo, evttime,ALS_ID,device_id,label
  
  
  __ALS0 sink (125 records)__:
  * 001, 2016-11-01T**12:11:00.0281424Z**,1,**device_119**,1
  * 002, 2016-11-01T12:11:03.8574693Z,1,device_120,-1
  * ...
  * 121, 2016-11-01T**12:19:09.9356207Z**,1,**device_239**,1
  * 122, 2016-11-01T12:19:13.1135193Z,1,device_240,1
  * 123, 2016-11-01T12:19:18.0717059Z,1,device_241,-1
  * 124, 2016-11-01T12:19:23.0858207Z,1,device_242,1
  * 125, 2016-11-01T12:19:28.0744674Z,1,device_243,-1
  
  
  __ALS1 sink (125 records)__:
  * 001, 2016-11-01T12:11:00.7468767Z,2,device_118,-1
  * 002, 2016-11-01T**12:11:03.8418308Z**,2,**device_119**,1
  * 003, 2016-11-01T12:11:03.3730783Z,2,device_120,-1
  * ...
  * 122, 2016-11-01T**12:19:13.1291452Z**,2,**device_239**,1
  * 123, 2016-11-01T12:19:18.0858606Z,2,device_240,1
  * 124, 2016-11-01T12:19:23.0858207Z,2,device_241,-1
  * 125, 2016-11-01T12:19:28.0744674Z,2,device_242,1
  
  
  __ALS2 sink (125 records)__:
  * 001, 2016-11-01T12:11:00.0437670Z,3,device_117,-1
  * 002, 2016-11-01T12:11:03.8574693Z,3,device_118,-1
  * 003, 2016-11-01T**12:11:03.3887035Z**,3,**device_119**,1
  * ...
  * 122, 2016-11-01T12:19:13.1135193Z,3,device_238,-1
  * 123, 2016-11-01T**12:19:18.0858606Z**,3,**device_239**,1
  * 124, 2016-11-01T12:19:23.0858207Z,3,device_240,1
  * 125, 2016-11-01T12:19:28.0744674Z,3,device_241,-1
  
  
  __ALS3 sink (125 records)__:
  * 001, 2016-11-01T12:11:00.0437670Z,4,device_116,1
  * ...
  * 004, 2016-11-01T**12:10:59.8718949Z**,4,**device_119**,1
  * ...
  * 122, 2016-11-01T12:19:13.1135193Z,4,device_237,-1
  * 123, 2016-11-01T12:19:18.0858606Z,4,device_238,1
  * 124, 2016-11-01T**12:19:23.1014391Z**,4,**device_239**,1
  * 125, 2016-11-01T12:19:28.0744674Z,4,device_240,1
  
  
  __ALS4 sink (125 records)__:
  * 001, 2016-11-01T12:11:00.2312624Z,5,device_115,1
  * ...
  * 005, 2016-11-01T**12:11:00.5510285Z**,5,**device_119**,-1
  * ...
  * 122, 2016-11-01T12:19:13.1291452Z,5,device_236,-1
  * 123, 2016-11-01T12:19:18.1058094Z,5,device_237,-1
  * 124, 2016-11-01T12:19:23.1014391Z,5,device_238,1
  * 125, 2016-11-01T**12:19:28.1049632Z**,5,**device_239**,1
  
###Test Scenario 2 (process **all messages in Event hub buffer, including those received before ASA start**)  

If we repeat the above test scenario (starting again with the ASA job stopped and empty containers):
-   12:58:34 GMT+0000: start Simulated Data Generator
-   13:12:45 GMT+0000: start ASA
-   13:25:55 GMT+0000: stop Simulated Data Generator

but with ASA job start time set to **"Custom", and selecting a moment (12:50:00) before the data generator start**:

![ASA job start time set to custom](https://cloud.githubusercontent.com/assets/16708375/19971709/bbf8f4ea-a1d7-11e6-99d2-5e5f855e840c.png)

Solution Event Hub Metrics show a much larger processing spike in the output messages:

![Data flow](https://cloud.githubusercontent.com/assets/16708375/19974173/79172086-a1df-11e6-927c-a8fe701e82a4.png)

Monitoring data flow.
- [ ] Simulated Data Generator was started about 20 minutes before the ASA job 
- [ ] When the ASA begins, it looks at all messages in the event hub buffer, since its **start time** property is set (12:50:00) 8 mintes before the data generator started. 
- [ ] After Simulated Data Generator is stopped, no more data enters or leaves the event hub.
- [ ] Content of blob sinks (shown below) capture the initial snapshot of the devices moving along the assembly line. As the ASA starts, we see an initial spike in outgoing messages, corresponding to data being processed almost instantaneously from the Event Hub buffer (e.g. *device_0*). After the Hub buffer is emptied, the devices are processed every 5 seconds, as they enter the asembly line (e.g. *device_330*). 

In contrast to scenario one, we now have predictions for all devices (0 to 331 for ALS0, 0 to 3328 for ALS4). 

Blob storage sinks content (.csv files) has this header:

   recNo, evttime,ALS_ID,device_id,label
  
  
  __ALS0 sink (333 records)__:
  * 001, 2016-11-03T**13:12:54.3469891Z**,1,**device_0**,1
  * 002, 2016-11-03T13:12:55.5813447Z,1,device_1,1
  * 003, 2016-11-03T13:12:49.1446001Z,1,device_2,-1
  * ... 
  * 331, 2016-11-03T**13:25:41.7416238Z**,1,**device_330**,1
  * 332, 2016-11-03T13:25:46.7416299Z,1,device_331,1  
  * 333, 2016-11-03T13:25:51.8197512Z,1,device_332,1
  
  
  
  __ALS1 sink (332 records)__:
  * 001, 2016-11-03T**13:12:55.6017034Z**,2,**device_0**,-1
  * 002, 2016-11-03T13:12:49.4110249Z,2,device_1,1
  * 003, 2016-11-03T13:12:52.6927579Z,2,device_2,-1
  * ... 
  * 331, 2016-11-03T13:25:41.7783959Z,2,device_329,1
  * 332, 2016-11-03T**13:25:46.7736562Z**,2,**device_330**,1
  * 333, 2016-11-03T13:25:51.8197512Z,2,device_331,1
  
  
  
  __ALS2 sink (331 records)__:
  * 001, 2016-11-03T**13:12:49**.4110249Z,3,**device_0**,-1
  * 002, 2016-11-03T13:12:52.6927579Z,3,device_1,1
  * 003, 2016-11-03T13:12:54.3626147Z,3,device_2,-1
  * ... 
  * 331, 2016-11-03T13:25:41.7416238Z,3,device_328,-1
  * 332, 2016-11-03T13:25:46.7736562Z,3,device_329,1
  * 333, 2016-11-03T**13:25:51.8367132Z**,3,**device_330**,1
  
  
  
  __ALS3 sink (330 records)__:
  * 001, 2016-11-03T**13:12:52.6927579Z**,4,**device_0**,-1
  * 002, 2016-11-03T13:12:54.4127584Z,4,device_1,1
  * 003, 2016-11-03T13:12:55.6750906Z,4,device_2,-1
  * ... 
  * 331, 2016-11-03T13:25:41.7783959Z,4,device_327,-1
  * 332, 2016-11-03T13:25:46.7572610Z,4,device_328,-1
  * 333, 2016-11-03T13:25:51.8367132Z,4,device_329,1
  
  
  
  
  __ALS4 sink (329 records)__:
  * 001, 2016-11-03T**13:12:54.3836248Z**,5,**device_0**,-1
  * 002, 2016-11-03T13:12:55.6438347Z,5,device_1,1
  * 003, 2016-11-03T13:12:49.6127917Z,5,device_2,-1
  * ... 
  * 331, 2016-11-03T13:25:41.7783959Z,5,device_326,-1
  * 332, 2016-11-03T13:25:46.7572610Z,5,device_327,-1
  * 333, 2016-11-03T13:25:51.8197512Z,5,device_328,-1
  
  
  
   
  