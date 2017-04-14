# Quality Assurance - A Cortana Intelligence Solution How-to Guide

For example, quality assurance is imperative in digital manufacturing - ensuring quality throughout the assembly line production process. Identifying errors, slowdowns and potential failures before they occur rather than after they are detected can help companies be more proactive and improve productivity. The goal of the solution guide presented here is to predict failures in manufacturing pipelines (assembly lines).  This is done by leveraging test systems already in place and failure data, specifically looking at returns and functional failures at the end of assembly line. By combining these with domain knowledge and root cause analysis within a modular design that encapsulates main processing steps, we provide a generic advanced analytics solution that uses machine learning to predict failures before they happen. Early prediction of future failures allows for less expensive repairs or even discarding, which are usually more cost efficient than going through recall and warranty cost.

## Solution Dashboard
The snapshot below shows an example PowerBI dashboard that gives insights into the manufacturing assembly lines that are monitored.
![Insights](https://cloud.githubusercontent.com/assets/9042064/20733041/891bb5f2-b65f-11e6-8362-068b9ba92698.PNG)

## Solution Architecture
![Solution Diagram Picture](https://cloud.githubusercontent.com/assets/16708375/20932195/acb87330-bbcb-11e6-8a89-27d8b6e17bdf.png)

## Example Use Case
As an example, an **assembly line scenario** for a computer mother board manufacturer is given in Figure 2 characterized by Assembly Line Steps (ALS). In this scenerio, at Assembly Line Step 3 (ALS3), the solder paste is applied on the empty board, and then a test is performed to check that the paste droplets are correct size and that they are at the center or each pad. Boards that pass this already in place test, move to next assembly line step and eventually the electronic circuits are placed on the board and then connected to the board pads by melting the solder droplets in an oven. At the end of ALS5, a final electronic functional check is done before the board is shipped to client OEM PC manufacturer. Some of the failures discovered at ALS5 are due to defective solder paste application performed at ALS3, even if the board passed the regular ALS3 test. We can use **Machine Learning (ML)** to predict solder related failures right after the solder paste is applied in ALS3, but before the expensive electronic components like microprocessors and other integrated circuits are added to the board at ALS5. Rewashing a board with a functional defect that passed the regular Quality Asssurance (QA) test at ALS1 may cost cents, while scrapping a defective board at ALS5 after the processor and all other components are applied may cost hundreds of dollars. 

![Jabil Showcase Assembly Line](https://cloud.githubusercontent.com/assets/16708375/19811557/90a87280-9d3a-11e6-8f2b-f573c3b02eca.png)


[Jabil Showcase](https://www.jabil.com/blog/microsoft-and-jabil-collaborate-to-create-predictive-analytics-quality-assurance-platform)


OEMs which use contract manufacturers (such as Microsoft and Jabil) can minimize post sale service and warranty costs using **ML** to build models that use shop floor data to predict, before shipping, returns and repairs that may happen months or years after device is shipped. This enables predicting **future failures** while the device is still in the **early manufacturing line stages** or is already assembled but is **not yet shipped**, so that fixing or even discarding it may be cheaper than going through recall and warranty costs.

This solution uses [SECOM](https://archive.ics.uci.edu/ml/datasets/SECOM) dataset from ([UCI ML Data repository](http://archive.ics.uci.edu/ml/datasets.html)) to generalize the example scenario described above where the main goal is to predict failures on an assembly line with 5 steps. As devices travel through the assembly line, certain measurements are collected at each step. These measurements are then used to predict possible failures at the end of the assembly line using a predictive model developed for each step. For technical details, please refer to the [*Solution Description*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Manual%20Deployment%20Guide/SolutionDescription.md) under the [*Manual Deployment Guide*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing).

## Getting Started

This solution package contains materials to help both technical and business audiences understand our Quality Assurance solution for manufacturing built on the [Cortana Intelligence Suite](https://www.microsoft.com/en-us/server-cloud/cortana-intelligence-suite/Overview.aspx).

## Business Audiences

In this repository you will find a folder labeled [*Solution Overview for Business Audiences*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/blob/master/Solution%20Overview%20for%20Business%20Audiences) which contains a  presentation covering this solution and benefits of using the Cortana Intelligence Suite

For more information on how to tailor Cortana Intelligence to your needs [connect with one of our partners](http://aka.ms/CISFindPartner).

## Technical Audiences

See the [*Manual Deployment Guide*](https://github.com/Azure/cortana-intelligence-quality-assurance-manufacturing/tree/master/Manual%20Deployment%20Guide) folder for a full set of instructions on how to put together and deploy a Quality Assurance solution using the Cortana Intelligence Suite. For technical problems or questions about deploying this solution, please post in the issues tab of the repository.

## Related Resources
A playbook for approaching predictive maintenance problems, which can be considered to include use cases such as that discussed within this solution, is published [here](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance) and includes detail on how to [prepare data sources](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#data-preparation), create features, and [set up a machine learning model](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#modeling-techniques).
There is also a modelling guide published in the form of a Jupyter notebook written in R [here](https://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Modelling-Guide-1) and some details on how to think about evaluating predictive maintenance models [here](https://blogs.technet.microsoft.com/machinelearning/2016/04/19/evaluating-failure-prediction-models-for-predictive-maintenance/).
