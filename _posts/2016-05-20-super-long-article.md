---
layout: page
title:  "A Practical Approach to Dealing With Categorical Variables During Modelling"
date:   2018-12-23 21:21:21 +0530
categories: "Data Science"
---


<img src="{{ '/assets/img/1*dmh91HqnhxnMtF3tt2Z32A.jpeg' | prepend: site.baseurl }}"  width="1000%" height="1000%">


<p>All Data Analysts have reached a point in solving a problem where they end up with a dataset that has nothing but categorical variables. You are probably thinking that this isnt a big issue because you can just one-hot encode them (If you have not heard of one-hot encoding — read this) or create label encoders. But what do you when you have a dataset with a large number of categorical variables and your local machine is not a supercomputer?</p>
  
**Curse of Dimensionality**

<p>I have been working on a project to predict flight delays using supervising learning methods. The dataset that I used was from Kaggle (click here to access the dataset) and was acquired from the US Department of Transportation for the year 2015. Each row in the dataset is an individual flight, and contains information on the airline, airports (name, location, origin airport, destination airport) and flight level information such as time schedule (day,month,year), flight distance, departure time and delay, etc. Our dataset contains around 6 million flights for 14 different airlines amongst 322 airports. As these are records that have happened in the past, most columns or features are expected to be categorical in nature.</p>

**Snapshot of the Dataset**

<p>To give you a rough idea of the scale, there are 322 different airport codes in both airport fields (Origin and destination), nearly 14 airlines, 12 months and nearly 30 days in each month. If you were to use one-hot encoding for the problem, the number of columns in the dataset will increase to 522,567,360. Most classification models require a high amount of computation power to train on 500 million rows which is generally available in cloud computing services like AWS or GCP.
But, what are my options now?</p>

<img src="{{ '/assets/img/1*shu_QNp6umKtZvKX80tblQ.png' | prepend: site.baseurl }}" id="about-img">
  
<p>In order to reduce dimensionality, the variable with the highest number of levels can be grouped into clusters. In the case of my project, the origin and destination airport have the highest number of levels and can be grouped into clusters. But clustering will reduce resolution of the data and the model might not learn the nuances between different levels of the variable. As a result, the model will not perform as well as we would expect and will have a lower AUC which is undesirable.
Instead of training the model on the whole dataset, what if you trained a separate model for each airport? For each airport, the number of columns reduces to about 1 million and the average number of rows reduces to a manageable 2000. This is a huge improvement over 6 Million Rows and 550 Million Columns.</p>
  
**Steps of Execution**

<p>Step 1: Subset the data based on the categorical variable that has the highest number of levels. In this case, the airport of origin can be used to subset the data into 322 different chunks of data.</p>
<p>Step 2: Perform train-test splits for each chunk of data separately and train the classifier on the train split of each chunk.</p>
<p>Step 3: Record predictions on the test split of each chunk with their corresponding classifiers. Combine all predictions into a single dataset.</p>
<p>Step 4: Perform hyper-parametric tuning for the whole dataset or for individual classifiers depending on the number of levels in each category.</p>

**Layout for Training Multiple Classifiers**

<img src="{{ '/assets/img/1*fZUExkhlDojlMDwwVi7Z1A.png' | prepend: site.baseurl }}" id="about-img">

**Result**
<p>The models, although there were 322 of them, ran on my MacBook Pro with a 8 GB of RAM without any problems. In addition, the Area Under Curve for the overall prediction was 0.65 which is a tad lesser than a model for the whole dataset (Run on AWS) which yielded 0.68. I tried using random forest, SVM and logistic regression and saw the best results from logisitic regression. This can be easily applied to any problem where the regressor that is the most predictive of the response is a categorical variable with a large number of levels.</p>
  
<p>Click here to access the github repo.</p>
