# Drug Price Prediction

## Project Description

The objective is to predict the price for each drug in the test data set (`drugs_test.csv`).

### Approach
***The preprocessing and encoding steps***
1. Cleaned all text data and created a year column
2. Encoded all features:
   1. For high cardinality features like **dosage_form**, **route_of_administration**... created a **mean target** with cross validation
   2. **Ingredient** are encoded with mean target and for each drug then we take the average of the ingredients it contains 
   3. For **pharmacy** column, I took only the first one on the list. This choice is justified by the fact there are only 2 samples where we have multiple companies. Then it was encoded with mean target 
   4. **Percentage** column was converted to an integer between  0 and 1 
   5. Other columns to **one hot encoding**

***Models tested***
1. Tried multiple methods linear regression (LR), decision tree (DT), random forest, xgboost but ended choosing RandomForest as LR and DT were underfitting and xgboost overfitting
2. Tried ensemble, but it was worse.
3. Chose **RandomForest**

***Fine-tuning***
1. Grid search with cross validation to find the best parameters
2. Feature selection using Recursive feature selection (RFE)

**RMSE**:  **51.92** on val dataset and **41.75** on train (Still need to improve the model) -> check the ***drugs.ipynb*** in the notebooks folder
## Getting Started
Installation
------------
    $ git clone https://github.com/abdel-ely-ds/drug-price-predictions.git
    $ cd drugs-price-predictions
    $ pip install .

Usage
------------

```python
import os

import pandas as pd
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split

from drugs import Drugs
from drugs.utils.constants import DRUG_ID, PRICE, SEED

df = pd.read_csv(os.path.join("../data/", "drugs_train.csv"))
df_ingredient = pd.read_csv(os.path.join("../data/", "active_ingredients.csv"))

train_df, val_df = train_test_split(df, random_state=SEED, test_size=0.2)

train_df_ingredient = df_ingredient[df_ingredient[DRUG_ID].isin(train_df[DRUG_ID])]
val_df_ingredient = df_ingredient[df_ingredient[DRUG_ID].isin(val_df[DRUG_ID])]

drugs = Drugs()
drugs.fit(
    df=train_df,
    df_ingredient=train_df_ingredient
)
preds = drugs.predict(val_df, val_df_ingredient)[PRICE]
print(f"RMSE: {mean_squared_error(preds, val_df[PRICE], squared=False)}")
```

From CLI
------------
    $ drugs-price train --data-dir ./data/ --output-dir ./artifacts --df-filename drugs_train.csv --df-ingredient-filename active_ingredients.csv

Project Organization
------------

    ├── LICENSE            <- MIT License.
    ├── README.md          <- README of the project.
    ├── data               <- Raw data
    ├── src                <- source code for training and predicting drug prices
    ├── notebooks          <- Jupyter notebooks
    ├── requirements.txt   <- The requirements file contains all the necessary libs to run the project
    ├── tests              <- tests forlder
    └── noxfile.py         <- black, build, tests               

--------
## Files & Field Descriptions

You'll find five CSV files:
- `drugs_train.csv`: training data set,
- `drugs_test.csv`: test data set,
- `active_ingredients.csv`: active ingredients in the drugs.
- `drug_label_feature_eng.csv`: feature engineering on the text description,
- `sample_submission.csv`: the expected output for the predictions.

### Drugs

Filenames: `drugs_train.csv` and `drugs_test.csv`

| Field | Description |
| --- | --- |
| `drug_id` | Unique identifier for the drug. |
| `description` | Drug label. |
| `administrative_status` | Administrative status of the drug. |
| `marketing_status` | Marketing status of the drug. |
| `approved_for_hospital_use` | Whether the drug is approved for hospital use (`oui`, `non` or `inconnu`). |
| `reimbursement_rate` | Reimbursement rate of the drug. |
| `dosage_form` | See [dosage form](https://en.wikipedia.org/wiki/Dosage_form).|
| `route_of_administration` | Path by which the drug is taken into the body. Comma-separated when a drug has several routes of administration. See [route of administration](https://en.wikipedia.org/wiki/Route_of_administration). |
| `marketing_authorization_status` | Marketing authorization status. |
| `marketing_declaration_date` | Marketing declaration date. |
| `marketing_authorization_date` | Marketing authorization date. |
| `marketing_authorization_process` | Marketing authorization process. |
| `pharmaceutical_companies` | Companies owning a license to sell the drug. Comma-separated when several companies sell the same drug. |
| `price` | Price of the drug (i.e. the output variable to predict). |

**Note:** the `price` column only exists for the train data set.

### Active Ingredients

Filename: `active_ingredients.csv`

| Field | Description |
| --- | --- |
| `drug_id` | Unique identifier for the drug. |
| `active_ingredient` | [Active ingredient](https://en.wikipedia.org/wiki/Active_ingredient) in the drug. |

**Note:** some drugs are composed of several active ingredients.

### Text Description Feature Engineering

Filename: `drug_label_feature_eng.csv`

This file is here to help you and provide some feature engineering on the drug labels.

| Field | Description |
| --- | --- |
| `description` | Drug label. |
| `label_XXXX` | Dummy coding using the words in the drug label (e.g. `label_ampoule` = `1` if the drug label contains the word `ampoule` - vial in French). |
| `count_XXXX` | Extract the quantity from the description (e.g. `count_ampoule` = `32` if the drug label  the sequence `32 ampoules`). |

**Note:** This data has duplicate records and some descriptions in `drugs_train.csv` or `drugs_test.csv` might not be present in this file.

