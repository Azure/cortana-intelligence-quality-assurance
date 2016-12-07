# Quality Assurance for Manufacturing - A Cortana Intelligence Solution How-to Guide

In digital manufacturing, quality assurance across the assembly line is imperative. Identifying errors, slowdowns and potential failures before they occur rather than after they are detected can help companies be more proactive and improve productivity. The goal of the solution guide presented here is to predict failures in manufacturing pipelines (assembly lines).  This is done by leveraging test systems already in place and failure data, specifically looking at returns and functional failures at the end of assembly line. By combining these with domain knowledge and root cause analysis within a modular design that encapsulates main processing steps, we provide a generic advanced analytics solution that uses machine learning to predict failures before they happen. Early prediction of future failures allows for less expensive repairs or even discarding, which are usually more cost efficient than going through recall and warranty cost.

## Solution Dashboard
The snapshot below shows an example PowerBI dashboard that gives insights into the manufacturing assembly lines that are monitored.
![Insights](https://cloud.githubusercontent.com/assets/9042064/20733041/891bb5f2-b65f-11e6-8362-068b9ba92698.PNG)

## Solution Architecture
![Solution Diagram Picture](https://cloud.githubusercontent.com/assets/16708375/20932195/acb87330-bbcb-11e6-8a89-27d8b6e17bdf.png)

## Example Use Case
**Assembly line scenario**: For a computer mother board assembly line (Figure 2), at ALS 3 (ALS_03) the solder paste is applied on the empty board, and then a regular test is applied to check if the paste droplets are not too big or not too small and if they are at the center or each pad. Boards than pass the **existing, already in place** regular test move to next ALS-s, eventually the electronic circuits are placed on the board and then connected to the board pads by melting the solder droplets in an oven, and in the end of ALS 5 (ALS_05), a final electronic functional check is done before the board is shipped to client OEM PC manufacturer. Some of the failures discovered at ALS_05 are due to defective solder paste application performed in ALS_03, even if the board **passed the regular ALS_03 test**. We can use **ML** to predict **solder related** failures right after the solder paste is applied in ALS_03, but **before the expensive electronic components** like microprocessors and other integrated circuits are added to the board in ALS_05. Rewashing a board with functional defect yielding solder paste application that **passed the regular QA test** at ALS_01 may cost cents, while scrapping a defective board at ALS_05 after the processor and all other components are applied may cost hundreds of dollars. 

![Jabil Showcase Assembly Line](https://cloud.githubusercontent.com/assets/16708375/19811557/90a87280-9d3a-11e6-8f2b-f573c3b02eca.png)

 <sub>
Figure 2 [Jabil Showcase](http://www.jabil.com/blog/microsoft-and-jabil-collaborate-to-create-predictive-analytics-quality-assurance-platform/) 
</sub>

**OEMs who use contract manufacturers (Apple-Foxconn, MSFT-Jabil), and want to minimize post sale service and warranty costs**: Use **ML** to build models that use test/shop floor data (that belongs to OEM and or CMs) to predict, before shipping, field return and repairs that may happen months or years after device is shipped. This enables predicting **future failures** while the device is still in the **early manufacturing line stages** or is already assembled but is **not yet shipped**, so that fixing or even discarding it may be cheaper than going through recall and warranty costs.

## Getting Started

This solution package contains materials to help both technical and business audiences understand our Quality Assurance solution for manufacturing built on the [Cortana Intelligence Suite](https://www.microsoft.com/en-us/server-cloud/cortana-intelligence-suite/Overview.aspx).

## Business Audiences

In this repository you will find a folder labeled [*Solution Overview for Business Audiences*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Solution%20Overview%20for%20Business%20Audiences) which contains a  presentation covering this solution and benefits of using the Cortana Intelligence Suite

For more information on how to tailor Cortana Intelligence to your needs [connect with one of our partners](http://aka.ms/CISFindPartner).

## Technical Audiences

See the [*Technical Deployment Guide*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Technical%20Deployment%20Guide) folder for a full set of instructions on how to put together and deploy a Quality Assurance solution using the Cortana Intelligence Suite. For technical problems or questions about deploying this solution, please post in the issues tab of the repository.

## Related Resources
A playbook for approaching predictive maintenance problems, which can be considered to include use cases such as that discussed within this solution, is published [here](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance) and includes detail on how to [prepare data sources](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#data-preparation), create features, and [set up a machine learning model](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#modeling-techniques).
There is also a modelling guide published in the format of a Jupyter notebook written in R [here](https://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Modelling-Guide-1) and some details on how to think about evaluating predictive maintenance models [here](https://blogs.technet.microsoft.com/machinelearning/2016/04/19/evaluating-failure-prediction-models-for-predictive-maintenance/).
