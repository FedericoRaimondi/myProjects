

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import StratifiedKFold
from sklearn.metrics import roc_auc_score
from sklearn.preprocessing import StandardScaler
from sklearn.feature_selection import VarianceThreshold
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis, LinearDiscriminantAnalysis
from sklearn.pipeline import Pipeline
from tqdm import tqdm_notebook
import warnings
import multiprocessing
from scipy.optimize import minimize  
import time
warnings.filterwarnings('ignore')
# STEP 2
train = pd.read_csv('../input/train.csv')
test = pd.read_csv('../input/test.csv')
cols = [c for c in train.columns if c not in ['id', 'target', 'wheezy-copper-turtle-magic']]
print(train.shape, test.shape)
# STEP 3
oof = np.zeros(len(train))
preds = np.zeros(len(test))

for i in tqdm_notebook(range(512)):

    train2 = train[train['wheezy-copper-turtle-magic']==i]
    test2 = test[test['wheezy-copper-turtle-magic']==i]
    idx1 = train2.index; idx2 = test2.index
    train2.reset_index(drop=True,inplace=True)

    data = pd.concat([pd.DataFrame(train2[cols]), pd.DataFrame(test2[cols])])
    pipe = Pipeline([('vt', VarianceThreshold(threshold=2)), ('scaler', StandardScaler())])
    data2 = pipe.fit_transform(data[cols])
    train3 = data2[:train2.shape[0]]; test3 = data2[train2.shape[0]:]

    skf = StratifiedKFold(n_splits=11, random_state=42)
    for train_index, test_index in skf.split(train2, train2['target']):

        clf = QuadraticDiscriminantAnalysis(0.5)
        clf.fit(train3[train_index,:],train2.loc[train_index]['target'])
        oof[idx1[test_index]] = clf.predict_proba(train3[test_index,:])[:,1]
        preds[idx2] += clf.predict_proba(test3)[:,1] / skf.n_splits

auc = roc_auc_score(train['target'], oof)
print(f'AUC: {auc:.5}')

# STEP 4
for itr in range(4):
    test['target'] = preds
    test.loc[test['target'] > 0.955, 'target'] = 1 # initial 94
    test.loc[test['target'] < 0.045, 'target'] = 0 # initial 06
    usefull_test = test[(test['target'] == 1) | (test['target'] == 0)]
    new_train = pd.concat([train, usefull_test]).reset_index(drop=True)
    print(usefull_test.shape[0], "Test Records added for iteration : ", itr)
    new_train.loc[oof > 0.995, 'target'] = 1 # initial 98
    new_train.loc[oof < 0.005, 'target'] = 0 # initial 02
    oof2 = np.zeros(len(train))
    preds = np.zeros(len(test))
    for i in tqdm_notebook(range(512)):

        train2 = new_train[new_train['wheezy-copper-turtle-magic']==i]
        test2 = test[test['wheezy-copper-turtle-magic']==i]
        idx1 = train[train['wheezy-copper-turtle-magic']==i].index
        idx2 = test2.index
        train2.reset_index(drop=True,inplace=True)

        data = pd.concat([pd.DataFrame(train2[cols]), pd.DataFrame(test2[cols])])
        pipe = Pipeline([('vt', VarianceThreshold(threshold=2)), ('scaler', StandardScaler())])
        data2 = pipe.fit_transform(data[cols])
        train3 = data2[:train2.shape[0]]
        test3 = data2[train2.shape[0]:]

        skf = StratifiedKFold(n_splits=11, random_state=time.time)
        for train_index, test_index in skf.split(train2, train2['target']):
            oof_test_index = [t for t in test_index if t < len(idx1)]
            
            clf = QuadraticDiscriminantAnalysis(0.5)
            clf.fit(train3[train_index,:],train2.loc[train_index]['target'])
            if len(oof_test_index) > 0:
                oof2[idx1[oof_test_index]] = clf.predict_proba(train3[oof_test_index,:])[:,1]
            preds[idx2] += clf.predict_proba(test3)[:,1] / skf.n_splits
    auc = roc_auc_score(train['target'], oof2)
    print(f'AUC: {auc:.5}')
    
