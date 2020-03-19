
# Building a Regression model

-------------------------

### This is just an example of how applying Machine Learning with Google Bigquery.
https://cloud.google.com/bigquery-ml/docs

---------------------


Let's remember what we can do

![alt](../pics/tablecontent.png "")

------------------------------

Let's use the london_bicycles dataset

Let's assume we have 2 types of bicycles: hardy commuter bikes and fast but fragile road bikes. if a bicycle rental is likely to be for a long duration, we need to have road bikes in stock, but if the rental is likely to be for a short duration, we need to have commuter bikes in stock. Therefore, to build a system to properly stock bicycles, we need to predict the duration of bicycle rentals.

- Choose the label

Because the goal of our first model is to predict the duration of a rental based on our historical dataset of cycle rentals, the label is the duration of the rental.
However, is this the correct objective for the problem? Should we be predictiong the duration of each rental, or should we be predicting the total duration of all rentals at a station over, ofr instance, an hour? If the latter is the better formulation, the label should be the sum of all the rentals in a specific hour. It's just there are a lot of options. Choose carefully your label.

In this case, let's decide we need to build 2 models: one in which we predict the duration of a rental, and the other in which we predict the probability that the rental will be longer than 30 minutes. Then we have the end user make their decission based on the 2 predictions


- Exploring the dataset to find features

Feature engineering is often the most important part of building accurate machine learning models. Good featureengineering requires deep understanding of the data and the domain. It is often a process of hypothesis testing; you have an idea for a feature, you check to see whether it works (has mutual information with the label), and then you add it to the model. If it doesn't work, you try the next idea

- 1st. we'll check all possible features
---------------------

### Impact of station

To check wether the duration of a rental varies by station, you can visualize the result of the following query in Data Studio using t
he start_station_name as the dimension and duration as metric (in google data studio)

        SELECT
        start_station_name,
        AVG( duration)
        FROM
        `bigquery-public-data.london_bicycles.cycle_hire`
        GROUP BY
        start_station_name

Run it and select EXPLORE DATA

![alt](../pics/start_station.png "")
f_0 = duration (in seconds), ojo

There are just a few stations that are associated with long-duration rentals (over 3k seconds), but that the majority of station have durations that lie in a relatively narrow range. Had all the stations in London been associated with durations within a narrow range, the station at which the rental commenced would not ahve been a good feature. But in this problem it is demonstrated the start_station_name does matter.

Note that you cannot use end_station_name as feature because at the time the bicycle is being rented, you won't know to chich station the bicycle is going to be returned. Because we are creating a machine learning model to predict events in the future, you need to be mindful of not using any columns that whill not be known at the time the prediciton is made.

### Day of the week

We perform similarly. you can check whether day of week (or hour) matters

        SELECT
        -- dayofweek is a parameter. check it out
        EXTRACT(dayofweek
        FROM
            start_date) AS dayofweek,
        AVG(duration) AS duration
        FROM
        `bigquery-public-data.london_bicycles.cycle_hire`
        GROUP BY
        dayofweek

![alt](../pics/daysofweekregression.png "")

![alt](../pics/regressionourofday.png "")

- It looks like duration are longer on weekends than weekdays. Similarly, durations are longer early in them orning and in the midafternoon, so dayofweek and hourofday are good features

- Number of bicycles

A potential feature is the number of bikes per station

        SELECT
        bikes_count,
        AVG(duration) AS duration
        FROM
        `bigquery-public-data.london_bicycles.cycle_hire` a
        JOIN
        `bigquery-public-data.london_bicycles.cycle_stations` b
        ON
        a.start_station_name = b.name
        GROUP BY
        bikes_count

![alt](../pics/regression_bikes.png "")
We can see the relationship is noisy with no visible trend, so the number of bicycles is not a good feature.

----------------------------------------------------
### Remember that for joins you must alias the tables (a,b, for example)

- There's a mathematical way of checking this

        SELECT
        CORR(bikes_count,
            duration) AS CORR
        FROM
        `bigquery-public-data.london_bicycles.cycle_hire` a
        JOIN
        `bigquery-public-data.london_bicycles.cycle_stations` b
        ON
        a.start_station_name = b.name

    Pearson coefficient = -0.0049 which means it's independent. 
    ### (Pearson coefficient goes from 0 to 1, so you can imagine) 

