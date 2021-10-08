# Dataset

Social media platforms often look at when users churn. In order to analyze the user churn we will use a dataset of user events. The dataset contains information about the users, i.e., the events they performed, the count of each event, as well as the target variable indicating retention score of the user. The lower the retention score, the higher the probability of users
churning. The dataset includes data from 373296 users and 52 user activities, features. Features include activities like comment liked, comment posted, screen view.

## First Look

The dataset does not include any missing feature values. We see positive skewnes in the target variable and feature distributions. Therefore, we can say that none of the columns in the dataset follow normal distribution. To analyze it further, skewness of the features and the target variable is visualized below.</br></br>

<figure>
  <img src="../assets/skewness.png" width="800" />
  <figcaption>Skewness of the features and the target variable</figcaption>
</figure>

As you can see from the graph above, all of the variables are right-skewed and some of them have significant skewness in their distribution. Positive skewness is the consequence of large number of zeros in the feature columns. Since our features represent the number of event occurences for a user, we would expect to have many zeros in the columns. One step further, the percentage of non-zero values for each feature column is visualized in the plot below.</br></br>

<figure>
  <img src="../assets/nonzero_percent.png" width="800" />
  <figcaption>Non-zero Percentage of the features</figcaption>
</figure>

By looking at the column non-zero percentages, we can see that `screen_view` has the least non-zero rows with 25% of user database having this activity. `screen_view` is followed by `first_open`, `item_added` and `item_removed` whose percentages lie between 10 and 20. Furthermore, for some features, events, users have little to none activity such as `user_blocked`, `firebase_in_app_message_action`, `firebase_in_app_message_dismiss` and 22 other features has less than 1% nonzero values that represent user acitivity. Therefore, our inference from this values is: some variables have slight to none variability which might indicate less information gain on those variables; this will be taking into account in the feature selection stage of our model tuning stage.

## Correlations

Analyzing correlations between variable pairs is a basic but significant part of our dataset analysis. Therefore, the correlations are visualized in a correlation heatmap in the figure below. It can be seen that almost all pairs of variables have an absolute correlation coefficient between 0 and 0.5, indicating a low or negligible correlation. Also, only strongly correlated pair in the dataset seem to be the `retentionScore-item_added` pair which can be seen in the dark red color in the heatmap. Other than that, there are little to no correlations among the features meaning no multicollinearity exist in the dataset.</br></br>

<figure>
  <img src="../assets/corr.png" width="800" />
  <figcaption>Correlations Heatmap</figcaption>
</figure>

Now let us focus on the feature-target variable pair correlations. Feature-target variable, `retentionScore`, correlations are analyzed and ten of those having highest absolute correlation coefficient are visualized below.</br></br>

<figure>
  <img src="../assets/corr_target.png" width="800" />
  <figcaption>Top 10 Predictor Correlations with the Target Variable</figcaption>
</figure>

In the graph above, you can see that event `item_added` is the highest correlated feature with `retentionScore` with positive correlation coefficient of 0.94. We also see that `outfit_liked` has the second highest correlation with the target variable with positive correlation coefficient of 0.34. Moreover, we see low positive correlation with events like `comment_liked`, `comment_posted`, and low negative correlation with events like `app_remove`. Later, we will analyze features' impacts on the prediction when proper machine learning models are used.

## Analysis of Feature - Target Patterns

The correlations of features and the target variable gave us the first insight to the feature-target relationships. Now, we will analyze those relationships furher by focusing on the first five most correlated features. To do that, we would like to have less-skewed and equal-spreaded data. Also, since we are interested in retetion score of the user it would be useful to have a look at the target variable from narrower scale. Therefore, the target variable is log-transformed and rounded into categories. Discrete log-retention score distributions of the users are shown below.</br></br>

<figure>
  <img src="../assets/target_value_counts.png" width="800" />
  <figcaption># of Users in the Retention Category</figcaption>
</figure>

We want to see value distributions of the features over the discretized retention scores of the users are differ from each other. To analyze that, firstly, mean event counts for each retention category is visualized below:</br></br>

<figure>
  <img src="../assets/mean_imp_bar.png" width="800" />
  <figcaption>Average event counts grouped by retention category</figcaption>
</figure>

As we can see from the figure above, event counts for the lower retention categories, meaning lower retention scores, are close to zero compare to higher scores. Therefore, we see positive relationship between these event counts and the retention scores. However, as we saw from the target value counts plot above, sample sizes between categories can be significantly different, and zero values in the categories might be misleading us in the large sized categories. Therefore, currently no inference can be made; features needs to be further analyzed in the model evaluation stage.

Next, boxplot of the event counts for each retention category is visualized below. By looking at the plot, we see that outliers are mostly exist in the higher retention scores except `item_removed`. Moreover, for `item_added` we see increasing medians with retention scores.</br></br>

<figure>
  <img src="../assets/box_imp_var.png" width="800" />
  <figcaption>Boxplots of event counts grouped by retention category</figcaption>
</figure>

<figure>
  <img src="../assets/box_item_added.png" width="800" />
  <figcaption>Boxplot of item_added counts grouped by retention category</figcaption>
</figure>

Finally, `item_added` versus `retentionScore` scatter with regression line is plotted and visualized below. From there we see a regression line with positive slope fitted between the variables.</br></br>

<figure>
  <img src="../assets/item_added_vs_score.png" width="800" />
  <figcaption>item_added vs. retentionScore scatter plot with fitted regression line</figcaption>
</figure>

Now, that we have a initial understanding of the dataset, we want to first build machine learning models, which can predict user retention scores as precise as possible (see section [Modeling](model.md)) and then evaluate their performance.
