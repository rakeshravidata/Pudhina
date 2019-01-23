---
layout: page
title: "One-Hot Encoding is making your Tree-Based Ensembles worse, here’s why?"
subtitle: "Optimizing Tree-Based Models"
date:   2019-01-11 21:21:21 +0530
categories: ["Data Science"]
---

<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*WMUQM7RFLh3-ej0_M1hHMQ.jpeg' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
</p>
</center>

<p>Most real-world datasets are a mix of categorical and continuous variables. While continuous variables fit into all machine learning models quite easily, the implementation of categorical variables is vastly different in different models and different programming languages. For instance, R deals with categories using a variable type called factor which elegantly fits all models without an issue and Python would require you to convert the categories into label encoding for the models to work. Python is my choice of programming language and I routinely perform some sort of encoding for categorical variables. In general, one hot encoding provides better resolution of the data for the model and most models end up performing better. It turns out this is not true for all models and to my surprise, random forest performed consistently worse for datasets with high cardinality categorical variables.</p>
  
**Test Setup**

<p>Being a Kobe Bryant fan, I decided to use his shot selection data to build a model to predict whether a shot given the circumstances would land in or not. Based on feature importance analysis, it was clear that the two variables describing the type of shot were the most important in predicting the outcome of a given shot and both variables were categorical. I ran random forest on the dataset with label encoding (assuming that there was an order) and with one-hot encoding and the outcome was nearly the same thing. In fact, it didnt seem to matter whether there was any order.</p>

**How did the models fare?**
  
<p>In addition to accuracy of prediction, I have also used the logarithmic loss of the model for comparison as that was the evaluation criteria for the Kaggle competition.</p>


**Snapshot of Performance**
<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*vWhYH9KaUeDhjNB6cOU_sw.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="35%" height="35%"/>
</p>
</center>

<p>We would normally expect one-hot encoding to improve the model but as the screenshot suggests, the model with one-hot encoding performing significantly worse than the model without it. One-hot encoding contributed to a decrease of 0.157 units of logarithmic loss. Kaggle competition winners are decided in slightest of margins and an offset of 0.05 could be the difference between being in the top 75% and top 25% on the leaderboard. Now that we have established one-hot encoding is making the models worse, let’s examine the reasons behind why this happens.</p>

**Why does this happen?**

<p>To understand this, let’s get into the inner dynamics of tree-bases models. For every tree-based algorithm, there is a sub-algorithm that is used to split the dataset into two bins based on a feature and a value. The splitting algorithm considers all possible splits (based on all features and all possibles values for each feature) and finds the most optimum split based on a criterion. We will not get into details of the criterion (there are multiple ways to do this) but qualitatively speaking, the criterion helps the sub-algorithm select the split that minimizes the impurity of bins. Purity is a proxy for the number of positive or negative samples in a particular bin in a classification problem. I found a very useful visualization that can helped me grasp this concept.</p>

**Dense Decision Tree (Model without One Hot Encoding)**
<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*rJldhOQ8qofb-UsvEDwAdg.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
</p>
</center>

<p>If a continuous variable is chosen for a split, then there would be a number of choices of values on which a tree can split and in most cases, the tree can grow in both directions. The resulting tree from a dataset containing a majority of continuous variables that would look like something in the figure to the left.</p>

**Sparse Decision Tree (Model with One Hot Encoding)**
<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*waMbIQifR03o_1hHzNYgbw.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
</p>
</center>

<p>Categorical variables are naturally disadvantaged in this case and have only a few options for splitting which results in very sparse decision trees. The situation gets worse in variables that have a small number of levels and one-hot encoding falls in this category with just two levels. The trees generally tend to grow in one direction because at every split of a categorical variable there are only two values (0 or 1). The tree grows in the direction of zeroes in the dummy variables. If that didn’t make sense, follow the example below closely.</p>

**Dataset A with Dummy Variables**
<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*UXFyRYdfpCYwfkAuMcDjMg.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="35%" height="35%"/>
</p>
</center>

<p>For instance, consider the following dataset which contains dummy variables for
categorical variables that have levels A, B, C and D. If we were to fit a decision tree to this data just to understand the splits, it would resemble the following.</p>

**Decision Tree for Dataset A**
<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*jOMNT-nHwABGVchKX0Pi3Q.jpeg' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
</p>
</center>

<p>In conclusion, if we have a categorical variable with q levels, the tree has to choose from ((2^q/2)-1) splits. For a dummy variable, there is only one possible split and this induces sparsity.</p>

**Effect on Model Performance**
  
<p>Now that we have understood why the decision trees for datasets with dummy variable look like the above figure, we can delve into understanding how this affects prediction accuracy and other performance metrics.</p>
  
<p>By one-hot encoding a categorical variable, we are inducing sparsity into the dataset which is undesirable.</p>
  
<p>From the splitting algorithm’s point of view, all the dummy variables are independent. If the tree decides to make a split on one of the dummy variables, the gain in purity per split is very marginal. As a result, the tree is very unlikely to select one of the dummy variables closer to the root.</p>
  
<p>One way to verify this is to check the feature importance for both models and see which features come out on top.</p>

**Feature Importance**
  
<p>In spite of delivering better performance, the tree based models have a slightly tarnished reputation primarily because of its interpretability which arises due to the way in which the feature importance is calculated. Feature importance in most tree-ensembles is calculated based an importance score. The importance score is a measure of how often the feature was selected for splitting and how much gain in purity was achieved as a result of the selection.</p>

<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*aMaOMQ0bIt9txo_YMcSiTQ.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
</p>
</center>

<p>The most important feature is the action_type which is a high cardinality categorical variable and clearly much more important than the ones preceding it. To provide some context, I had one-hot encoded action_type and combined_shot_type which were both high cardinality categorical variable. Obviously, dummy variables increase the dimensionality of the dataset and the curse of dimensionality comes into play and produces the chart below for feature importance. This is clearly not interpretable.</p>

<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*VT1vwxH9k_Ra1B6JkMwSJw.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
</p>
</center>

<p>To take a magnified view, I decided to look at the top 15 features which seemed to influence the prediction the most.</p>

<center>
<p class="aligncenter">
<img src="{{ '/assets/img/1*gMBpcUHN0LFCYr6JQ95_dg.png' | prepend: site.baseurl }}" class="center" alt="centered image" width="75%" height="75%"/>
  </p>
</center>

<p>When we examine the chart above, it’s clear that only one dummy variable is featured in the top 15 and that too in the 14th position which validates our hypothesis of variable selection for splitting. One-hot encoding has also obscured the order of importance of features that weren’t involved in the encoding and this makes the model inefficient.</p>
  
[Link to Github](https://github.com/rakeshravidata/Kobe-Shot-Selection-Prediction-)

  
**TLDR**

<p>One-hot encoding categorical variables with high cardinality can cause inefficiency in tree-based ensembles. Continuous variables will be given more importance than the dummy variables by the algorithm which will obscure the order of feature importance resulting in poorer performance.</p>
  
**Extra Reading**

[Visualization](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/)<br>
[Textbook](https://web.stanford.edu/~hastie/ElemStatLearn/)<br>
[Engaging Discussion](https://www.kaggle.com/c/zillow-prize-1/discussion/38793)<br>
[Summary of all types of Encoding](https://medium.com/data-design/visiting-categorical-features-and-encoding-in-decision-trees-53400fa65931)
