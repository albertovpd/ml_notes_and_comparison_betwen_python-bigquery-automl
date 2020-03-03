# Evaluation: Python? BigQuery Machine Learning Tools or Google AutoML APIs?

## The process:

### Python

- Link BigQuery with Google Colab
- Feature engineering (selecting features, cleaning and transforming columns...Etc)
- Manual GridSearch to find the best algorithm with the selected data



### BigQuery

- Use the selected columns from the python feature engineering

### AutoML Google Cloud

- Import the dataset we want to analyze
- Select the label
- Select the same columns than in the other studies, but without any preparation


# Metrics (RMSE)

- Python

![alt](./pics/python.png "")

- BigQuery

![alt](./pics/ml_bigquery.png "")

![alt](./pics/conversionprediction.png "")

![alt](./pics/bq-predictions-regression.png "")
From here we can say groping by month we can have great results in average. That's the key of this picture. It is not that good in short-term (and of course we need an increasing volume of data)

- AutoML

![alt](./pics/automl.png "")


# Correlation

- Python

![alt](./pics/python_correlation.png "")

- Google AutoML

![alt](./pics/automl_feature_importance.png "")
We can find resemblances between 2 easily, nevertheless they have different algorightms

- BigQuery
It's a coñazo to find correlations in BigQuery. You need to select the label, and 1 by 1, make a visualization in google data studio, or performing a query of 2 columns, to get a metric. (way better the python or automl way)

# To sum up:


- We need MORE metrics, ROC, AUC of ALL systems before being confident with them (RMSE it's just a measure of many), and draws it

- We need a deeper cleaning of data before using it (I can clean all perfectly in Python and save it like a new dataset in BigQuery. In that way I'm sure Bigquery ML and AutoML will improve exponentially)

- In the end everything must be imported to google data studio, to have beautiful pony dashboards


# Conclusions. What do we do with this

- Cleaning with Python is the best, by far (not necessary with google colab). So what i think could be the best is to automate the cleaning with Python (in colab, or pipelines), and export the clean datasets again to the Google Environment, for ML, dashboards, etc.

- Can we do all cleaning of the dataset in Google Colab (Python), and after it, import to BigQuery the clean dataset?

    I think no, and in the end it makes no sense. It makes sense to import to Data Studio

- Can we clean and do the Machine Learning thing in Colab and link the results to google data studio? (for better visualizations)

    I'm sure you can export the dataset to wherever from Colab. I know how to export it locally, I don't know how to export it to the Google environment. But I'm sure it can be done (Alex, help)

- Can we create a cleaning functions pipeline to automatize the process of cleaning the dataset in BigQuery?

    I know perfectly how to pipeline for whatever purpose with Python, or Google Colab and Python scripts. To pipelining with BigQuery we should use dataflow or whatever tool involved. 
    
    - I don't have idea at the moment of how to do it (but I can write the cleaning functions in Python).
    - We can also program queries to clean and link directly to google data studio, and that's like a pipeline in the end. So, this is solved

So, at 28.02, glorious andalusian day, I think the challenges are:

- How to pipeline in bigquery
- How to program queries
- How to draw beautiful dashboards from Google Colab (how to export the results to Data Studio)