# STEP 5
sub1 = pd.read_csv('../input/sample_submission.csv')
sub1['target'] = preds
# sub.to_csv('submission.csv',index=False)
```

    (262144, 258) (131073, 257)
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.9649
    112714 Test Records added for iteration :  0
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97154
    119378 Test Records added for iteration :  1
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97074
    119788 Test Records added for iteration :  2
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97068
    119865 Test Records added for iteration :  3
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97068
    


```python
import numpy as np
import pandas as pd
from sklearn.model_selection import StratifiedKFold
from sklearn.metrics import roc_auc_score
from sklearn.preprocessing import StandardScaler
from sklearn.feature_selection import VarianceThreshold
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis, LinearDiscriminantAnalysis
from sklearn.pipeline import Pipeline
from tqdm import tqdm_notebook
import warnings
import multiprocessing
from scipy.optimize import minimize  
warnings.filterwarnings('ignore')

train = pd.read_csv('../input/train.csv')
test = pd.read_csv('../input/test.csv')
cols = [c for c in train.columns if c not in ['id', 'target', 'wheezy-copper-turtle-magic']]
print(train.shape, test.shape)


oof = np.zeros(len(train))
preds = np.zeros(len(test))

for i in tqdm_notebook(range(512)):

    train2 = train[train['wheezy-copper-turtle-magic']==i]
    test2 = test[test['wheezy-copper-turtle-magic']==i]
    idx1 = train2.index; idx2 = test2.index
    train2.reset_index(drop=True,inplace=True)

    data = pd.concat([pd.DataFrame(train2[cols]), pd.DataFrame(test2[cols])])
    pipe = Pipeline([('vt', VarianceThreshold(threshold=2)), ('scaler', StandardScaler())])
    data2 = pipe.fit_transform(data[cols])
    train3 = data2[:train2.shape[0]]; test3 = data2[train2.shape[0]:]

    skf = StratifiedKFold(n_splits=11, random_state=42)
    for train_index, test_index in skf.split(train2, train2['target']):

        clf = QuadraticDiscriminantAnalysis(0.5)
        clf.fit(train3[train_index,:],train2.loc[train_index]['target'])
        oof[idx1[test_index]] = clf.predict_proba(train3[test_index,:])[:,1]
        preds[idx2] += clf.predict_proba(test3)[:,1] / skf.n_splits

auc = roc_auc_score(train['target'], oof)
print(f'AUC: {auc:.5}')

for itr in range(4):
    test['target'] = preds
    test.loc[test['target'] > 0.94, 'target'] = 1
    test.loc[test['target'] < 0.06, 'target'] = 0
    usefull_test = test[(test['target'] == 1) | (test['target'] == 0)]
    new_train = pd.concat([train, usefull_test]).reset_index(drop=True)
    print(usefull_test.shape[0], "Test Records added for iteration : ", itr)
    new_train.loc[oof > 0.98, 'target'] = 1
    new_train.loc[oof < 0.02, 'target'] = 0
    oof2 = np.zeros(len(train))
    preds = np.zeros(len(test))
    for i in tqdm_notebook(range(512)):

        train2 = new_train[new_train['wheezy-copper-turtle-magic']==i]
        test2 = test[test['wheezy-copper-turtle-magic']==i]
        idx1 = train[train['wheezy-copper-turtle-magic']==i].index
        idx2 = test2.index
        train2.reset_index(drop=True,inplace=True)

        data = pd.concat([pd.DataFrame(train2[cols]), pd.DataFrame(test2[cols])])
        pipe = Pipeline([('vt', VarianceThreshold(threshold=2)), ('scaler', StandardScaler())])
        data2 = pipe.fit_transform(data[cols])
        train3 = data2[:train2.shape[0]]
        test3 = data2[train2.shape[0]:]

        skf = StratifiedKFold(n_splits=11, random_state=42)
        for train_index, test_index in skf.split(train2, train2['target']):
            oof_test_index = [t for t in test_index if t < len(idx1)]
            
            clf = QuadraticDiscriminantAnalysis(0.5)
            clf.fit(train3[train_index,:],train2.loc[train_index]['target'])
            if len(oof_test_index) > 0:
                oof2[idx1[oof_test_index]] = clf.predict_proba(train3[oof_test_index,:])[:,1]
            preds[idx2] += clf.predict_proba(test3)[:,1] / skf.n_splits
    auc = roc_auc_score(train['target'], oof2)
    print(f'AUC: {auc:.5}')

