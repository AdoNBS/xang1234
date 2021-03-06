
## Imbalanced Datasets with imbalanced-learn 

### 1. Introduction
Machine learning classsification algorithms tend to produce unsatisfactory results when trying to classify unbalanced datasets. The number of observations in the class of interest is very low compared to the total number of observations. Examples of applications with such datasets are customer churn identification, financial fraud identification, identification of rare diseases, detecting defects in manufacturing, etc. Classifiers give poor results on such datasets as they favor the majority class, resulting in a high missclassification rate for the minority class of interest. 

For example, when faced with a credit card fraud dataset where only 1% of transactions are fraudulent, a classification algorithm such as logistic regression would tend to predict that all transactions are legitimate, resulting in 99% accuracy. Needless to say, such an algorithm would be of little value. In real world cases, collecting more data might be an option and should be explored if possible. 

To improve the prediction, our machine learning algorithms would require a roughly equal number of observations from each class. This new dataset can be constructed by resampling our observations. [Imbalanced learn](https://github.com/scikit-learn-contrib/imbalanced-learn) is a [scikit-learn](http://scikit-learn.org/stable/) compatible package which implements various resampling methods to tackle imbalanced datasets. In this post we explore the usage of imbalanced-learn and the various resampling techniques that are implemented within the package.  


### 2. Dataset
For our study we will use the [Credit Card Fraud Detection](https://www.kaggle.com/mlg-ulb/creditcardfraud) dataset that has been made available by the [ULB Machine Learning Group](http://mlg.ulb.ac.be) on Kaggle. The dataset contains almost 285k observations. The available features are `Time`, anonymous features `V1` to `V28`(which are the results of PCA transformation) and `Amount` while the classification is given by `class`. Only 0.17% of the transactions are fraudulent. 


```python
import pandas as pd
df=pd.read_csv(r'C:\Users\david\Documents\MITB\Portfolio\creditcard.csv')
df.head()
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
      <th>Time</th>
      <th>V1</th>
      <th>V2</th>
      <th>V3</th>
      <th>V4</th>
      <th>V5</th>
      <th>V6</th>
      <th>V7</th>
      <th>V8</th>
      <th>V9</th>
      <th>...</th>
      <th>V21</th>
      <th>V22</th>
      <th>V23</th>
      <th>V24</th>
      <th>V25</th>
      <th>V26</th>
      <th>V27</th>
      <th>V28</th>
      <th>Amount</th>
      <th>Class</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>-1.359807</td>
      <td>-0.072781</td>
      <td>2.536347</td>
      <td>1.378155</td>
      <td>-0.338321</td>
      <td>0.462388</td>
      <td>0.239599</td>
      <td>0.098698</td>
      <td>0.363787</td>
      <td>...</td>
      <td>-0.018307</td>
      <td>0.277838</td>
      <td>-0.110474</td>
      <td>0.066928</td>
      <td>0.128539</td>
      <td>-0.189115</td>
      <td>0.133558</td>
      <td>-0.021053</td>
      <td>149.62</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0</td>
      <td>1.191857</td>
      <td>0.266151</td>
      <td>0.166480</td>
      <td>0.448154</td>
      <td>0.060018</td>
      <td>-0.082361</td>
      <td>-0.078803</td>
      <td>0.085102</td>
      <td>-0.255425</td>
      <td>...</td>
      <td>-0.225775</td>
      <td>-0.638672</td>
      <td>0.101288</td>
      <td>-0.339846</td>
      <td>0.167170</td>
      <td>0.125895</td>
      <td>-0.008983</td>
      <td>0.014724</td>
      <td>2.69</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>-1.358354</td>
      <td>-1.340163</td>
      <td>1.773209</td>
      <td>0.379780</td>
      <td>-0.503198</td>
      <td>1.800499</td>
      <td>0.791461</td>
      <td>0.247676</td>
      <td>-1.514654</td>
      <td>...</td>
      <td>0.247998</td>
      <td>0.771679</td>
      <td>0.909412</td>
      <td>-0.689281</td>
      <td>-0.327642</td>
      <td>-0.139097</td>
      <td>-0.055353</td>
      <td>-0.059752</td>
      <td>378.66</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>-0.966272</td>
      <td>-0.185226</td>
      <td>1.792993</td>
      <td>-0.863291</td>
      <td>-0.010309</td>
      <td>1.247203</td>
      <td>0.237609</td>
      <td>0.377436</td>
      <td>-1.387024</td>
      <td>...</td>
      <td>-0.108300</td>
      <td>0.005274</td>
      <td>-0.190321</td>
      <td>-1.175575</td>
      <td>0.647376</td>
      <td>-0.221929</td>
      <td>0.062723</td>
      <td>0.061458</td>
      <td>123.50</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.0</td>
      <td>-1.158233</td>
      <td>0.877737</td>
      <td>1.548718</td>
      <td>0.403034</td>
      <td>-0.407193</td>
      <td>0.095921</td>
      <td>0.592941</td>
      <td>-0.270533</td>
      <td>0.817739</td>
      <td>...</td>
      <td>-0.009431</td>
      <td>0.798278</td>
      <td>-0.137458</td>
      <td>0.141267</td>
      <td>-0.206010</td>
      <td>0.502292</td>
      <td>0.219422</td>
      <td>0.215153</td>
      <td>69.99</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 31 columns</p>
</div>



### 3. Exploratory Data Analysis (EDA)
Although our goals is to see how the imbalanced-learn package work, we will still need to explore the data and correct for any issues as in any machine learning pipeline. In order to speed up the process, I referred to the kaggle kernals below:  

* https://www.kaggle.com/joparga3/in-depth-skewed-data-classif-93-recall-acc-now/notebook
* https://www.kaggle.com/janiobachmann/credit-fraud-dealing-with-imbalanced-datasets

Of particular note, the `Time` and `Amount` are skewed:


```python
import seaborn as sns 
import matplotlib.pyplot as plt

fig, ax = plt.subplots(1, 2, figsize=(18,4))

amount_val = df['Amount'].values
time_val = df['Time'].values

sns.distplot(amount_val, ax=ax[0], color='r')
ax[0].set_title('Distribution of Transaction Amount', fontsize=14)
ax[0].set_xlim([min(amount_val), max(amount_val)])

sns.distplot(time_val, ax=ax[1], color='b')
ax[1].set_title('Distribution of Transaction Time', fontsize=14)
ax[1].set_xlim([min(time_val), max(time_val)])



plt.show()
```

    C:\Users\david\AppData\Local\Continuum\Anaconda3\lib\site-packages\matplotlib\axes\_axes.py:6462: UserWarning: The 'normed' kwarg is deprecated, and has been replaced by the 'density' kwarg.
      warnings.warn("The 'normed' kwarg is deprecated, and has been "
    


![png](output_4_1.png)


The transaction amount is heaviliy skewed towards small transactions and should be scaled. One option would be to use a log transformation on the data set. The transaction time seems to follow a night and day pattern with the number of transactions going down at night. As such I choose not to scale the `time` variable


```python
import numpy as np
from sklearn.preprocessing import StandardScaler, RobustScaler

df['scaled_amount'] = np.log(df['Amount']+1)
rob_scaler = RobustScaler()
#df['scaled_amount'] = rob_scaler.fit_transform(df['Amount'].values.reshape(-1,1))
df['scaled_time'] = rob_scaler.fit_transform(df['Time'].values.reshape(-1,1))

#df.drop(['Amount'], axis=1, inplace=True)
df.drop(['Time','Amount'], axis=1, inplace=True)


scale_amount_val = df['scaled_amount'].values
sns.distplot(scale_amount_val, color='r').set_title("Scaled Distribution of Transaction Amount")
plt.show()

```

    C:\Users\david\AppData\Local\Continuum\Anaconda3\lib\site-packages\matplotlib\axes\_axes.py:6462: UserWarning: The 'normed' kwarg is deprecated, and has been replaced by the 'density' kwarg.
      warnings.warn("The 'normed' kwarg is deprecated, and has been "
    


![png](output_6_1.png)



```python
scale_time_val = df['scaled_time'].values
sns.distplot(scale_time_val, color='r').set_title("Scaled Distribution of Transaction Time")
plt.show()
```

    C:\Users\david\AppData\Local\Continuum\Anaconda3\lib\site-packages\matplotlib\axes\_axes.py:6462: UserWarning: The 'normed' kwarg is deprecated, and has been replaced by the 'density' kwarg.
      warnings.warn("The 'normed' kwarg is deprecated, and has been "
    


![png](output_7_1.png)


### 4. Splitting the data into training and validation subsets 

We split the data into the training and validation datasets cross-validation with a 70:30 ratio. This is a simple hold out method. Note that the training should be done on the rebalanced/resampled training dataset while the **evaluation should be done one the original holdout validation dataset.** 


```python
from sklearn.model_selection import train_test_split
y=df['Class']
x=df.drop(['Class'], axis=1)

x_train,x_test,y_train,y_test=train_test_split(x,y,test_size=0.3, stratify=y, random_state=0)
```

### 5. Training on an unbalanced dataset 

Let us try to naively train bagging (decision tree) and random forest classifiers to our dataset. These 2 classifiers are chose for illustration purposes only and we will use `sklearn` default settings. The focus is not to find the best classifier and tune it, but to understand how to rebalanced skewed datasets. For our metric we will look at the confusion matrix of the test dataset.


```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import BaggingClassifier
from sklearn.metrics import confusion_matrix
from sklearn.model_selection import cross_val_score


def fit_model(x_train,x_test,y_train,y_test):
    classifiers={"Random Forest":RandomForestClassifier(),
                 "Bagging-Decision Tree":BaggingClassifier(),}
    fig, ax = plt.subplots(1,2,figsize=(12,6))
    i=0
 
    for key, classifier in classifiers.items():
        classifier.fit(x_train, y_train)
        y_pred=classifier.predict(x_test)
        cm=confusion_matrix(y_test, y_pred)
        sns.heatmap(cm, ax=ax[i], annot=True, 
                    cmap=plt.cm.Blues,
                   xticklabels=['No Fraud', 'Fraud'],
                   yticklabels=['No Fraud', 'Fraud']).set_title(key)

        i+=1
    plt.show()
        
```


```python
fit_model(x_train,x_test,y_train,y_test)
```


![png](output_12_0.png)


We observe that decision tree type classifiers are able to do a decent job to separate the `Fraud` and `No Fraud` classes although the classes are heavily imbalanced. This is usually not the case for imbalanced classes; the classifier typically **classifies everything as belonging to the majority class.** 

### 5. Various techniques for resampling the training dataset

Here we explore the various options to rebalance the training dataset. The methods can either:
* Under-sample the majority class
* Over-sample the minority class.
* Combine over and under-sampling.
* Create ensemble balanced sets.

#### 5.1 Undersampling the majority class

There are various algorithms implemented in imbalanced-learn that supports undersampling the majority class. They can be divided into generative and selective algorithms; generative algos try to summarize the majority class and then the samples are drawn from this generated data instead of the actual majority class observations. On the other hand, selective algorithms use various heuristics to select (or reject) samples to be drawn from the majority class. There are many undersampling algorithms and we will not run through all of them. 

##### Undersampling with Random Under Sampler
The simplest selective under-sampler is the random under sampler **`RandomUnderSampler`**; we select the majority class randomly. Bootstrapping is possible by setting the parameter `replacement` to `True`. 


```python
from imblearn.under_sampling import RandomUnderSampler
rus = RandomUnderSampler(random_state=0)
x_rus, y_rus = rus.fit_sample(x_train, y_train)
```

We also define the function `plot_dim` to display the data after t-SNE, PCA and Truncated SVD transformations. 


```python
from sklearn.manifold import TSNE
from sklearn.decomposition import PCA, TruncatedSVD
import time
import matplotlib.patches as mpatches


def plot_dim(x,y):
    # T-SNE Implementation
    t0 = time.time()
    X_reduced_tsne = TSNE(n_components=2, random_state=10).fit_transform(x)
    t1 = time.time()
    print("T-SNE took {:.2} s".format(t1 - t0))    

    # PCA Implementation
    t0 = time.time()
    X_reduced_pca = PCA(n_components=2, random_state=10).fit_transform(x)
    t1 = time.time()
    print("PCA took {:.2} s".format(t1 - t0))

    # TruncatedSVD
    t0 = time.time()
    X_reduced_svd = TruncatedSVD(n_components=2, algorithm='randomized', random_state=10).fit_transform(x)
    t1 = time.time()
    print("Truncated SVD took {:.2} s".format(t1 - t0))
    
    f, (ax1, ax2, ax3) = plt.subplots(1, 3, figsize=(24,6))
    # labels = ['No Fraud', 'Fraud']
    f.suptitle('Clusters using Dimensionality Reduction', fontsize=14)


    blue_patch = mpatches.Patch(color='#0A0AFF', label='No Fraud')
    red_patch = mpatches.Patch(color='#AF0000', label='Fraud')


    # t-SNE scatter plot
    ax1.scatter(X_reduced_tsne[:,0], X_reduced_tsne[:,1], s=4, c=(y == 0),
                cmap='coolwarm', label='No Fraud', linewidths=2)
    ax1.scatter(X_reduced_tsne[:,0], X_reduced_tsne[:,1], s=4, c=(y == 1),
                cmap='coolwarm', label='Fraud', linewidths=2)
    ax1.set_title('t-SNE', fontsize=14)

    ax1.grid(True)

    ax1.legend(handles=[blue_patch, red_patch])


    # PCA scatter plot
    ax2.scatter(X_reduced_pca[:,0], X_reduced_pca[:,1], s=4, c=(y == 0), 
                cmap='coolwarm', label='No Fraud', linewidths=2)
    ax2.scatter(X_reduced_pca[:,0], X_reduced_pca[:,1], s=4, c=(y == 1),
                cmap='coolwarm', label='Fraud', linewidths=2)
    ax2.set_title('PCA', fontsize=14)

    ax2.grid(True)

    ax2.legend(handles=[blue_patch, red_patch])

    # TruncatedSVD scatter plot
    ax3.scatter(X_reduced_svd[:,0], X_reduced_svd[:,1], s=4, c=(y == 0),
                cmap='coolwarm', label='No Fraud', linewidths=2)
    ax3.scatter(X_reduced_svd[:,0], X_reduced_svd[:,1], s=4, c=(y == 1), 
                cmap='coolwarm', label='Fraud', linewidths=2)
    ax3.set_title('Truncated SVD', fontsize=14)

    ax3.grid(True)

    ax3.legend(handles=[blue_patch, red_patch])

    plt.show()
```


```python
plot_dim(x_rus,y_rus)
```

    T-SNE took 1.7e+01 s
    PCA took 0.003 s
    Truncated SVD took 0.002 s
    


![png](output_18_1.png)


When we look at the data obtained from undersampling using dimensionality reduction, the `No Fraud` and `Fraud` classes appear separable. This is in contrast to the original imbalanced datasets where there are very few `Fraud` cases and they do not appear to be easily separable


```python
x_train_plot,x_plot,y_train_plot,y_plot=train_test_split(x,y,test_size=0.2, stratify=y, random_state=0)
plot_dim(x_plot,y_plot)
```

    T-SNE took 2.7e+03 s
    PCA took 0.24 s
    Truncated SVD took 0.17 s
    


![png](output_20_1.png)


Now that we have the used Random Undersampling to replaced the classes, we use a classifier to see how the model would perform when trained on a rebalanced dataset. 


```python
fit_model(x_rus,x_test,y_rus,y_test)
```


![png](output_22_0.png)


Although more of the `Fraud` classes are detected, this comes at an elevated False Positive rate - we will label more than 2k **No Fraud** cases as **Fraud**. When undersampling the majority or `No Fraud` class. It appears that we have oversimplified it, causing classification errors.  

##### Under Sampling with Tomek Links

The classifier detects *Tomek's Links* : this link exists if 2 samples from different classes are the nearest neighbours of each other. The sample from the majority class is then removed from the dataset.  



```python
from imblearn.under_sampling import TomekLinks
tomekl = TomekLinks(random_state=0,n_jobs=3)
x_tomekl, y_tomekl = tomekl.fit_sample(x_train, y_train)
```


```python
from sklearn.utils import resample
x_tomekl_plot,y_tomekl_plot=resample(x_tomekl,y_tomekl,n_samples=10000,random_state=0)
plot_dim(x_tomekl_plot,y_tomekl_plot)
```

    T-SNE took 3.6e+02 s
    PCA took 0.028 s
    Truncated SVD took 0.025 s
    


![png](output_26_1.png)


Even after running `TomekLink`we observe that the dataset is similar to the initial imbalanced dataset. Thus it is unsurprising to get similar classification results as shown below. 


```python
fit_model(x_tomekl,x_test,y_tomekl,y_test)
```


![png](output_28_0.png)


#### 5.2 Oversample the minority class

Instead of reducing the majority class. We can choose to oversample the minority class instead to rebalance the dataset.

##### Random Over Sampling
In Random Oversampling, the we favor the minority class when sampling to rebalance the dataset. 


```python
from imblearn.over_sampling import RandomOverSampler
ros = RandomOverSampler(ratio='minority',random_state=0)
x_ros, y_ros = ros.fit_sample(x_train, y_train)
```


```python
x_ros_plot,y_ros_plot=resample(x_ros,y_ros,n_samples=10000,random_state=0)
plot_dim(x_ros_plot,y_ros_plot)
```

    T-SNE took 3.2e+02 s
    PCA took 0.038 s
    Truncated SVD took 0.025 s
    


![png](output_31_1.png)


Random over sampling gives us a more separable dataset. We obtained slightly better results for the Random Forest classifier. 


```python
fit_model(x_ros,x_test,y_ros,y_test)
```


![png](output_33_0.png)


##### SMOTE

SMOTE or Synthetic Minority Over-Sampling Technique creates additional samples of the minority class. The parameter `k_neighbors` determines the number of neighbours of the minority class to consider when creating synthetic data. The new datapoint is created somewhere between these k neighbours. Refer to the original [article](https://www.jair.org/index.php/jair/article/view/10302) for more information


```python
from imblearn.over_sampling import SMOTE 
sm = SMOTE(random_state=0)
x_smote, y_smote = sm.fit_sample(x_train, y_train)
```


```python
x_smote_plot,y_smote_plot=resample(x_smote,y_smote,n_samples=10000,random_state=0)
plot_dim(x_smote_plot,y_smote_plot)
```

    T-SNE took 3.3e+02 s
    PCA took 0.033 s
    Truncated SVD took 0.023 s
    


![png](output_36_1.png)


We observe that the resulting dataset is less separable compared to the random over sampled dataset, especially after running `t-SNE`.


```python
fit_model(x_smote,x_test,y_smote,y_test)
```


![png](output_38_0.png)


#### 5.3 Combine oversampling and undersampling

It is also possible to combine both over sampling and undersampling to rebalance the datasets. 

##### SMOTE ENN

SMOTE ENN combines SMOTE (over sampling) and ENN or Edited Nearest Neighbours (Under Sampling)


```python
from imblearn.combine import SMOTEENN 
enn = SMOTEENN(random_state=0)
x_enn, y_enn = enn.fit_sample(x_train, y_train)
```


```python
x_enn_plot,y_enn_plot=resample(x_enn,y_enn,n_samples=10000,random_state=0)
plot_dim(x_enn_plot,y_enn_plot)
```

    T-SNE took 3.3e+02 s
    PCA took 0.034 s
    Truncated SVD took 0.023 s
    


![png](output_41_1.png)



```python
fit_model(x_enn,x_test,y_enn,y_test)
```


![png](output_42_0.png)


##### SMOTE TOMEK 

SMOTE TOMEK combines SMOTE (over sampling) and Tomek links (Under Sampling)



```python
from imblearn.combine import SMOTETomek 
stomek = SMOTETomek (random_state=0)
x_stomek, y_stomek = stomek.fit_sample(x_train, y_train)
```


```python
x_stomek_plot,y_stomek_plot=resample(x_stomek,y_stomek,n_samples=10000,random_state=0)
plot_dim(x_stomek_plot,y_stomek_plot)
```

    T-SNE took 3.2e+02 s
    PCA took 0.036 s
    Truncated SVD took 0.024 s
    


![png](output_45_1.png)



```python
fit_model(x_stomek,x_test,y_stomek,y_test)
```


![png](output_46_0.png)


Neither SMOTE ENN nor SMOTE Tomek are able to improve the classification results. 

#### 5.4 Ensemble Balancing 

##### Balanced Bagging Classifier

This is essentially a bagging classifier with additional balancing. We try it out with the default *Decision Tree* and *Support Vector Machine* classifier. 


```python
from imblearn.ensemble import BalancedBaggingClassifier 
bb=BalancedBaggingClassifier(random_state=0, n_jobs=3)
bb.fit(x_train, y_train)
```




    BalancedBaggingClassifier(base_estimator=None, bootstrap=True,
                 bootstrap_features=False, max_features=1.0, max_samples=1.0,
                 n_estimators=10, n_jobs=3, oob_score=False, random_state=0,
                 ratio='auto', replacement=False, verbose=0, warm_start=False)




```python
y_pred=bb.predict(x_test)
cm=confusion_matrix(y_test, y_pred)
sns.heatmap(cm, annot=True, 
                    cmap=plt.cm.Blues,
                   xticklabels=['No Fraud', 'Fraud'],
                   yticklabels=['No Fraud', 'Fraud']).set_title('Balanced Bagging')
```




    Text(0.5,1,'Balanced Bagging')




![png](output_50_1.png)



```python
from sklearn.svm import SVC
bbsvm=BalancedBaggingClassifier(base_estimator=SVC(),n_estimators=50,random_state=0, n_jobs=3)
bbsvm.fit(x_train, y_train)


```




    BalancedBaggingClassifier(base_estimator=SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
      decision_function_shape='ovr', degree=3, gamma='auto', kernel='rbf',
      max_iter=-1, probability=False, random_state=None, shrinking=True,
      tol=0.001, verbose=False),
                 bootstrap=True, bootstrap_features=False, max_features=1.0,
                 max_samples=1.0, n_estimators=50, n_jobs=3, oob_score=False,
                 random_state=0, ratio='auto', replacement=False, verbose=0,
                 warm_start=False)




```python
y_pred=bbsvm.predict(x_test)
cm=confusion_matrix(y_test, y_pred)
sns.heatmap(cm, annot=True, 
                    cmap=plt.cm.Blues,
                   xticklabels=['No Fraud', 'Fraud'],
                   yticklabels=['No Fraud', 'Fraud']).set_title('Balanced Bagging - SVM')
```




    Text(0.5,1,'Balanced Bagging - SVM')




![png](output_52_1.png)


The Balanced Bagging SVM classifier is able to detect the highest amount of `Fraud`, at the cost of high false positives for `No Fraud`

### Conclusion

We have run through the various options in `imbalanced-learn` to rebalance and imbalanced dataset. However the original imbalanced dataset can already be classified relatively well. As such it is more difficult to illustrate the full benefits of rebalancing the dataset.

This post was written as a Jupyter Notebook. Click [here]() to view it
