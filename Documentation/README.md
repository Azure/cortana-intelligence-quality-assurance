# Cortana Intelligence Suite Solution Template
# Predictive Analytics for Quality Assurance Process in Manufacturing


## Solution Post-Deployment monitoring
===============

Solution status can be monitored using [Azure Management Portal](https://ms.portal.azure.com). We present here an test scenario that explaining the solution data flow:

-   11:59:34 GMT+0000: started Simulated Data Generator
-   12:11:02 GMT+0000: started ASA
-   12:19:46 GMT+0000: stopped Simulated Data Generator

Using the solution Event Hub Metrics blade on [Azure Management Portal](https://ms.portal.azure.com), we can monitor metrics like Archive backlog messages, Archive message throughput, Archive messages, Failed Requests, Incoming Messages, Incoming Requests, Incoming bytes per second, Internal Server Errors, Other Errors, Outgoing Messages, Outgoing bytes per second, Server Busy Errors, Successful Requests):

![Data flow](https://cloud.githubusercontent.com/assets/16708375/19895489/e31c0f16-a048-11e6-830d-b579a1e1168d.png)
Data flow visualization


![Data flow](https://cloud.githubusercontent.com/assets/16708375/19895542/1160112e-a049-11e6-84e2-063a508969ae.png)
Data flow visualization (Incoming Bytes/second detail)

Monitoring data flow.
- [ ] Simulated Data Generator was started about 10 minutes before the ASA job 
- [ ] When the ASA starts, it takes a snapshot of the device sequence, as it goes through different ALS. 
- [ ] After Simulated Data Generator is stopped, no more data enters or leaves the event hub.
- [ ] Content of blob sinks (shown below) capture the initial snapshot of the devices moving along the assembly line. As the ASA starts, we see an intitial spike corresponding to data being processed almost instantaneously from the Event Hub buffer (e.g. *device_119*). After the Hub buffer is emptied, the devices are processed every 5 seconds, as they enter the asembly line (e.g. *device_239*).

recNo, evttime,waypoint,device_id,label

**ALS0 sink (125 records)**:
001, 2016-11-01T*12:11:00.0281424Z*,1,*device_119*,1
002, 2016-11-01T12:11:03.8574693Z,1,device_120,-1
...
121, 2016-11-01T*12:19:09.9356207Z*,1,*device_239*,1
122, 2016-11-01T12:19:13.1135193Z,1,device_240,1
123, 2016-11-01T12:19:18.0717059Z,1,device_241,-1
124, 2016-11-01T12:19:23.0858207Z,1,device_242,1
125, 2016-11-01T12:19:28.0744674Z,1,device_243,-1

**ALS1 sink (125 records)**:
001, 2016-11-01T12:11:00.7468767Z,2,device_118,-1
002, 2016-11-01T*12:11:03.8418308Z*,2,*device_119*,1
003, 2016-11-01T12:11:03.3730783Z,2,device_120,-1
...
122, 2016-11-01T*12:19:13.1291452Z*,2,*device_239*,1
123, 2016-11-01T12:19:18.0858606Z,2,device_240,1
124, 2016-11-01T12:19:23.0858207Z,2,device_241,-1
125, 2016-11-01T12:19:28.0744674Z,2,device_242,1

**ALS2 sink (125 records)**:
001, 2016-11-01T12:11:00.0437670Z,3,device_117,-1
002, 2016-11-01T12:11:03.8574693Z,3,device_118,-1
003, 2016-11-01T*12:11:03.3887035Z*,3,*device_119*,1
...
122, 2016-11-01T12:19:13.1135193Z,3,device_238,-1
123, 2016-11-01T*12:19:18.0858606Z*,3,*device_239*,1
124, 2016-11-01T12:19:23.0858207Z,3,device_240,1
125, 2016-11-01T12:19:28.0744674Z,3,device_241,-1

**ALS3 sink (125 records)**:
001, 2016-11-01T12:11:00.0437670Z,4,device_116,1
...
004, 2016-11-01T*12:10:59.8718949Z*,4,*device_119*,1
...
122, 2016-11-01T12:19:13.1135193Z,4,device_237,-1
123, 2016-11-01T12:19:18.0858606Z,4,device_238,1
124, 2016-11-01T*12:19:23.1014391Z*,4,*device_239*,1
125, 2016-11-01T12:19:28.0744674Z,4,device_240,1

**ALS4 sink (125 records)**:
001, 2016-11-01T12:11:00.2312624Z,5,device_115,1
...
005, 2016-11-01T*12:11:00.5510285Z*,5,*device_119*,-1
...
122, 2016-11-01T12:19:13.1291452Z,5,device_236,-1
123, 2016-11-01T12:19:18.1058094Z,5,device_237,-1
124, 2016-11-01T12:19:23.1014391Z,5,device_238,1
125, 2016-11-01T*12:19:28.1049632Z*,5,*device_239*,1