sub2 = pd.read_csv('../input/sample_submission.csv')
sub2['target'] = preds
# sub.to_csv('submission.csv',index=False)
```

    (262144, 258) (131073, 257)
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.9649
    114545 Test Records added for iteration :  0
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97178
    120834 Test Records added for iteration :  1
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97094
    121313 Test Records added for iteration :  2
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97087
    121394 Test Records added for iteration :  3
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97086
    


```python
import numpy as np
import pandas as pd
from sklearn.model_selection import StratifiedKFold
from sklearn.metrics import roc_auc_score
from sklearn.preprocessing import StandardScaler
from sklearn.feature_selection import VarianceThreshold
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis, LinearDiscriminantAnalysis
from sklearn.pipeline import Pipeline
from tqdm import tqdm_notebook
import warnings
import multiprocessing
from scipy.optimize import minimize  
import time
from sklearn.model_selection import GridSearchCV, train_test_split

warnings.filterwarnings('ignore')
# STEP 2
train = pd.read_csv('../input/train.csv')
test = pd.read_csv('../input/test.csv')
cols = [c for c in train.columns if c not in ['id', 'target', 'wheezy-copper-turtle-magic']]
print(train.shape, test.shape)
# STEP 3
oof = np.zeros(len(train))
preds = np.zeros(len(test))
params = [{'reg_param': [0.1, 0.2, 0.3, 0.4, 0.5]}]

# 512 models
reg_params = np.zeros(512)
for i in tqdm_notebook(range(512)):

    train2 = train[train['wheezy-copper-turtle-magic']==i]
    test2 = test[test['wheezy-copper-turtle-magic']==i]
    idx1 = train2.index; idx2 = test2.index
    train2.reset_index(drop=True,inplace=True)

    data = pd.concat([pd.DataFrame(train2[cols]), pd.DataFrame(test2[cols])])
    pipe = Pipeline([('vt', VarianceThreshold(threshold=2)), ('scaler', StandardScaler())])
    data2 = pipe.fit_transform(data[cols])
    train3 = data2[:train2.shape[0]]; test3 = data2[train2.shape[0]:]

    skf = StratifiedKFold(n_splits=11, random_state=42)
    for train_index, test_index in skf.split(train2, train2['target']):

        qda = QuadraticDiscriminantAnalysis()
        clf = GridSearchCV(qda, params, cv=4)
        clf.fit(train3[train_index,:],train2.loc[train_index]['target'])
        reg_params[i] = clf.best_params_['reg_param']
        oof[idx1[test_index]] = clf.predict_proba(train3[test_index,:])[:,1]
        preds[idx2] += clf.predict_proba(test3)[:,1] / skf.n_splits

auc = roc_auc_score(train['target'], oof)
print(f'AUC: {auc:.5}')

# STEP 4
for itr in range(10):
    test['target'] = preds
    test.loc[test['target'] > 0.955, 'target'] = 1 # initial 94
    test.loc[test['target'] < 0.045, 'target'] = 0 # initial 06
    usefull_test = test[(test['target'] == 1) | (test['target'] == 0)]
    new_train = pd.concat([train, usefull_test]).reset_index(drop=True)
    print(usefull_test.shape[0], "Test Records added for iteration : ", itr)
    new_train.loc[oof > 0.995, 'target'] = 1 # initial 98
    new_train.loc[oof < 0.005, 'target'] = 0 # initial 02
    oof2 = np.zeros(len(train))
    preds = np.zeros(len(test))
    for i in tqdm_notebook(range(512)):

        train2 = new_train[new_train['wheezy-copper-turtle-magic']==i]
        test2 = test[test['wheezy-copper-turtle-magic']==i]
        idx1 = train[train['wheezy-copper-turtle-magic']==i].index
        idx2 = test2.index
        train2.reset_index(drop=True,inplace=True)

        data = pd.concat([pd.DataFrame(train2[cols]), pd.DataFrame(test2[cols])])
        pipe = Pipeline([('vt', VarianceThreshold(threshold=2)), ('scaler', StandardScaler())])
        data2 = pipe.fit_transform(data[cols])
        train3 = data2[:train2.shape[0]]
        test3 = data2[train2.shape[0]:]

        skf = StratifiedKFold(n_splits=11, random_state=time.time)
        for train_index, test_index in skf.split(train2, train2['target']):
            oof_test_index = [t for t in test_index if t < len(idx1)]
            
            clf = QuadraticDiscriminantAnalysis(reg_params[i])
            clf.fit(train3[train_index,:],train2.loc[train_index]['target'])
            if len(oof_test_index) > 0:
                oof2[idx1[oof_test_index]] = clf.predict_proba(train3[oof_test_index,:])[:,1]
            preds[idx2] += clf.predict_proba(test3)[:,1] / skf.n_splits
    auc = roc_auc_score(train['target'], oof2)
    print(f'AUC: {auc:.5}')
    
