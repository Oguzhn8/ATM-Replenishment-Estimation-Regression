# ATM-Replenishment-Estimation-Regression
In this study, last two-week cash replenishment estimation was made for various ATMs from approximately 2 years of data.
## Dataset
The dataset contains 57 columns. While 2 columns contain ATM id and date information, the remaining 55 columns contain various information for the relevant ATM. Column names, number of non-null records and data types can be seen in the following table. <br/>
![image](https://user-images.githubusercontent.com/78887209/225242958-551a5a92-bb7e-4775-ad03-470e9ec59870.png) <br/>
It is noticed that there are NULL records in columns 42-52. The NULL Handling process will be discussed in the following sections. <br/> 
### Creating Target Variable
When the columns in the data set were examined, it was noticed that there was no ready target variable. For this reason, the money requirement for each ATM on the relevant day was created using the relevant columns in the data set. The data set includes the total amount of withdrawals and deposits of banknotes in circulation. This information is kept in deposit_5,withdrawal_5 etc columns. In addition, the 'using_taken_money' column contains the information whether the relevant ATM can recirculate the deposited money. Using this information, the target variable was created with the help of the following codes.  <br/>
![image](https://user-images.githubusercontent.com/78887209/225249169-93888cdf-8fcb-4e67-bf91-fd4a0e1cf12f.png) <br/>
If the relevant ATM can use the money it has received, the amount of money required for the relevant ATM is found by subtracting the money deposited from the withdrawn money. On the other hand, if the relevant ATM cannot use the money it has received, the money withdrawn directly is collected and the required amount of money is found.
In addition, the 'total withdrawal' and 'total deposit' columns created during the target variable generation process were also included in the variable set.
### Changing the time frequency of the dataset
The first few rows of the dataset can be seen in the figure below <br/>
![image](https://user-images.githubusercontent.com/78887209/225243596-06e88bc1-48d9-4094-844a-20de11718eef.png) <br/>
As can be seen from the figure, the data set contains information at daily frequency. In order to make weekly money replenishment estimation for the relevant ATM, the table must be arranged with a weekly frequency. While the data set is updated on a weekly basis, each column should be arranged according to the characteristics of the column. For example, columns that do not change on a daily basis and are specific to ATMs (eg nof_food, nof_deposit_casset) can be imported into the table on a weekly basis unchanged. Since the columns containing the withdrawal/deposit amounts of the banknotes in circulation and target variable change on a daily basis, they have been added to the new table by taking the weekly average. Finally, 5 categorical variables (official_holiday, special_day, holiday_before_after, is_workday, atm_place) in the data set were added to the weekly table by taking the mode value of the relevant week. The day_of_month column has been omitted from the data set as it will not make sense in the weekly data. The column week_of_month containing the week information in the relevant month, month_of_year containing the month information in the related year, and week_of_year containing the week information in the related year are included in the data set. <br/>
Daily table 'initial_data.xlsx' and weekly table 'weekly_data.xlsx' are available in excel.

## Data Preprocessing
During the data preprocessing phase, the NULL imputation process will be carried out for the columns containing NULL values mentioned in the previous section, and the encoding methods for categorical variables will be discussed. In the data preprocessing stage, NULL assignment will be made for columns containing NULL values mentioned in the previous section, and coding methods for categorical variables will be discussed. The methods to be discussed were applied on the training set of the model in order to prevent the problem of data leakage. Data leakage is when information from outside the training dataset is used to create the model. This additional information can allow the model to learn or know something that it otherwise would not know and in turn invalidate the estimated performance of the mode being constructed.
### Encoding Categorical Variables
There are 5 categorical variables in the data set. The number of unique values in each categorical variable is important for the encoding method to be applied. The chart below shows the categorical variables and unique value numbers in the data set. 
![image](https://user-images.githubusercontent.com/78887209/225440140-226fb1d6-266f-40f3-934c-261a70050901.png) <br/>
One hot encoding method is applied for categorical variables with less than 10 unique values. On the other hand, for categorical variables with more than 10 unique values, one hot encoding method can greatly enlarge the data set, so an alternative method, Target Encoding, has been applied. <br/>
#### Target Encoding
A feature with a large number of categories can be troublesome to encode: a one-hot encoding would generate too many features and alternatives, like a label encoding, might not be appropriate for that feature. A target encoding derives numbers for the categories using the feature's most important property: its relationship with the target. Target encoding is the process of replacing a categorical value with the mean of the target variable. The target variable mean of the relevant categorical variable in the data set is found and the categorical variable is replace with the mean value.
### NULL Imputation
It has been noticed that there are NULL values in 11 variables in the data set. The NULL numbers in the training set are as follows. <br/>
![image](https://user-images.githubusercontent.com/78887209/225448413-6adcd43f-bf6d-41b2-b535-0802e9398308.png) <br/> 
There are 670 rows in the training set, any of which is NULL. This information shows that in rows where any of the related variables are not NULL, none of them are NULL. The ATM codes where the respective variables get NULL values are 1, 512, 491, 133, 537, 539, 547, 549, 459, 429, and 656. When the time series of ATMs are examined, it is noticed that the related variables do not take any value other than NULL on any date. For this reason, the related variables were subjected to the NULL imputation process by means of the KNN Imputer method.
#### KNN Imputer
KNNimputer is a multivariate method used to fill out or predict the missing values in a dataset. It is a more useful method that works on the basic approach of the KNN algorithm by using variables that may be related to each other instead of the naive approach where all values are filled with the mean or median. In this approach, firstly specify a distance from the missing values which is also known as the K parameter. The missing value will be predicted in reference to the mean of the neighbours. Apart from 11 variables containing NULL values, 40 variables were used in NULL imputation. The number of neighborhoods was determined as 7, which is the approximate square root of 51 variables to be used in KNN.
## Feature Selection Steps
In the 3-stage variable elimination process, firstly, adjusted r square elimination was performed.
### Adjusted R Square Elimination
Adjusted R-squared tells us how well a set of predictor variables is able to explain the variation in the response variable, adjusted for the number of predictors in a model. In the first stage of the Feature Elimination process, it was tried to find out how well the related variable alone could explain the target variable by finding the adjusted r2 value with XGBoostRegressor separately for all non categorical variables in the data set. If the adjusted r2 value of the related variable for training set is less than 0.05, the related variable is eliminated.
### Correlation Elimination 
If your dataset has purely positive or negative attributes, the performance of the model is likely to be affected by a problem called "Multicollinearity". Multicollinearity occurs when one predictor variable in a multiple regression model can be predicted linearly with a higher degree of accuracy than the others. This can lead to skewed or misleading results. Luckily, decision trees and boosting trees algorithms immune to multicollinearity by nature. When they decide to split, the tree will choose only one of the perfectly related features. For this reason, although correlation elimination is not a necessary step for decision trees and boosting trees algorithms, the correlation elimination process was carried out in order to make the model work more efficiently and quickly. As a footnote, since logistic and linear regression algorithms are not immune to the Multicollinearity problem, correlation elimination is an essential step for these algorithms.
#### Algorithm
1- First, the correlation of each variable with the target variable is calculated. <br/> 
2- The correlation of each variable with other variables other than the target variable is calculated. <br/>
3- If the correlation between two variables is greater than 0.8, the variable with the smaller correlation with the target variable is eliminated. <br/>
### Feature Importance Elimination 
Feature importance refers to a class of techniques for assigning scores to input features to a predictive model that indicates the relative importance of each feature when making a prediction. Feature importance can be used to improve a predictive model. This can be achieved by using the importance scores to select those features to delete (lowest scores) or those features to keep (highest scores). This is a type of feature selection and can simplify the problem that is being modeled, speed up the modeling process, and in some cases, improve the performance of the model. <br/>
Obtained importance values are shown below: <br/>
![image](https://user-images.githubusercontent.com/78887209/227522270-69203c05-fa76-4087-ae02-47427bc64b42.png) <br/>
As a result of various simulations, the threshold value was determined as 0.01. The r2 scores obtained using the remaining 7 variables are given below: <br/>
![image](https://user-images.githubusercontent.com/78887209/227523231-9c62e0c5-0834-4b5f-b483-db2584d8ab49.png) <br/>
As can be seen from the table, the overfitting problem is clearly visible. XGBoost (and other gradient boosting machine routines too) has a number of parameters that can be tuned to avoid over-fitting. A few of them are as follows: <br/>
1 - colsample_bytree : the ratio of features used. Lower ratios avoid overfitting. <br/>
2 - subsample : the ratio of the training instances used. Lower ratios avoid overfitting. <br/>
3 - max_depth : the maximum depth of a tree. Lower values avoid over-fitting. <br/>
4 - gamma : the minimum loss reduction required to make a further split. Larger values avoid over-fitting. <br/>
5 - eta : the learning rate of our GBM (i.e. how much we update our prediction with each successive tree). Lower values avoid over-fitting. <br/>
6 - min_child_weight: the minimum sum of instance weight needed in a leaf, in certain applications this relates directly to the minimum number of instances needed in a node. Larger values avoid over-fitting. <br/>
7 - lambda : This term is a constant that is added to the second derivative (Hessian) of the loss function during gain and weight (prediction) calculations. Larger values avoid overfitting. <br/>
The accuracy r2 scores obtained after regularizing the hyperparameters are as follows: <br/>
![image](https://user-images.githubusercontent.com/78887209/227530922-ca51047b-8a86-45e5-9ac8-2a02faba3eee.png) <br/>
When the overfitting problem is handled, it is clearly noticed that the r2 score of the model is low. Therefore, in the next section, alternative variables generated from the data set will be discussed. <br/>
## Creating Variables from Target and Descriptive variables used in the creation of the target variable
Since the main purpose of the problem is to estimate the ATM Replenishment for the last two weeks in the data set, two-week lag variables were created. For the first two dates in the data set, 2017-01 and 2017-02, since there is no information about the past two weeks, the rows on these dates are calculated as 'NaN'. NULL imputation in these lines is done with the following method: <br/>
1 - First of all, the data set is grouped on the basis of unique_id (ATM Code). <br/>
2 - The 'NaN' rows are filled with the mean value of the relevant ATM. <br/>
Thus, the weight of the role of the values in the 'NaN' rows in the branch splitting is reduced.
It has been noticed that only the last two or a week data has been received from 4 ATMs. For this reason, NULL imputation could not be made to the related ATMs with the algorithm mentioned above. Relevant ATMs were excluded from the data set. <br/>
### Updated Feature Elimination and Modelling Process
The 'Feature Selection Steps' mentioned above were re-executed with the new variables created. Obtained adj_r2 scores and correlation values can be found in the relevant excels. Obtained feature importance values are shown in the table below: <br/>
![image](https://user-images.githubusercontent.com/78887209/227536137-5dd254f7-a39c-486c-af3d-77662a4c10b4.png)
As can be seen from the table, the new variables created are quite explanatory and important. 

SMAPE in the table means symmetric mean absolute percent error. To explain why SMAPE is used instead of mean absolute percent error, it is useful to take a look at the MAPE formula first:
![image](https://user-images.githubusercontent.com/78887209/225578914-56ae7b0a-1698-46c2-9265-8006bc4db8e6.png) <br/>
As can be seen from the formula, if the actual target variable contains the value 0, MAPE is equal to infinity. When the target variable is examined, it contains 0 values. Therefore, SMAPE, an alternative to MAPE, was used in performance evaluation. The SMAPE formula is as follows: <br/>
![image](https://user-images.githubusercontent.com/78887209/225579766-b3a3b2a9-21f0-424e-b46f-d47059355ec6.png) <br/>
As it can be clearly seen from the formula, for the SMAPE value to be infinite, both the estimated value and the actual value must be 0. As it can be seen from the performance metrics table above, since such a situation did not occur, the SMAPE function was used as a performance metric. <br/>
Final feature list and feature importance weights are shown below: <br/>
![image](https://user-images.githubusercontent.com/78887209/225581054-9c324208-0668-4d1b-9972-595f33ce002c.png) <br/>
## Modelling
In the first stage of the modeling phase, the PyCaret library, which allows to try many algorithms with a single code block, will be discussed.
### PyCaret
PyCaret is an open-source, low-code machine learning library in Python that automates machine learning workflows. Compared with the other open-source machine learning libraries, PyCaret is an alternate low-code library that can be used to replace hundreds of lines of code with a few lines only. The following code block compares many algorithms with final variables. <br/>
![image](https://user-images.githubusercontent.com/78887209/226093912-27cb2ee6-6dee-41a6-a92c-0100e566aab8.png) <br/>
The top 5 models in terms of R2 Score can be seen below: <br/>
![image](https://user-images.githubusercontent.com/78887209/226093952-28abb252-af39-4718-9702-5576ce5b2719.png) <br/>
As can be seen from the table, the best performing model was the Extra Tree Regressor model. Therefore, Extra Tree Regressor will be used as the final algorithm.
### Deep Learning
In order to make a comparative analysis, training with a deep learning network was also designed. Since deep neural networks are more successful than machine learning algorithms in capturing nonlinear relationships between features and target variable, all features are presented to the network without any elimination step.
