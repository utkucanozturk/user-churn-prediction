# Modeling

Since it is unknown what are the measures for the user retention, various machine learning algorithms are compared in terms of their ability to distinguish user retention score. To be able to assess the quality of each model a 3-fold cross-validation procedure is performed. Finally the evaluation of promising algorithms will be done in [Model Evaluation](model.md#model-evaluation).

## Evaluation Metric

Probably the most straight forward metric to evaluate regression tasks is root mean squared error (RMSE) and in this regression analysis we will use RMSE as our evaluation metric. RMSE is the square root of the average of squared errors. The RMSE serves to aggregate the magnitudes of the errors in predictions for various data points into a single measure of predictive power.

## Algorithms

We will use two gradient boosting frameworks in our modeling approach which are Extreme Gradient Boosting (XGBoost) and Light Gradient Boosting Machine (LightGBM). Both algorithms working by growing boosted decision trees. However, there is slight difference in the growing method. While XGBoost applies level-wise tree growth LightGBM applies leaf-wise tree growth. Level-wise approach meaning it grows horizontal whereas leaf-wise grows vertical. Moreover, leaf-wise approach is mostly faster than the level-wise approach making LightGBM appealing. However, leaf-wise more likely to overfit on the data, therefore we can say XGBoost has the advantage of building more robust models.

### Ligth Gradient Boosting Machine (LightGBM)

For the LightGBM parameters `number of estimators` (n_estimators), `maximum tree depth` (max_depth) and `learning rate` (learning_rate) is tuned prior to the benchmark in a 3-fold cross validation. Also, `feature selector` that removes low-variance features is tuned with different thresholds of 0, 0.01 and 0.001. Moreover, all the features are scaled to a standard scale using `standard-scaler`.

By using the data twice, once for hyperparameter tuning and once for the benchmark, we might underestimate the generalization error. However since hyperparameter tuning is computationally expensive, and we are not interested in the precise generalization error of each model but rather the general tendency of each model's performance, we refrained from nested resampling.

After the tuning, following parameter set resulted with the best mean test set score:

| Parameter | Lower Bound | Upper Bound | RMSE Optimal Value |
| - | - | - | - |
| variance_threshold | 0.001 | 0 | 0 |
| learning_rate | 0.01 | 0.2 | 0.2 |
| max_depth | -1 | 20 | -1 |
| n_estimators | 50 | 500 | 500 |

Our best model's test set score averaged at ~0.88 for the LightGBM. For further analysis, mean test score versus parameters plot can be seen below.</br></br>

<figure>
  <img src="../assets/lightgbm_tuning.png" width="800" />
  <figcaption>LightGBM Parameters vs. Mean Test Score</figcaption>
</figure>

### Extreme Gradient Boosting (XGBoost)

For the XGBoost parameters `number of estimators` (n_estimators), `maximum tree depth` (max_depth) and `learning rate` (eta) is tuned prior to the benchmark in a 3-fold cross validation. Here, `standard-scaler` used but feature selector hasn't applied since it showed no difference in the previous analysis.

After the tuning, following parameter set resulted with the best mean test set score:

| Parameter | Lower Bound | Upper Bound | RMSE Optimal Value |
| - | - | - | - |
| learning_rate | 0.05 | 0.2 | 0.05 |
| max_depth | 3 | 12 | 3 |
| n_estimators | 100 | 500 | 300 |

Our best model's test set score averaged at ~0.95 for the XGBoost. For further analysis, mean test score versus parameters plot can be seen below.</br></br>

<figure>
  <img src="../assets/xgb_tuning.png" width="800" />
  <figcaption>XGBoost Parameters vs. Mean Test Score</figcaption>
</figure>

### Support Vector Machines (SVM)

Support vector machines (SVM) try to find a separating hyperplane between the target categories. In this analysis, a radial kernel is recommended and the hyperparameters cost and gamma should be tuned in cross-validation prior to the benchmark. It would be nice to have a kernel base model trained on the dataset however it requires much more power and time to train SVMs. Therefore, we will stick to the tree-based methods which seems to have good performance on our dataset.

## Model Evaluation

Since we now know the best parameters for our models. Models trained on training set with those parameters then metrics are calculated over the whole dataset. Error metrics for the both models are tabulated below:

| Model | MSE | RMSE | R2 | MAE | MAD |
| - | - | - | - | - | - |
| XGBoost | 55.547 | 7.453 | 0.997 | 0.420 | 0.192 |
| LightGBM | 1271.374 | 35.656 | 0.924 | 0.842 | 0.061 |

By looking at the metrics above, we can say that the XGBoost outperformed the LightGBM regressor. However, we should analyze the residuals to be sure about our decision. First we would like to check whether the residuals are centered around zero or not. To do that 95% confidence interval is constructed around the residual means. For XGBoost 95% CI corresponds to `(-0.010, 0.038)` and for LightGBM it is `(-0.202, 0.026)`. For both predictors' confidence intervals covers zero then we can say that our predictions are unbiased.

Next we will be looking at the scatter plot of the residuals of the model. From the figure below, you can see the residual distribution of the predictions which fall into 99.9th percentile.</br></br>

<figure>
  <img src="../assets/scatter_residuals.png" width="800" />
  <figcaption>Scatter plot of the residuals within 99.9th percentile</figcaption>
</figure>

From the above plot, residual distributions seem to be randomly distributed with mean close to zero.

Reverse cumulative distribution of the residuals is show below in order to further asses our predictor performances. Reverse cumulative distribution of the residuals of the models are proof for the model performances. As you can see from the plot, ~90% of the residuals are fall into the range between 0 and 1 indicating little to no errors. We see steeper decrease of reverse cumulative function for the LightGBM however XGBoost covers it up in the higher residual values. Therefore, LightGBM might be a better fit for more than 80% of the data while XGBoost is more robust over the dataset.</br></br>

<figure>
  <img src="../assets/reverse_residuals.png" width="800" />
  <figcaption>Reverse cumulative distribution of the residuals</figcaption>
</figure>

Lastly, we want to compare predictions and actual scores for XGBoost model in the plot below. Here, we don't see many points far from the diagonal which is a good sign. Green points represents the errors with less than 5% mean absolute percentage and we see a lot of green points when we zoomed into the bottom-left corner of the plot. </br></br>

<figure>
  <img src="../assets/predictions_scatter.png" width="800" />
  <figcaption>Comparison between predictions and actual scores for XGBoost</figcaption>
</figure>

<figure>
  <img src="../assets/predictions_scatter_zoom.png" width="800" />
  <figcaption>Comparison between predictions and actual scores for XGBoost bottom-left zoomed</figcaption>
</figure>

### Feature Importances

#### Permutation Feature Importance

Permutation feature importance is, as the name suggests, a method to quantify the importance of a feature. It is measured by calculating the increase in the model’s prediction error after permuting the feature of interest. Intuitively, permuting or shuffling all values of a feature destroys any relationship between the given feature and the target. If the error increases by breaking the relationship, the given feature must have been important for model prediction. 

It is important to analyze which features are models mostly depending on. Therefore, feature importance plots for XGBoost and LightGBm are shown below:</br></br>

<figure>
  <img src="../assets/xgb_feature_imp.png" width="800" />
  <figcaption>Feature impotances for XGBoost</figcaption>
</figure>

<figure>
  <img src="../assets/lgbm_feature_imp.png" width="800" />
  <figcaption>Feature importances for LightGBM</figcaption>
</figure>

When we look at the feature importance plots, we can see that top 4 important features and their order are the same for both models. Those features are `outfit_liked`, `item_added`, `comment_posted` and `comment_liked` with significant contribution compare to the other features. 

#### Shapley Feature Importance

Shapley values help us to measure each feature’s contribution to the prediction of an instance, more precisely they measure how much a feature contributed to the prediction compared to the average prediction. They are calculated by averaging the marginal payout or contribution for each feature. Assuming we are interested in the contribution of feature x. First, the prediction for every combination of features not including x is calculated. Then we measure the marginal contribution of x, i.e. how adding x to each combination changes the prediction. Finally, all marginal contributions are averaged and we obtain the Shapley value for feature x. Since there are many possible coalitions of the feature values, calculating Shapley values is computationally expensive.

We want to asses whether the feature importance is also consistent when using a Shapley based measure instead of permutation feature importance. For each feature, the Shapley feature importance is the mean absolute value of the Shapley values of all observations. This means that features which have high (absolute) Shapley values for many emails have a high importance in general. Shapley values of XGBoost model can be found in the figure below. By looking at the mean shapley values of the features, we can say that permutation and shapley plots are consistent with each other on the top 4 features.</br></br>

<figure>
  <img src="../assets/shap_xgb.png" width="800" />
  <figcaption>Shapley feature impotances for XGBoost</figcaption>
</figure>

## Statistical Models

Machine learning techniques provided good results on the retention score prediction. Now, we would like to see how statistical models are performing on the user data. For that purpose we will use generalized linear models (GLMs). The GLM regression finds the most likely value of variable coefficients (ß) for the given data. In most cases this requires an iterative algorithm that maximises the log-likelihood function.

In our case, since our data is right-skewed it needs prior transformation to fit the regression models on otherwise model assumptions will be violated. Response functions that could be useful to model right-skewed outcome are poisson and negative binomial. However, those family functions require discrete response variable. In our case, the target variable that we have is continuous, positive and right-skewed. Therfore, we will use Gaussian and Gamma families and apply log-transformation on the dataset.

As our initial predictors, we will use the top 4 features from our machine learning models which are `outfit_liked`, `item_added`, `comment_posted` and `comment_liked`. Also, we will not include any interaction terms for now.</br></br>

<figure>
  <img src="../assets/stat_gauss_fit.png" width="800" />
  <figcaption>Gaussian model fit</figcaption>
</figure>

Above plot shows the fit for retention score by the gaussian model. As we go towards higher retention scores in the plot, the prediction errors seem to go larger. Therefore, residuals versus predictions are visualized below. From there, gaussian model overpredicts in the higher range of retention scores since we see larger negative errors.</br></br>

<figure>
  <img src="../assets/stat_gauss_residuals.png" width="800" />
  <figcaption>Gaussian model residuals</figcaption>
</figure>

To eliminate the tendency of overprediction, interaction terms are decided to be added to the model. Resulting model fit is visualized below.</br></br>

<figure>
  <img src="../assets/stat_gauss2_fit.png" width="800" />
  <figcaption>Gaussian model with interactions fit</figcaption>
</figure>

The model with interactions has much smaller deviance and pearson score indicating better fit to our data.

Then, gamma model applied but it wasn't a good fit to the data.

We stopped our anaylsis at this point. However, models with different link functions or penalties might be introduced to find a better fitting model. Also, different approcahes like ridge or lasso regression can be implemented with different link functions.
