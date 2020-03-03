
### Working with the Google census dataset

- copy the dataset you want to work to a personal place

- go here 
        https://console.cloud.google.com/automl-tables/datasets?_ga=2.209126802.1840264490.1582536652-152910185.1580744498&_gac=1.245514928.1580979926.EAIaIQobChMI-cf7k8m85wIVxpTVCh1bHAuMEAAYASAAEgKDw_D_BwE&project=isentropic-road-260315

- import data from BigQuery (fill the gaps)

    - isentropic-road-260315
    - vargas_data_studies_US (because country stuff)
    - census

Once the data is imported

- select the label

- you can select additional parameters (column weights and stuff)

- Training

        Training budget 1h
        Feature columns: all columns (exactly like in BigQuery)

![alt](../pics/classification_metris.png " ")

![alt](../pics/classification_automl.png " ")

![alt](../pics/class_confusionmatrix.png " ")

![alt](../pics/classification_automl_feature.png " ")