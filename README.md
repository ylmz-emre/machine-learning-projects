# Rainfall Prediction with Machine Learning

This project explores whether tomorrow’s rainfall can be predicted using today’s weather observations from the Melbourne region.

I compared two classification models:

- Random Forest
- Logistic Regression

The workflow includes data preprocessing, train/test splitting, model tuning, evaluation, and feature-importance analysis.

## Results

Random Forest performed slightly better overall:

- Accuracy: 84%
- Precision for rain: 76%
- Recall for rain: 50%

Logistic Regression achieved:

- Accuracy: 83%
- Precision for rain: 68%
- Recall for rain: 51%

The main limitation is that both models still miss about half of the actual rainy days.