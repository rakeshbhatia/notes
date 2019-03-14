
# **In Sample Evaluation and Cross Validation**


```python
import clf
import pandas as pd
import sklearn
import numpy as np
import pandas_summary
import matplotlib.pyplot as plt
from sklearn.model_selection import KFold
from sklearn.model_selection import cross_val_score
```


```python
# Helper function.
def fold_i_of_k(dataset, i, k):
    n = len(dataset)
    return dataset[n*(i-1)//k:n*i//k]
```


```python
# Grab and process the raw data.
data_path = ("https://raw.githubusercontent.com/Thinkful-Ed/data-201-resources/"
             "master/sms_spam_collection/SMSSpamCollection"
            )
sms_raw = pd.read_csv(data_path, delimiter= '\t', header=None)
sms_raw.columns = ['spam', 'message']

# Enumerate our spammy keywords.
keywords = ['click', 'offer', 'winner', 'buy', 'free', 'cash', 'urgent']

for key in keywords:
    sms_raw[str(key)] = sms_raw.message.str.contains(
        ' ' + str(key) + ' ',
        case=False
)

sms_raw['allcaps'] = sms_raw.message.str.isupper()
sms_raw['spam'] = (sms_raw['spam'] == 'spam')
data = sms_raw[keywords + ['allcaps']]
target = sms_raw['spam']

from sklearn.naive_bayes import BernoulliNB
bnb = BernoulliNB()
y_pred = bnb.fit(data, target).predict(data)
```


```python
# Test your model with different holdout groups.

from sklearn.model_selection import train_test_split
# Use train_test_split to create the necessary training and test groups
X_train, X_test, y_train, y_test = train_test_split(data, target, test_size=0.2, random_state=20)
print('With 20% Holdout: ' + str(bnb.fit(X_train, y_train).score(X_test, y_test)))
print('Testing on Sample: ' + str(bnb.fit(data, target).score(data, target)))
```

    With 20% Holdout: 0.884304932735
    Testing on Sample: 0.89160086145



```python
cross_val_score(bnb, data, target, cv=10)
```




    array([ 0.89784946,  0.89426523,  0.89426523,  0.890681  ,  0.89605735,
            0.89048474,  0.88150808,  0.89028777,  0.88489209,  0.89568345])




```python
X = data
y = target
kf = KFold(n_splits=10)
bnb = BernoulliNB()
xval_err = 0
cross_val_scores = []

for train_index, test_index in kf.split(X):
    X_train, X_test = X.iloc[train_index], X.iloc[test_index]
    y_train, y_test = y.iloc[train_index], y.iloc[test_index]
    cross_val_scores.append(bnb.fit(X_train, y_train).score(X_test, y_test))
    
print(cross_val_scores)

```

    [0.88888888888888884, 0.87634408602150538, 0.9048473967684022, 0.89587073608617596, 0.89946140035906641, 0.89946140035906641, 0.88509874326750448, 0.88330341113105926, 0.88689407540394971, 0.89587073608617596]



```python
from sklearn.metrics import confusion_matrix

c = confusion_matrix(target, y_pred)

print('sensitivity: ', c[1,1]/(c[1,0]+c[1,1]))
print('specificity: ', c[0,0]/(c[0,1]+c[0,0]))
```

    sensitivity:  0.265060240964
    specificity:  0.988601036269

