# Predictive Analytics for Quality Assurance Processes in Manufacturing

1. Approach
===============
Manufacturing data is characterized (see Figure 1) by Assembly Line Steps (ALS):

-   Main target is the list of failures collected at a later or final ALS (e.g at 5-th ALS).
-   These can be subset using SME to extract the failures likely related to manufacturing processes performed at an earlier ALS (e.g. 3rd ALS), even if the device **passed the existing QA test** at that earlier stage.
-   Build a training dataset by pairing each ALS specific subset of failures with measures collected at its corresponding step and earlier (e.g. ALS 1, 2, and 3).
-   Use the above described datasets to train/build machine learning (**ML**) models that will predict early (i.e. **at step 3**) failures that would happen in step 5.

![Solution Concept](https://cloud.githubusercontent.com/assets/16708375/19810774/6eeae1e4-9d37-11e6-8fed-13ba99f4846b.png)

Figure 1. Design of the ML modules in the CIS solution template for Predictive Analytics for Manufacturing.
- [ ] CM blocks indicate assembly lines from contract manufacturers. Each device goes through and is tested at the exit of each individual Assembly Line Step (ALS). Some devices, even after passing individual ALS tests, still turn out to be failures (see green modules and associated results table, with true labels in black) at the final functional test, or after they are sent to OEM. 
- [ ] OEM and CMs form a Hierarchy of pipelines, where ALS are separated in time and/or space. 
- [ ] Post sale/delivery failures passed the integrated test system already in place. Yet they still happen months or years after delivery and incur repair/warranty costs (see brown module and associated results table, with true labels in black).
- [ ] By pairing such failures with test measurements from their likely ALS steps, we can use Machine Learning to build models (yellow) that will predict failures at early stages (e.g. ALS1 for the model with red contour).



Prediction is done before the failures happen, at an early ALS when correcting or even scrapping the device is much cheaper than dealing with a failure at a later or final stage.

2. Examples:
============

**Assembly line scenario**: For a computer mother board assembly line (Figure 2), at ALS 3 (ALS_03) the solder paste is applied on the empty board, and then a regular test is applied to check if the paste droplets are not too big or not too small and if they are at the center or each pad. Boards than pass the **existing, already in place** regular test move to next ALS-s, eventually the electronic circuits are placed on the board and then connected to the board pads by melting the solder droplets in an oven, and in the end of ALS 5 (ALS_05), a final electronic functional check is done before the board is shipped to client OEM PC manufacturer. Some of the failures discovered at ALS_05 are due to defective solder paste application performed in ALS_03, even if the board **passed the regular ALS_03 test**. We can use **ML** to predict **solder related** failures right after the solder paste is applied in ALS_03, but **before the expensive electronic components** like microprocessors and other integrated circuits are added to the board in ALS_05. Rewashing a board with functional defect yielding solder paste application that **passed the regular QA test** at ALS_01 may cost cents, while scrapping a defective board at ALS_05 after the processor and all other components are applied may cost hundreds of dollars. 

![Jabil Showcase Assembly Line](https://cloud.githubusercontent.com/assets/16708375/19811557/90a87280-9d3a-11e6-8f2b-f573c3b02eca.png)

<sub>
Figure 2
[Jabil](http://www.jabil.com/blog/microsoft-and-jabil-collaborate-to-create-predictive-analytics-quality-assurance-platform/) Showcase
</sup>

**OEMs who use contract manufacturers (Apple-Foxconn, MSFT-Jabil), and want to minimize post sale service and warranty costs**: Use **ML** to build models that use test/shop floor data (that belongs to OEM and or CMs) to predict, before shipping, field return and repairs that may happen months or years after device is shipped. This enables predicting **future failures** while the device is still in the **early manufacturing line stages** or is already assembled but is **not yet shipped**, so that fixing or even discarding it may be cheaper than going through recall and warranty costs.


3. Data Sources:
================
Solution demo uses [SECOM](https://archive.ics.uci.edu/ml/datasets/SECOM) dataset ([UCI ML Data repository](http://archive.ics.uci.edu/ml/datasets.html)), upsampled using an algorithm similar to [SMOTE](http://jair.org/media/953/live-953-2037-jair.pdf).  
  
Each ALS measurements set was chosen by random sampling original SECOM feature columns. Data simulator sends a data point for each ALS every 5 seconds. Each ALS dataset also contains all features from previous ALS. Predictions are made for individual devices as they travel down a production line passing through 5 different ALS’s (waypoints). 


4. ML implementation:
=====================
The predictive analytics part of the solution is implemented using [Azure ML studio](https://studio.azureml.net/). The training experiments use a two class linear [SVM](https://msdn.microsoft.com/en-us/library/azure/dn905835.aspx) classifier, fine tuned by optimizing the AUC (area under the ROC curve) metric using the Azure ML Studio ["Tune Model Hyperparameters"](https://msdn.microsoft.com/en-us/library/azure/dn905810.aspx) module.


![Azure ML studio Training Experiment for ALS/waypoint 0](https://cloud.githubusercontent.com/assets/16708375/20055465/198cec34-a4d9-11e6-95b0-93bee2269005.png)

<sub>
Figure 3
Azure ML studio Training Experiment for ALS/waypoint 0
</sup>

The corresponding predictive experiments for all ALS are [published](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-0-Predictive-Exp-1) with this solution.  
  
When adapting this solution to other datasets, the ML section can be expanded as needed:  
	- More preprocessing [Azure ML studio](https://studio.azureml.net/) modules for [feature engineering](https://msdn.microsoft.com/en-us/library/azure/dn905834.aspx) and [selection](https://msdn.microsoft.com/en-us/library/azure/dn905912.aspx) can be added.  
	- Different machine learning [algorithms](https://msdn.microsoft.com/en-us/library/azure/dn905812.aspx) for Classification or Regression can be added and compared.  
	- Furthermore, the best classifier performance can be further tweaked after training by [choosing the appropriate ROC operating point](https://blogs.msdn.microsoft.com/andreasderuiter/2015/02/09/using-roc-plots-and-the-auc-measure-in-azure-ml/) that gives the desired sensitivity-specificity trade-off driven by the cost of false positives and false negative in each uses case.  
	
See also the related resources at the bottom of this page for more details on how to approach the data science and machine learning component of this solution for your own data.
  
5. Solution PBI dashboard (visualization):
==========================================
![Solution Dashboard](https://cloud.githubusercontent.com/assets/16708375/19904139/5cacc6e8-a069-11e6-830c-d42c3a94e678.png)

Figure 4. Real-time Dashboard

We show here the upper part of the solution dashboard, visualizing results from the first 3 waypoints (ALS 0, ALS 1, and ALS 2) of a simulated assembly line. We implemented 5 ALS in total in this solution, but the user can add as many ALS as needed, and show their results in the dashboard accordingly. For each ALS, we show a global device counter (the gauge on the left), and the number (upper) and ids (lower) of the failed (middle column) and passed (right column) devices in the last 30 seconds. Each report is updated dinamically every 5 second as new ALS measurements come in. 

Each tile is defined using quasi natural language (NLP) query (e.g. **How many device_id with label=1 in waypoint 0 in the last 30 seconds?**), typed in the **Ask a question about your data** text box in PBI dashboard. 

![PBI](https://cloud.githubusercontent.com/assets/9042064/20732304/23c67ee2-b65c-11e6-969a-0a8f81bf5963.PNG)

Figure 5. Insights Dashboard

We show here an example dashboard that is built on top of the SQL Data Warehouse that collects all the information about the devices over time as well as the predictions that are made at each assembly line step.

6. Related Resources
==========================================
A "playbook" for approaching predictive maintenance problems, which can be considered to include use cases such as that discussed within this solution, is published [here](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance) and includes detail on how to [prepare data sources](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#data-preparation), create features, and [set up a machine learning model](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#modeling-techniques).

There is also a modelling guide published in the format of a Jupyter notebook written in R [here](https://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Modelling-Guide-1) and some details on how to think about evaluating predictive maintenance models [here](https://blogs.technet.microsoft.com/machinelearning/2016/04/19/evaluating-failure-prediction-models-for-predictive-maintenance/).
