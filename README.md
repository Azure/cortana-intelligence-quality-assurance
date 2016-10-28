# cortana-intelligence-quality-assurance-manufacturing

1. Introduction
===============

The purpose of the project is a manufacturing solution template for predicting failures in manufacturing pipelines (assembly lines). The idea is to leverage test systems already in place and failures data, specifically looking at returns and functional failures at the end of assembly line. By combining these with domain knowledge and root cause analysis within a modular design that encapsulates main processing steps, we provide a generic advanced analytics solution that uses machine learning to predict failures before they happen. Early prediction of future failures allows for less expensive repairs or even discarding, which are usually more cost efficient than going through recall and warranty cost.

The generic out of the box solution is implemented in the cloud, using the flexible on-line Microsoft Azure platform that decouples infrastructure components (data ingestion, storage, data movement, visualization) from analytics engine that supports modern DS languages like R and Python. The solution modeling component can thus be retrained as needed and be implemented using high performance Azure Machine Learning algorithms, or open source (R/Python) libraries, or from a third-party solution vendor.


2. Approach
===============
Manufacturing data is characterized (see Figure 1) by Assembly Line Steps (ALS):

-   Main target is the list of failures collected at a later or final ALS (e.g at 5-th ALS).
-   These can be subset using SME to extract the failures likely related to manufacturing processes performed at an earlier ALS (e.g. 3rd ALS), even if the device **passed the existing QA test** at that earlier stage.
-   Build a training dataset by pairing each ALS specific subset of failures with measures collected at its corresponding step and earlier (e.g. ALS 1, 2, and 3).
-   Use the above described datasets to train/build machine learning (**ML**) models that will predict early (i.e. **at step 3**) failures that would happen in step 5.

![alt tag](https://cloud.githubusercontent.com/assets/16708375/19810774/6eeae1e4-9d37-11e6-8fed-13ba99f4846b.png)
<sub><sup>
Figure 1
Design of the ML modules in the CIS solution template for Predictive Analytics for Manufacturing
CM blocks indicate assembly lines from contract manufacturers. Each device goes through and is tested at the exit of each individual Assembly Line Step (ALS). Some devices, even after passing individual ALS tests, still turn out to be failures (see green modules and associated results table, with true labels in black) at the final functional test, or after they are sent to OEM. 
OEM and CMs form a Hierarchy of pipelines, where ALS are separated in time and/or space. 
Post sale/delivery failures passed the integrated test system already in place. Yet they still happen months or years after delivery and incur repair/warranty costs (see brown module and associated results table, with true labels in black).
By pairing such failures with test measurements from their likely ALS steps, we can use Machine Learning to build models (yellow) that will predict failures at early stages (e.g. ALS1 for the model with red contour).

</sup></sub>
