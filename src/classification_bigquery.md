# Building a classification model for bigquery

https://cloud.google.com/bigquery-ml/docs/logistic-regression-prediction

- Step one: Create a dataset to store your model.
The first step is to create a BigQuery dataset to store your model.

- Step two: Examine your data.
In this step, examine the dataset and identify which columns to use as training data for your logistic regression model.

- Step three: Select your training data.
The next step is to prepare the data you use to train your binary logistic regression model by running a query against the census_adult_income table. This step identifies the relevant features and stores them in a view for later queries to use as input data.

- Step four: Create a logistic regression model.
In this step, use the CREATE MODEL statement to create your logistic regression model.

- Step five: Use the ML.EVALUATE function to evaluate your model.
Then, use the ML.EVALUATE function to provide statistics about model performance.

- Step six: Use the ML.PREDICT function to predict a participant's income.
Finally, you use the ML.PREDICT function to predict the income bracket for a given set of census participants.


- Step two: Examine your data
The next step is to examine the dataset and identify which columns to use as training data for your logistic regression model. You can use a standard SQL query to return rows from the dataset.

The following query returns 100 rows from the US Census Dataset:

        SELECT
        *
        FROM
        `bigquery-public-data.ml_datasets.census_adult_income`
        LIMIT
        100;

The query results show that the income_bracket column in the census_adult_income table has only one of two values: <=50K or >50K. It also shows that the columns education and education_num in the census_adult_income table express the same data in different formats. The functional_weight column is the number of individuals that the Census Organizations believes a particular row represents; the values of this column appear unrelated to the value of income_bracket for a particular row.

- Step three: Select your training data

Next, you select the data used to train your logistic regression model. In this tutorial, you predict census respondent income based on the following attributes:

    - Age
    - Type of work performed
    - Country of origin
    - Marital status
    - Level of education
    - Occupation
    - Race
    - Hours worked per week
    - The following query creates a view that compiles your training data. This view is included in your CREATE MODEL statement later in this tutorial.

        CREATE OR REPLACE VIEW
        `isentropic-road-260315.vargas_data_studies_US.census` AS
        SELECT
        age,
        workclass,
        native_country,
        marital_status,
        education_num,
        occupation,
        race,
        hours_per_week,
        income_bracket,
        CASE
            WHEN MOD(functional_weight, 10) < 8 THEN 'training'
            WHEN MOD(functional_weight, 10) = 8 THEN 'evaluation'
            WHEN MOD(functional_weight, 10) = 9 THEN 'prediction'
        END AS dataframe
        FROM
        `bigquery-public-data.ml_datasets.census_adult_income`

# Step four: Create a logistic regression model

Now that you have examined your training data, the next step is to create a logistic regression model using the data.

You can create and train a logistic regression model using the CREATE MODEL statement with the option 'LOGISTIC_REG'. The following query uses a CREATE MODEL statement to train a new binary logistic regression model on the view from the previous query.



        CREATE OR REPLACE MODEL
        `isentropic-road-260315.vargas_data_studies_US.classification_census`
        OPTIONS
        ( model_type='LOGISTIC_REG',
            auto_class_weights=TRUE,
            input_label_cols=['income_bracket']
        ) AS
        SELECT
        *
        FROM
        `isentropic-road-260315.vargas_data_studies_US.census`
        WHERE
        dataframe = 'training'

![alt](../pics/classification_metris.png "")

### Query details

- The CREATE MODEL statement trains a model using the training data in the SELECT statement.

- The OPTIONS clause specifies the model type and training options. Here, the LOGISTIC_REG option specifies a logistic regression model type. It is not necessary to specify a binary logistic regression model versus a multiclass logistic regression model: BigQuery ML can determine which to train based on the number of unique values in the label column.

- The input_label_cols option specifies which column in the SELECT statement to use as the label column. Here, the label column is income_bracket, so the model will learn which of the two values of income_bracket is most likely based on the other values present in each row.

- The 'auto_class_weights=TRUE' option balances the class labels in the training data. By default, the training data is unweighted. If the labels in the training data are imbalanced, the model may learn to predict the most popular class of labels more heavily. In this case, most of the respondents in the dataset are in the lower income bracket. This may lead to a model that predicts the lower income bracket too heavily. Class weights balance the class labels by calculating the weights for each class in inverse proportion to the frequency of that class.

- The SELECT statement queries the view from Step 2. This view contains only the columns containing feature data for training the model. The WHERE clause filters the rows in input_view so that only those rows belonging to the training dataframe are included in the training data.

# Run the create model query

To run the query that creates your logistic regression model, complete the following steps:

1. In the BigQuery web UI, click the Compose new query button.

2. Enter the following standard SQL query in the Query editor text area:

        CREATE OR REPLACE MODEL
        `isentropic-road-260315.vargas_data_studies_US.classification_15iterations_census`
        OPTIONS
        ( model_type='LOGISTIC_REG',
            auto_class_weights=TRUE,
            input_label_cols=['income_bracket'],
            max_iterations=15)
        AS
        SELECT
        *
        FROM
        `isentropic-road-260315.vargas_data_studies_US.census`
        WHERE
        dataframe = 'training'
        
# Step five: Use the ML.EVALUATE function to evaluate your model

After creating your model, evaluate the performance of the model using the ML.EVALUATE function. The ML.EVALUATE function evaluates the predicted values against the actual data.

The query to evaluate the model is as follows:

        SELECT
        *
        FROM
        ML.EVALUATE (MODEL `isentropic-road-260315.vargas_data_studies_US.classification_15iterations_census`,
            (
            SELECT
            *
            FROM
            `isentropic-road-260315.vargas_data_studies_US.census`
            WHERE
            dataframe = 'evaluation' ) )

- we'll get the same stuff that we can see in the model created before

![alt](../pics/evaluate.png " ")

- (Optional) To set the processing location, click More > Query settings. For Processing location, choose US. This step is optional because the processing location is automatically detected based on the dataset's location.


# Step six: Use the ML.PREDICT function to predict income bracket

To identify the income bracket to which a particular respondent belongs, use the ML.PREDICT function. The following query predicts the income bracket of every respondent in the prediction dataframe.

        SELECT
        *
        FROM
        ML.PREDICT (MODEL `isentropic-road-260315.vargas_data_studies_US.classification_15iterations_census`,
            (
            SELECT
            *
            FROM
            `isentropic-road-260315.vargas_data_studies_US.census`
            WHERE
            dataframe = 'prediction' ) )

### Query details

The ML.PREDICT function predicts results using your model and the data from input_view, filtered to include only rows in the 'prediction' dataframe. The top-most SELECT statement retrieves the output of the ML.PREDICT function.

![alt](../pics/predictionclass.png " ")