- The Pearson correlation coefficient isn't a perfect test for whether a feature is useful because it looks only at linear dependence. Sometimes a feature might have a nonlinear dependence with the label. Still, the Pearson coefficient is a good starting point. (Maybe it's better to link to Colab and create a correlation Matrix)
------------------------------------------------------------

I'm trying to query from Google Colab and use a proper correlation matrix to evaluate together all features we saw before, but it's taking goddamn time

### Correlation Matrix in Google Colab 

(if you're working with 1GB it can take a lot)

        from google.colab import auth
        auth.authenticate_user()
        print('Authenticated')

To use pandas

        %load_ext google.colab.data_table

To query whatever you're querying:

        %%bigquery --project yourprojectid <== CHANGE THIS STUFF!!!

        SELECT
        bikes_count,
        EXTRACT(dayofweek
        FROM
            start_date) AS dayofweek,
        EXTRACT(hour
        FROM
            start_date) AS hourofday,
        AVG(duration) AS duration,
        start_station_name
        FROM
        `bigquery-public-data.london_bicycles.cycle_hire` a
        JOIN
        `bigquery-public-data.london_bicycles.cycle_stations` b
        ON
        a.start_station_name = b.name
        GROUP BY
        bikes_count,
        start_date,
        start_station_name

        df

        %matplotlib inline
        # https://github.com/albertovpd/datamad1019/blob/lab-supervised-learning/module-3/lab-supervised-learning/your-code/main.ipynb

        import numpy as np
        import pandas as pd
        import matplotlib.pyplot as plt
        import seaborn as sns

        corre = df.corr()
        sns.heatmap(corre)

![alt](../pics/correlation_matrix.png "")

- Well, this result is dumb, but helpful when working with great amount of labels. Remember to erase columns with strong correlation as first approach (but try everything before deleting them)

## (This is already donde in other repo, better done, or just done like it should be)

-------------------------------------------------------------

# Creating a Training Dataset

After taking a glance to the daset, we can prepare the training dataset by pulling out the selected features and the label:

- We do a CAST to transform the date into string. Why we do that? 

#### Feature columns have to be either numeric (INT64, FLOAT64, etc) or categorical (STRING). Datestuff is not allowed. So to prepare the dataset we need to pull out the selected features and label. If the feature is numeric but needs to be treated  as categorical, we need to cast it as a STRING.

        SELECT 
        duration,
        start_station_name,
        CAST(EXTRACT(dayofweek FROM start_date) AS STRING ) as dayofweek,
        CAST(EXTRACT(hour FROM start_date) AS STRING ) as hourofday
        FROM `bigquery-public-data.london_bicycles.cycle_hire` 

If preparing the data involves computationally expensive transformations or joins, it might be a good idea to save the prepared training data as a table so as to not repeat that work during experimentation. 

# Training and evaluating the model 

- This can be PURE SHIT. If the dataset you're working with is stored in EEUU and you're in other part, you won't be able to do shit. So you need to create a project and copy the dataset or tables to your project (and copying costs the same than querying).
- Finally, it's quite a fuss the way if naming the project. What I did:

        - create an empty dataset called vargas_data_studies in isentropic-road...
        - copy the tables I was working with the London bikes dataset

        CREATE OR REPLACE MODEL `isentropic-road-260315.vargas_data_studies.regression_london_bicycles`

        OPTIONS(input_label_cols=["duration"], model_type="linear_reg")
        as

        SELECT
        duration,
        start_station_name,
        CAST(EXTRACT(dayofweek FROM start_date) AS STRING ) as dayofweek,
        CAST(EXTRACT(hour FROM start_date) AS STRING ) as hourofday

        FROM `isentropic-road-260315.vargas_data_studies.london_bicycles_cycle_hire` 

- The label is numeric, so this is a regression problem. 

- The SELECT statement above prepares the training dataset and pulls in the label and feature columns.

# Evaluating the model

- In the created table, we can check "Training", "Evaluation", they contains useful information. In this particular case the coefficient of determination is R²=0.0036, which is kind of absolutely
 terrible.

 You can also check the evaluations with a query

        SELECT * FROM ML.EVALUATE(MODEL `isentropic-road-260315.vargas_data_studies.regression_london_bicycles`)

![alt](../pics/terrible.png "")
Mean absolute error is in seconds. Check out the dimension of other metrics

- Note that the query options also identifies the model type. Here, we have picked the simplest regression model that BigQuery supports. We strongly encourage you to pick the simplest model and to spend a lot of time considering and bringing in alternate data choices, because the payoff of a new/improved input feature greatly outweighs the payoff of a better model.
#### Only when you've reached the limits of your data experimentation should you try more complex models. (In the end, try stuff, multiply stuff, check features).

### Combining days of the week

There are other ways that you could have chosen to represent the features that you have. For example, recall that when we exported the relationship between dayofweek and the duration of rentals, we found that durations were longer on weekends than on weekdays. Therefore, instead of treating the raw value of dayofweek as a feature, you can employ this insight by fusing several dayofweek values into  the weekday category

        CREATE OR REPLACE MODEL
        `isentropic-road-260315.vargas_data_studies.regression_london_bicycles_weekday` OPTIONS(input_label_cols=["duration"],
            model_type="linear_reg") AS
        SELECT
        duration,
        start_station_name,
        IF
        (EXTRACT(dayofweek
            FROM
            start_date) BETWEEN 2
            AND 6,
            "weekday",
            "weekend") AS dayofweek,
            -- Esto es como el if/else
        CAST(EXTRACT(hour
            FROM
            start_date) AS STRING ) AS hourofday
        FROM
        `isentropic-road-260315.vargas_data_studies.london_bicycles_cycle_hire`

- This model has a mean abolute error of 966.8886, which is less than the model we did before (1,025.5926), nevertheless R² is even more terrible. I'm starting to doubt about the R squared being R²...It makes no sense (the book says it is an improvement)

- So let's keep on, because the Mean Absolute Error decreased and also all the other metrics

### Bucketizing thehour of day

Based on the realationship between hourofday and the duration, you can experiment with bucketizing the variable into 4 bins (-inf,5), [5,10),[10,17),[17,inf):

        CREATE OR REPLACE MODEL
        `isentropic-road-260315.vargas_data_studies.regression_london_bicycles_bucketized` OPTIONS(input_label_cols=["duration"],
            model_type="linear_reg") AS
        SELECT
        duration,
        start_station_name,
        IF
        (EXTRACT(dayofweek
            FROM
            start_date) BETWEEN 2
            AND 6,
            "weekday",
            "weekend") AS dayofweek,
        -- Esto es como el if/else
        -- Abajo: quizás bucketize te convierte las cosas directamente en int, checkea eso
        ML.BUCKETIZE(EXTRACT(hour
            FROM
            start_date),
            [5,
            10,
            17]) AS hourofday
        FROM
        `isentropic-road-260315.vargas_data_studies.london_bicycles_cycle_hire`

With this the mean absolute error decreased. So let's keep it.

# Predicting with the model

We can try out the prediction by passing in a set of rows for which to predict. Nevertheless, you can't obtain the predicted duration of a rental in Hyde Park at 5pm on a Tuesday using this code, because you grouped by weekday/weekend dude. Be careful. 
Anyway, you can saltar-a-la-torera-este-jaleo using TRANSFORM

        CREATE OR REPLACE MODEL
        `isentropic-road-260315.vargas_data_studies.regression_london_bicycles_bucketized_except` transform(* EXCEPT(start_date),
        IF
            (EXTRACT(dayofweek
            FROM
                start_date) BETWEEN 2
            AND 6,
            "weekday",
            "weekend") AS dayofweek,
            ML.BUCKETIZE(EXTRACT(HOUR
            FROM
                start_date),
            [5,
            10,
            17]) AS hourofday) OPTIONS (input_label_cols=["duration"],
            model_type="linear_reg") AS
        SELECT
        duration,
        start_station_name,
        start_date
        FROM
        `isentropic-road-260315.vargas_data_studies.london_bicycles_cycle_hire`

- Use the TRANSFORM clause and formulate the ML problem in such a way that anyone requiring prediction needs to provide just the raw data.

- The advantage of placing all preprocessing functions inside the TRANSFORM clause is that clients of the model do not need to know what kind of preprocessing has been carried out. Best practice, therefore, is to have the SELECT statement in a training query return just the raw data, and have all transformations done in the TRANSFORM clause.

## With the TRANSFORM clause, the prediction query becomes:

        SELECT
        *
        FROM
        ML.PREDICT(MODEL `isentropic-road-260315.vargas_data_studies.regression_london_bicycles_bucketized_except`,
            (
            SELECT
            "Park Lane, Hyde Park" AS start_station_name,
            CURRENT_TIMESTAMP() AS start_date))

    - And you'll have a prediction. a terrible one

## Generating batch predictions

You can also create a table of predictions for every hour at every station, starting at 3 a.m. the next day, using array generation:

        DECLARE tomorrow_3am TIMESTAMP;
        SET tomorrow_3am = TIMESTAMP_ADD(TIMESTAMP(DATE_ADD(CURRENT_DATE(), INTERVAL 1 DAY)),
        INTERVAL 3 HOUR);

        WITH generated AS (
        SELECT 
        name AS start_station_name,
        GENERATE_TIMESTAMP_ARRAY (
        tomorrow_3am,
        TIMESTAMP_ADD(tomorrow_3am, INTERVAL 24 HOUR),
        INTERVAL 1 HOUR) AS dates



        FROM `isentropic-road-260315.vargas_data_studies.london_bicycles_cycle_stations` ),

        features AS (
        SELECT
        start_station_name,
        start_date

        FROM

        generated,
        UNNEST(dates) AS start_date)

        SELECT * FROM ML.PREDICT(MODEL `vargas_data_studies.regression_london_bicycles_bucketized_except`, (SELECT * FROM features))

So we did everything just queryng stuff.

## Examining model weights 

A linear regression model... It's fucking linear. Not cuadratic, not cubic. It's a sum or subtract of features. To examine the given weights to each column:

        SELECT
        *
        FROM
        ML.WEIGHTS(MODEL `isentropic-road-260315.vargas_data_studies.regression_london_bicycles_bucketized_except`)


![alt](../pics/weightseconds.png "")

This means that the contribution to this feature to the overal predicted duration is 3.07 seconds.

The weights of different input features are not very meaningful - pretty much the only reason you might need to examine the weights in this manner is if you want to carry out predictions outside of BigQuery.

- Stuff out-of-needs right now: 

    You can program the weights of linear regression in python and insert it in the BigQuery model (Examining model weights, Machine Learning in BigQuery)

## More complex regression models 

A linear regression model is the simplest form of regression model. BigQuery supports dnn_regressor and xgboost models 

- Deep Neural Networks

https://cloud.google.com/bigquery-ml/docs/reference/standard-sql/bigqueryml-syntax-create-tensorflow

https://cloud.google.com/bigquery-ml/docs

DNN can be thought of an extension of linear models. The 1st layer is a linear transform and the 2nd one in nonlinear, so cool.

I really think this way is deprecated. Now you can use Tensorflow as model. Need to researh this jaleo


## Gradient bossting trees

- Decision trees are awesome. Period. They by themselves are quite shitty, but it can be improved with a boosting technique

        CREATE OR REPLACE MODEL
        `isentropic-road-260315.vargas_data_studies.xgboost_london_bicycles` TRANSFORM (* EXCEPT(start_date),
        IF
            (EXTRACT(dayofweek
            FROM
                start_date) BETWEEN 2
            AND 6,
            'weekday',
            'weekend') AS dayofweek,
            ML.BUCKETIZE(EXTRACT(HOUR
            FROM
                start_date),
            [5,
            10,
            17]) AS hourofday) OPTIONS (input_label_cols=['duration'],
            model_type='boosted_tree_regressor',
            max_tree_depth=4) AS
        SELECT
        duration,
        start_station_name,
        start_date
        FROM
        `isentropic-road-260315.vargas_data_studies.london_bicycles_cycle_hire`

