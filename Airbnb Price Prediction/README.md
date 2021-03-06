[<img src="https://github.com/csmangum/portfolio/blob/master/img/airbnb_portfolio_title.png" width="900">](https://github.com/csmangum/portfolio/tree/master/Airbnb%20Price%20Prediction)
<img src="https://github.com/csmangum/portfolio/blob/master/img/quad_v2.gif">

# Overview
Airbnb is a popular and fast-growing alternative to traditional lodging options. It opened in 2008 in San Francisco California and quickly grew around the world. Los Angeles California is a diverse and popular destination with multiple amusement parks, beaches, and outdoor locations offering people many reasons to visit the area. With the rise in the sharing economy, Airbnb has the potential to offer both customers and hosts more choices.</p>

The purpose of this project is to predict listing price based on several possible features available from data collected through [Inside Airbnb](http://insideairbnb.com/get-the-data.html). The following high-level overview will showcase my work with the data while there will be links to the specific notebooks with all the work completed to achieve project results.

### Contents
1. [Initial Cleaning & Processing](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#1-initial-cleaning--processing)
2. [Data Exploration](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#2-data-exploration)
3. [Feature Engineering](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#3-feature-engineering)
4. [Feature Selection](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#4-feature-selection)
5. [Initial Model Development](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#5-initial-model-development)
6. [Model Optimization](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#6-model-optimization)
7. [NLP - Topic Modeling](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#7-natural-language-processing---topic-modeling)
8. [Conclusion](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/README.md#8-conclusion)
9. [Notebooks](https://github.com/csmangum/portfolio/tree/master/Airbnb%20Price%20Prediction#notebooks)

***
# 1. Initial Cleaning & Processing
The data quality is fair but there are some issues since the listings were scraped from the Airbnb website. There are 31,253 rows and more than 50 columns. For my use, I will only import 41 of them since many other are not useful for modeling. This first step includes some simple cleaning of data types and strings, followed by joining external data sources to provide further context for the algorithms. Many columns have missing data and four columns have more than 90% of values missing.

## Actions
* I removed columns that have more than 90% of values missing. Other columns with missing data will be handled later with imputation
* I removed columns 'experiences_offered', 'country_code', 'country', 'has_availability' due to having only one unique value
* Converted all the columns to appropriate data type
* Changed T/F columns to binary
* Cleaned the currency related fields, zipcodes, and percentage columns
* Removed listings with $500 or more daily price. Around 95% of listings are below this amount
* Added topic models from NLP work on the listing description (Described in a later section)
* Added zip code-based metrics for income and population

***

# 2. Data Exploration
I will look to predict daily listing price in the Los Angeles area. There are over 20,000 listings in the area with an average price around $126 a daily. Interestingly, the lowest daily price is $10 and the maximum all the way up to $10,000.

## Findings
* Most hosts respond within an hour
* Most hosts are neither verified or superhosts
* Bed type, requires license, and host has a profile pic are not a useful field
* There are a handful of features that are correlated with price
* A few features are highly correlated with each other
* The target variable is skewed and will need to be log transformed 

***Distribution of daily pricing before transformation***  
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/original_price.png" width="500">

***Heatmap of listings across Los Angeles area***  
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/heatmap.png" width="900">

***Top 10 Correlations with daily price***  
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/top_10_correlations.png" width="900">

***

# 3. Feature Engineering
My goal with feature engineering was to leverage exisitng data in a more useful way, especially for the diffirent algorithms I'll be using. In the first step of my project I imported external data sources to expand the dataset. In this step, I make alterations to the data like transforming the target and using some simple string methods to get more out of the ammenities feature.

## Actions
* From the security deposit field, I create a boolean feature if there is/ or is not a security deposit required
* With property type, 'Apartment', 'House', 'Condominium', 'Townhouse', 'Loft', and 'Guesthouse' are the most frequent. I converted all other to a misc category named 'Other'
* From the amenities column I used simple text mining to identify boolean columns of specific amenities
* Created dummy variables for categorical features
* Imputed missing values with median values
* Log transformed target variable
* Created a clustering feature based on a few variables using k-means clustering

***Distribution of daily pricing after transformation***  
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/transformed_price.png" width="500">

***

# 4. Feature Selection

With this section I wanted to see the benefit of feature selection on model performance and runtime. I used a Step-forward feature selection algorithm from the mlxtend library. In most cases, 15 features made up 95% of feature importance and I kept that number of features through the algorithm. In the end, feature selection had little impact on model performance except for some decrease in overall runtime.

<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/featue_importance.png" width="800">
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/cum_importance.png" width="600">

***

# 5. Initial Model Development

I wanted to test out three different gradient boosting algorithms to see the differences in performance and runtime. I use estimators from scikit-learn, Microsoft LightGBM, and XGBoost. My main metric for performance was using Mean Squared Error, and taking the square root to get an interpretable metric.

Model performance was incredibly similar but there was a significant difference in runtime. Scikit-learn's version took much longer and the Microsoft implementation is efficient. Next, I will optimize all three gradient boosted models and see how they compare.

<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/main_results.png" width="800">

## Best Performing Model
Below, the XGBoost barely out-performed the other two estimators. It's clear that there might be an issue with generalization with how there is a difference between train and test data.  

<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/initial_model_xgb.png" width="800">

***

# 6. Model Optimization

I used scikit learn's gridsearchcv algorithm to optimize a set of parameters for each model. This did result in high runtimes but nothing impractical. I wanted to get a good sense of performance without risking significant overfitting. Again, the LightGBM model performed better and was optimized in significantly less time. However, the model scored on the training set overfit much more based on the difference when scored on the test data.

<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/main_optimized_results.png" width="800">

## Optimized Model
Overall, the LightGBM is my final choice with the much smaller runtime and similar performance to the others. I still have concerns with it's ability to generalize to unseen data with the difference between results here and with the training data.

<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/optimized_model_lightgbm.png" width="800">

***

# 7. Natural Language Processing - Topic Modeling

## Latent Dirichlet Allocation

### Process:
1. Tokenize listing descriptions
2. Remove stopwords and punctuation
3. Make bigrams
4. Lemmatization
5. Apply the Gensim LDA model
6. Optimize the model based on two coherence scores: C_V and U_MASS

**Images below are the two coherence scores during model optimization. In the end, I went with 8 topics.**  
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/nlp_cv_coherence.png" width="800">
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/nlp_umass_coherence.png" width="800">

## Wordclouds
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/topic_9_wordcloud.png" width="400"><img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/new_topic_2_wordcloud.png" width="400">

## Initial Model Development
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/nlp_results.png" width="800">


## Model Optimization
<img src="https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/img/nlp_optimized_results.png" width="800">

# 8. Conclusion
Unsurprisingly, the three boosting-based estimators performed similar in multiple circumstances, What was surprising, was how efficient Microsoft’s implementation is compared to scikit-learn's. Feature selection through a step-forward algorithm and very little impact on the metrics but had a small impact on runtime. Also, applying NLP to the listing description did provide a small impact to the overall results.

My final selected model was the optimized LightGB model with a root-mean-squared-error of $43.47. The number of people the listing accommodates is a large influencer to the model as well as a cleaning fee. Both follow common sense thinking but it was interesting to see some of my engineered features like distance to Downtown LA as such a powerful predictor. There is some instability in the model and it likely won't generalize well. This model is specifically fitted to the market area and does not transfer over to other markets.


## Future Plans
1. NLP sentiment analysis with listing reviews
2. Look to incoproate more external data sources
3. Leverage more than one market area
4. Further geospatial analysis

## Notebooks
1. [Initial Cleaning & Processing](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/notebooks/1.%20Initial%20Cleaning%20%26%20Processing.ipynb)
2. [Data Exploration](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/notebooks/2.%20Data%20Exploration.ipynb)
3. [Feature Engineering](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/notebooks/3.%20Feature%20Engineering.ipynb) 
4. [Feature Selection](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/notebooks/4.%20Feature%20Selection.ipynb)
5. [Model Selection and Optimization](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/notebooks/5.%20Model%20Selection%20%26%20Optimization.ipynb) 
6. [NLP - Topic Modeling](https://github.com/csmangum/portfolio/blob/master/Airbnb%20Price%20Prediction/notebooks/Topic%20Modeling%20-%20NLP.ipynb)
