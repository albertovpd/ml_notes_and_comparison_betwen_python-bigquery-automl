# Compare BigQuery, AutoML and Python in a regular classification model

- BigQuery


![alt](./pics/evaluate.png " ")

- AutoML

![alt](./pics/classification_metris.png " ")

- Python

![alt](./pics/classifiermetricspython1.png "")

![alt](./pics/classifiermetricspython2.png "")

# Conclusions:

We will be working with known datasets, that will be growing with time, but the type of columns will remain the same. From this background, this are my thoughts:

- A process in AutoML lasts way too much in comparison with the other tools, and the most important point, it's way too expensive. Around 20€/hour. It also gives the worst metrics.

- The best metrics are for Python, of course, where the cleaning can be extremely performed.

- BigQuery has not the best metrics, the cleaning can be quite messy sometimes, nevertheless it can be integrated in a periodic table and linked to data studio, for regular cool plots.

## Metrics

- ROC Curves summarize the trade-off between the true positive rate and false positive rate for a predictive model using different probability thresholds.
- Precision-Recall curves summarize the trade-off between the true positive rate and the positive predictive value for a predictive model using different probability thresholds.
- ROC curves are appropriate when the observations are balanced between each class, whereas precision-recall curves are appropriate for imbalanced datasets.
https://machinelearningmastery.com/roc-curves-and-precision-recall-curves-for-classification-in-python/