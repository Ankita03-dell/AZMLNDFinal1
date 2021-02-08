### Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary

# Problem Statement
The project uses the UCI Bank Marketing Dataset that contains data about banking clients, including personal details of clients like age, job, marital status etc and details regarding maketing campaign. 
The classification goal is to predict whether the client will subscribe a bank term deposit (column y).

# Solution Overview
The project approaches the problem using two different methods:
**Method 1**: Train a Scikit-learn Logistic Regression model with optmized hyperparameter tuning using HyperDrive
**Method 2**: Use AutoML to build and optimize a model
The results of the above methods are then compared.

The best performing model was the **Voting Ensemble** model selected through AutoML, which gave an accuracy of 0.91699 .
The Logistic Regression model with hyperparameters selected through HyperDrive gave an accuracy of 0.91512 .

## Scikit-learn Pipeline

**Steps involved in the entry script(train.py):-**
1. Creation of TabularDataset using TabularDatasetFactory.
   [Click here to find the Dataset used](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv)
2. Cleaning the data - removing rows with missing entries, one hot encoding the categorical data, feature engineering etc.
3. Splitting the data into train and test sets.
4. Training the logistic regression model using arguments from the HyperDrive runs.
5. Calculating the accuracy score.

**Steps involved in the project notebook(udacity-project.ipynb):-**
1. Assigning a compute cluster to be used as the target.
2. Specifying the parameter sampler(RandomParameterSampling in this project).
3. Specifying an early termination policy(BanditPolicy in this project).
4. Creating a SKLearn estimator for use with train.py.
5. Creating a HyperDriveConfig using the estimator, hyperparameter sampler, and policy.
6. Submitting the hyperdrive run to the experiment and showing run details with the widget.
7. Getting the best run id and saving the model from that run.
8. Saving the model under the workspace for deployment.

**Benefits of the parameter sampler :**
Choosing the right parameter sampling method is necessary as well a beneficial step to follow as it can have visible affects on your run time. 
Parameter sampling means to search the hyperparameter space defined for your model and select the best values of a particular hyperparameter. 
Azure supports three types of param eter sampling - Random sampling,Grid sampling and Bayesian sampling. 
RandomParameterSampling supports discrete and continous hyperparameters. 
In random sampling, hyperparameter values are chosen randomly, thus saving a lot of computational efforts.
It can also be used as a starting sampling method as we can use it to do an initial search and then continue with other sampling methods.

**Benefits of the early stopping policy :**
An early termination policy is quite helpful when the run becomes exhaustive. 
It ensures that we don't keep running the experiment for too long and end up wasting resources and time, in order to find what the optimal parameter is. 
A run is cancelled when the criteria of a specified policy are met. 
Examples of early termination policies we can use are - BanditPolicy, MedianStoppingPolicy and TruncationSelectionPolicy . 
In my project I have used BanditPolicy as the early termination policy.
This early termination policy is based on the slack factor and delay evaluation. 
It basically checks the job assigned after every 'n' number of iterations(n is passed as an argument). 
If the primary metric falls out of the slack_factor, Azure ML terminates the job.

### AutoML

**Description of the model and hyperparameters generated by AutoML**
Since it was a classifiaction task, the primary metric that was to be maximized was set to 'accuracy'. 
We provided the cleaned version of the data, and set no. of cross-validations folds to 5; this would prevent overfitting,if any (in case). 
The model was trained remotely on the compute cluster created in the beginning.
The model that gave the best results turned out to be VotingEnsembleClassifier that takes the average of the predictions of the base models. 
It gave as an accuracy score of 0.9169 i.e 0.9170 (approx) which was slightly better than the score achieved using HyperDrive.

## Pipeline comparison
**HyperDrive Model**	
*id* :	HD_7bc7e02e-eba5-466e-82d6-3f887d9c6a9e
*Accuracy* : 0.9151186315983079
**AutoML Model**	<br/>
*id* : AutoML_30fca76e-6c1f-416b-b797-fdb3335ce90f_21
*Accuracy* : 0.9169954476479514
*AUC_weighted* : 0.9471957199790382
*Algortithm* : VotingEnsemble

The difference in accuracy between the two models is rather trivial and although the HyperDrive model performed better in terms of accuracy.
I am of the opinion that the AutoML model is actually better because of its AUC_weighted metric which equals to 0.9471957199790382 and is more fit for the highly imbalanced data that we have here. 
If we were given more time to run the AutoML, the resulting model would certainly be much more better. 
And the best thing is that AutoML would make all the necessary calculations, trainings, validations, etc. without the need for us to do anything. 
This is the difference with the Scikit-learn Logistic Regression pipeline, in which we have to make any adjustments, changes, etc. by ourselves and come to a final model after many trials & errors.

## Future work
**Some areas of improvement for future experiments and how these improvements might help the model**

1. Our data is highly imbalanced.
   Class imbalance is a very common issue in classification problems in machine learning. 
   Imbalanced data negatively impact the model's accuracy because it is easy for the model to be very accurate just by predicting the majority class, while the accuracy for the      minority class can fail miserably. 
   This means that taking into account a simple metric like accuracy in order to judge how good our model is can be misleading.
   There are many ways to deal with imbalanced data. 
   These include using:
   (a)  A different metric; for example, AUC_weighted which is more fit for imbalanced data
   (b)  A different algorithm
   (c)  Random Under-Sampling of majority class
   (d)  Random Over-Sampling of minority class
   (e)  The imbalanced-learn package
   There are many other methods as well, but I will not get into much details here as it is out of scope.
   Concluding, the high data imbalance is something that can be handled in a future execution, leading to an obvious improvement of the model.
   
2. Another factor that I would improve is n_cross_validations. 
   As cross-validation is the process of taking many subsets of the full training data and training a model on each subset, the higher the number of cross validations is, the        higher the accuracy is achieved. However, a high number also raises computation time (i.e training time) thus costs so there must be a balance between the two factors.

   *Note: In case I would be able to improve n_cross_validations, I would also have to increase experiment_timeout_minutes as the current setting of 30 minutes would not be      enough.*

## Proof of cluster clean up

**Image of cluster marked for deletion**
Deletion_of_compute_target_proof
