# Predictive Analytics for Quality Assurance Processes in Manufacturing

Approach
===============
Key components of the solution are driven by manufacturing data that is characterized (see Figure 1) by Assembly Line Steps (ALS):

-   Main target is the list of failures collected at a later or final ALS (e.g at 5-th ALS).
-   Subset failures using domain knowledge to extract those likely related to manufacturing processes performed at an earlier ALS (e.g. 3rd ALS), even if the device **passed the existing QA test** at that earlier stage.
-   Build a training dataset by pairing each ALS specific subset of failures with measures collected at its corresponding step and earlier (e.g. ALS 1, 2, and 3).
-   Train/build machine learning (**ML**) models using the above described datasets. These models will predict early (i.e. **at step 3**) failures that would happen in step 5.
-   Build a cascade of models, each covering failures specific to an individual ALS.  

![Solution Concept](https://cloud.githubusercontent.com/assets/16708375/23796630/dbd17132-0593-11e7-8576-ee2262b94c84.png)  

Figure 1. Design of the ML modules in the CIS solution template for Predictive Analytics for Manufacturing. An Original Equipment Manufacturer (OEM), shown in the upper right block, uses two contract manufacturers (CM) shown on the left.  
- [ ] CM blocks indicate assembly lines from contract manufacturers, characterized by a sequence of Assembly Line Steps (ALS) shown by blue modules that ends with a functional test unit (green modules). Each device goes through and is tested at the exit of each individual ALS. Some devices, even after passing individual ALS tests, still turn out to be failures (see green modules and associated results table, with true labels in black) at the final functional test, or after they are sent to OEM. 
- [ ] OEM and CMs form a **hierarchy of pipelines**, where ALS are separated in time and/or space. 
- [ ] OEM post sale/delivery failures are devices that have passed the integrated test system (brown module) already in place. Yet they still happen months or years after delivery and incur repair/warranty costs (see brown module and associated results table, with true labels in black).
- [ ] By pairing such failures with test measurements from their likely ALS steps, we can use Machine Learning to build models (yellow) that will predict failures at early stages (e.g. ALS1 for the model with red contour, new functional tests labels are in red).  

Prediction is done before the failures happen, at an early ALS when correcting or even scrapping the device is much cheaper than dealing with a failure at a later or final stage.

Data Sources:
================
Solution demo uses [SECOM](https://archive.ics.uci.edu/ml/datasets/SECOM) dataset ([UCI ML Data repository](http://archive.ics.uci.edu/ml/datasets.html)), upsampled using an algorithm similar to [SMOTE](http://jair.org/media/953/live-953-2037-jair.pdf).  
  
Each ALS measurements set was chosen by random sampling original SECOM feature columns. Data simulator sends a data point for each ALS every 5 seconds. Each ALS dataset also contains all features from previous ALS. Predictions are made for individual devices as they travel down a production line passing through 5 different ALS’s (waypoints). 


ML implementation:
=====================
The predictive analytics part of the solution is implemented using [Azure ML studio](https://studio.azureml.net/). The [training experiments](https://gallery.cortanaintelligence.com/Experiment/Training-Model-ALS01-1) use a two class linear [SVM](https://msdn.microsoft.com/en-us/library/azure/dn905835.aspx) classifier, fine tuned by optimizing the AUC (area under the ROC curve) metric using the Azure ML Studio ["Tune Model Hyperparameters"](https://msdn.microsoft.com/en-us/library/azure/dn905810.aspx) module.


![Azure ML studio Training Experiment for ALS/waypoint 0](https://cloud.githubusercontent.com/assets/16708375/20055465/198cec34-a4d9-11e6-95b0-93bee2269005.png)


Figure 2
Azure ML studio Training Experiment for ALS/waypoint 0


The corresponding predictive experiments for all ALS are [published](http://gallery.cortanaintelligence.com/Experiment/Prediction-Model-Mk-IIII-w-p-0-Predictive-Exp-1) with this solution.  
  
When adapting this solution to other datasets, the ML section can be expanded as needed:  
	- More preprocessing [Azure ML studio](https://studio.azureml.net/) modules for [feature engineering](https://msdn.microsoft.com/en-us/library/azure/dn905834.aspx) and [selection](https://msdn.microsoft.com/en-us/library/azure/dn905912.aspx) can be added.  
	- Different machine learning [algorithms](https://msdn.microsoft.com/en-us/library/azure/dn905812.aspx) for Classification or Regression can be added and compared.  
	- Best classifier performance can be further tweaked after training by [choosing the appropriate ROC operating point](https://blogs.msdn.microsoft.com/andreasderuiter/2015/02/09/using-roc-plots-and-the-auc-measure-in-azure-ml/) that gives the desired sensitivity-specificity trade-off driven by the cost of false positives and false negative in each uses case.  
	
See also the related resources at the bottom of this page for more details on how to approach the data science and machine learning component of this solution for your own data.
  
Solution PBI dashboard (visualization):
==========================================
![Solution Dashboard](https://cloud.githubusercontent.com/assets/16708375/19904139/5cacc6e8-a069-11e6-830c-d42c3a94e678.png)


Figure 3. Real-time Dashboard


In figure above we show the upper part of the solution dashboard, visualizing results from the first 3 ALS/waypoints (ALS 0, ALS 1, and ALS 2) of a simulated assembly line. We implemented 5 ALS in total in this solution, but the user can add as many ALS as needed, and show their results in the dashboard accordingly. For each ALS, we show a global device counter (the gauge on the left), and the number (upper) and ids (lower) of the failed (middle column) and passed (i.e. not-failed, in right column) devices in the last 30 seconds. Each report is updated dynamically every 5 second as new ALS measurements come in. 

Each tile is defined using quasi natural language (NLP) query (e.g. **How many device_id with label=1 in waypoint 0 in the last 30 seconds?**), typed in the **Ask a question about your data** text box in PBI dashboard. 

![PBI](https://cloud.githubusercontent.com/assets/9042064/20732304/23c67ee2-b65c-11e6-969a-0a8f81bf5963.PNG)


Figure 4. Insights Dashboard


In figure above we show an example dashboard that is built on top of the SQL Data Warehouse that collects all the information about the devices over time as well as the predictions that are made at each assembly line step.

Related Resources
==========================================
A playbook for approaching predictive maintenance problems, which can be considered to include use cases such as that discussed within this solution, is published [here](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance) and includes detail on how to [prepare data sources](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#data-preparation), create features, and [set up a machine learning model](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance#modeling-techniques).

There is also a modelling guide published in the format of a Jupyter notebook written in R [here](https://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Modelling-Guide-1) and some details on how to think about evaluating predictive maintenance models [here](https://blogs.technet.microsoft.com/machinelearning/2016/04/19/evaluating-failure-prediction-models-for-predictive-maintenance/).
