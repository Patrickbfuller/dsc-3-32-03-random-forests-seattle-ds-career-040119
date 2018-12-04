
# Random Forests


## Introduction

In this lesson, we'll learn about a powerful and popular ensemble method that makes use of Decision Trees--a Random Forest!


## Objectives

You will be able to:

* Understand and explain how the Random Forests algorithm works
* Understand and explain how Random Forests make use of Bagging as a technique
* Understand and explain the subspace sampling method used in Random Forests, and why it makes Random Forests more resilient to noisy datasets and overfitting

## Understanding the Random Forest Algorithm

The **_Random Forest_** algorithm is a supervised learning algorithm that can be used for classification or regression tasks. If our task is a classification task, then our Random Forest will consist of many different Decision Trees for classification-if regression, then the trees will be built for regression. Decision Trees are the cornerstone of Random Forests--if you don't remember much about Decision Trees, now may be a good time to go back and review that section until you feel comfortable with the topic. 

Put simply, the Random Forest algorithm is an ensemble of Decision Trees. However, you may recall that Decision Trees are a **_greedy algorithm_**, meaning that given the same data, the algorithm will make choice that maximizes information gain at every step. By itself, this presents a problem--it doesn't matter how many trees we add to our forest, if they're all the same tree! Trees trained on the same dataset will come out the exact same way every time--there is no randomness to this algorithm. It doesn't matter if our forest has a million decision trees; if they are all exactly the same, then our performance will be no better than if we just had a single tree.

Think about this from a business perspective--would you rather have a team at your disposal where everyone has exactly the same training and skills, or a team where each member has their own individual strengths and weaknesses? The second team will almost always do much better!

As we learned when reading up on Ensemble Methods, variance is a good thing in any ensemble. So how do we create high variance among all the trees in our Random Forest? The answer lies in two clever techniques that the algorithm uses to make sure that each tree focuses on different things--**_Bagging_** and the **_Subspace Sampling Method_**.


## Bagging The Data

The first way to encourage differences among the trees in our forest is to train them on different samples of data. Although more data is generally better, if we gave every tree the entire dataset, we would end up with each tree being exactly the same. Because of this, we instead **_Bootstrap Aggregation_**, or **_Bagging_**, to a portion of our data with replacement. For each tree, we sample 2/3 of our training data with replacement--this is the data that will be used to to build our tree. The remaining data is used as an internal test set to test each tree--this remaining third is referred to as **_Out-Of-Bag Data_**, or **_OOB_**. For each new tree created, the algorithm then uses the remaining 1/3 of data that wasn't sampled to calculate the **_Out-Of-Bag Error_**, in order to get a running, unbiased estimate of overall tree performance for each tree in the forest. 

Training each tree on it's own individual "bag" of data is a great start for getting us some variability between the Decision Trees in our Forest. However, with just bagging, all the trees are still focusing on all the same predictors. This allows for a potential weakness to affect all the trees at once--if a predictor that usually provides strong signal provides bad information for a given observation, then it's likely that all the trees will fall for this false signal and make the wrong prediction. This is where the second major part of the Random Forest algorithm comes in!

## The Subspace Sampling Method

After bagging the data, the Random Forest uses the **_Subspace Sampling Method_** to further increase variability between the trees in the Random Forest. Although it has a fancy mathematical-sounding name, the Subspace Sampling Method refers to randomly selecting a subset of features to use as predictors for training a decision tree, instead of using all predictors available. 

Let's pretend we're training our Random Forest on a dataset with 3000 rows, and 10 columns. For each given tree, we would randomly "bag" 2000 rows with replacement. Next, we subspace sample by randomly selecting a number of predictors. Exactly how many predictors are used is a tunable parameter for this algorithm--for simplicity's sake, let's assume we pick 6 predictors in this example. 

This brings us to the following pseudocode so far:

For each tree in the dataset:

1. Bag 2/3 of the overall data--in our example, 2000 rows.
2. Randomly select a set number of features to use for training this particular tree--in this example, 6 features. 
3. Train the tree on modified dataset, which is now a DataFrame consisting of 2000 rows and 6 columns. 
4. Drop the unused columns from step 3 from the Out-Of-Bag rows that weren't bagged in step 2, and then use this as an internal testing set to calculate the Out-Of-Bag Error for this particular tree.