# STEP 5
sub3 = pd.read_csv('../input/sample_submission.csv')
sub3['target'] = preds
# sub.to_csv('submission.csv',index=False)
```

    (262144, 258) (131073, 257)
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.96462
    102002 Test Records added for iteration :  0
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97112
    117709 Test Records added for iteration :  1
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97045
    118766 Test Records added for iteration :  2
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    118916 Test Records added for iteration :  3
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97042
    118943 Test Records added for iteration :  4
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    118947 Test Records added for iteration :  5
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    118947 Test Records added for iteration :  6
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    118947 Test Records added for iteration :  7
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    118947 Test Records added for iteration :  8
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    118947 Test Records added for iteration :  9
    


    HBox(children=(IntProgress(value=0, max=512), HTML(value='')))


    
    AUC: 0.97043
    


```python
sub1.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1c13f2701648e0b0d46d8a2a5a131a53</td>
      <td>9.999999e-01</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ba88c155ba898fc8b5099893036ef205</td>
      <td>9.981074e-01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7cbab5cea99169139e7e6d8ff74ebb77</td>
      <td>2.744422e-09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ca820ad57809f62eb7b4d13f5d4371a0</td>
      <td>5.646675e-03</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7baaf361537fbd8a1aaa2c97a6d4ccc7</td>
      <td>2.129067e-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1c13f2701648e0b0d46d8a2a5a131a53</td>
      <td>1.000000e+00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ba88c155ba898fc8b5099893036ef205</td>
      <td>9.983874e-01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7cbab5cea99169139e7e6d8ff74ebb77</td>
      <td>1.188378e-09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ca820ad57809f62eb7b4d13f5d4371a0</td>
      <td>3.369092e-03</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7baaf361537fbd8a1aaa2c97a6d4ccc7</td>
      <td>1.850225e-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub3.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1c13f2701648e0b0d46d8a2a5a131a53</td>
      <td>9.999999e-01</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ba88c155ba898fc8b5099893036ef205</td>
      <td>9.983181e-01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7cbab5cea99169139e7e6d8ff74ebb77</td>
      <td>5.886219e-09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ca820ad57809f62eb7b4d13f5d4371a0</td>
      <td>3.949294e-03</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7baaf361537fbd8a1aaa2c97a6d4ccc7</td>
      <td>1.374365e-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub = pd.read_csv('../input/sample_submission.csv')
sub.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1c13f2701648e0b0d46d8a2a5a131a53</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ba88c155ba898fc8b5099893036ef205</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7cbab5cea99169139e7e6d8ff74ebb77</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ca820ad57809f62eb7b4d13f5d4371a0</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7baaf361537fbd8a1aaa2c97a6d4ccc7</td>
      <td>0.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub['target'] = 0.5*sub1.target + 0.3*sub2.target + 0.2*sub3.target
sub.to_csv('submission1.csv', index = False)
```


```python
sub['target'] = 0.2*sub1.target + 0.3*sub2.target + 0.5*sub3.target
sub.to_csv('submission2.csv', index = False)
```


```python
sub['target'] = 0.2*sub1.target + 0.4*sub2.target + 0.4*sub3.target
sub.to_csv('submission3.csv', index = False)
```