<img src='rf-diagram.png' height=75% width=75%>

### Resiliency to Overfitting

Once we've created our target number of trees, we'll be left with Random Forest filled with a diverse set of Decision Trees that trained on different sets of data, and also look at different subsets of features to make predictions. This amount of diversity among the trees in our forest will make for a model that is extremely resilient to noisy data, thus reducing the chance of overfitting.

To understand why this is the case, let's put it in practical terms. Let's assume that of the 10 columns that we mentioned in our hypothetical dataset, column 2 correlates heavily with our target. However, there is still some noise in this dataset, and this column doesn't correlate _perfectly_ with our target--there will times where it suggests one class or another, but this isn't actually the case--let's call these rows "false signals". In the case of a single decision tree, or even a forest where all trees focus on all the same predictors, we can expect to get the model to almost always get these false signal examples wrong. Why? Because the model will have learned to treat column 2 as a "star player" of sorts. When column 2 provides false signal, our models will fall for it, and get the prediction wrong. 

Now, let's assume that we have a Random Forest, complete with Subspace Sampling. If we randomly select 6 out of 10 predictors to use when creating each tree, then this means that ~40% of the trees in our forest won't even know Column 2 exists! In the cases where Column 2 provides "false signal", the trees that use column 2 will likely make an incorrect prediction--but that only matters to the ~60% that look at Column 2. Our forest still contains another 40% of trees that are essentially "immune" to the false signal in Column 2, because they don't use that predictor. In this way, the "wisdom of the crowd" buffers the performance of every constituent in that crowd. Although for any given example, some trees may draw the wrong conclusion from a particular predictor, the odds that _every tree_ makes the same mistake because they looked at the same predictor is infinitesimally small!

### Making Predictions With Random Forests

Once we have trained all the trees in our Random Forest, we can effectively use it to make predictions! When given data to make predictions on, the algorithm provides only the appropriate features to each tree in the forest, gets that tree's individual prediction, and then aggregates all predictions together to determine the overall prediction that the algorithm will make for said data. In essence, each tree "votes" for the prediction that the Forest will make, with the majority winning. 

## Benefits and Drawbacks

Like any algorithm, Random Forest come with their own benefits and drawbacks. 

### Benefits

* **_Strong performance_**: The Random Forest algorithm usually has very strong performance on most problems, when compared with other classification algorithms. Because this is an ensemble algorithm, the model is naturally resistant to noise and variance in the data, and generally tends to perform quite well. 

* **_Interpretability_**:  Conveniently, since each tree in the Random Forest is a **_Glass-Box Model_** (meaning that the model is interpretable, allowing us to see how it arrived at a certain decision), the overall Random Forest is, as well! You'll demonstrate this yourself in the upcoming lab, by inspecting feature importances for both individual trees and the entire Random Forest itself. 

### Drawbacks

* **_Computational Complexity_**: Like any ensemble method, training multiple models means paying the computational cost of training each model. On large datasets, the runtime can be quite slow compared to other algorithms.

* **_Memory Usage_**: Another side effect of the ensembled nature of this algorithm, having multiple models means storing each in memory. Random Forests tend to have a larger memory footprint that other models. Whereas a parametric model like a Logistic Regression just needs to store each of the coefficients, a Random Forest has to remember every aspect of every tree! It's not uncommon to see larger Random Forests that were trained on large datasets have memory footprints in the 10s, or even hundreds of MB. For data scientists working on modern computers, this isn't typically a problem--however, there are special cases where the memory footprint can make this an untenable choice--for instance, an app on a smartphone that uses machine learning may not be able to afford to spend that much disk space on a Random Forest model!


## (Optional) Reading the Random Forests White Paper

This algorithm was not invented all at once--there were several iterations by different researchers that built upon each previous idea. However, the version used today is the one created by Leo Breiman and Adele Cutler, who also own the trademark for the name "Random Forest". 

Although not strictly necessary for understanding how to use Random Forests, we highly recommend taking a look at the following resources from Breiman and Cutler if you're interested in really digging into how Random Forests work:

[Random Forests Paper](https://www.stat.berkeley.edu/~breiman/randomforest2001.pdf)

[Random Forests Website](https://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm)

## Summary

In this lesson, you learned about a random forest, which is a powerful and popular ensemble method that uses decision trees!

