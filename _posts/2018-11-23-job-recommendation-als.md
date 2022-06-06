---
title: Job Recommendation Engine with Implicit Feedback
excerpt: Trying out recommendation engine implementing Alternating Least Squares(ALS), a very popular algorithm for making recommendations.
published: true
date: 2018-11-23
categories: project
tags: intern jupyter-notebook recommendation-system data-science
---

**In [1]:**

{% highlight python %}
import sys
import pandas as pd
import numpy as np
import scipy.sparse as sparse
from scipy.sparse.linalg import spsolve
import random

from sklearn.preprocessing import MinMaxScaler

import implicit # The Cython library
{% endhighlight %}
 
## Load dataset 

**In [2]:**

{% highlight python %}
!ls ../jobs/data/*.tsv
{% endhighlight %}

    ../jobs/data/apps.tsv	     ../jobs/data/user_history.tsv
    ../jobs/data/jobs.tsv	     ../jobs/data/users.tsv
    ../jobs/data/test_users.tsv  ../jobs/data/window_dates.tsv


**In [3]:**

{% highlight python %}
path = '../jobs/data/'
users = pd.read_csv(path+'users.tsv', sep='\t', encoding='utf-8')
jobs = pd.read_csv(path+'jobs.tsv', sep='\t', encoding='utf-8', error_bad_lines=False)
apps = pd.read_csv(path+'apps.tsv', sep='\t', encoding='utf-8')
user_history = pd.read_csv(path+'user_history.tsv', sep='\t', encoding='utf-8')
test_users = pd.read_csv(path+'test_users.tsv', sep='\t', encoding='utf-8')
{% endhighlight %}

    b'Skipping line 122433: expected 11 fields, saw 12\n'
    b'Skipping line 602576: expected 11 fields, saw 12\n'
    b'Skipping line 990950: expected 11 fields, saw 12\n'
    /home/hectoryee/anaconda3/lib/python3.6/site-packages/IPython/core/interactiveshell.py:2785: DtypeWarning: Columns (8) have mixed types. Specify dtype option on import or set low_memory=False.
      interactivity=interactivity, compiler=compiler, result=result)


**In [4]:**

{% highlight python %}
jobs.replace('NaN',np.NaN)
jobs.head()
{% endhighlight %}




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
      <th>JobID</th>
      <th>WindowID</th>
      <th>Title</th>
      <th>Description</th>
      <th>Requirements</th>
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>Zip5</th>
      <th>StartDate</th>
      <th>EndDate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Security Engineer/Technical Lead</td>
      <td>&lt;p&gt;Security Clearance Required:&amp;nbsp; Top Secr...</td>
      <td>&lt;p&gt;SKILL SET&lt;/p&gt;\r&lt;p&gt;&amp;nbsp;&lt;/p&gt;\r&lt;p&gt;Network Se...</td>
      <td>Washington</td>
      <td>DC</td>
      <td>US</td>
      <td>20531</td>
      <td>2012-03-07 13:17:01.643</td>
      <td>2012-04-06 23:59:59</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>1</td>
      <td>SAP Business Analyst / WM</td>
      <td>&lt;strong&gt;NO Corp. to Corp resumes&amp;nbsp;are bein...</td>
      <td>&lt;p&gt;&lt;b&gt;WHAT YOU NEED: &lt;/b&gt;&lt;/p&gt;\r&lt;p&gt;Four year co...</td>
      <td>Charlotte</td>
      <td>NC</td>
      <td>US</td>
      <td>28217</td>
      <td>2012-03-21 02:03:44.137</td>
      <td>2012-04-20 23:59:59</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7</td>
      <td>1</td>
      <td>P/T HUMAN RESOURCES ASSISTANT</td>
      <td>&lt;b&gt;    &lt;b&gt; P/T HUMAN RESOURCES ASSISTANT&lt;/b&gt; &lt;...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Winter Park</td>
      <td>FL</td>
      <td>US</td>
      <td>32792</td>
      <td>2012-03-02 16:36:55.447</td>
      <td>2012-04-01 23:59:59</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>1</td>
      <td>Route Delivery Drivers</td>
      <td>CITY BEVERAGES Come to work for the best in th...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
      <td>NaN</td>
      <td>2012-03-03 09:01:10.077</td>
      <td>2012-04-02 23:59:59</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9</td>
      <td>1</td>
      <td>Housekeeping</td>
      <td>I make  sure every part of their day is magica...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
      <td>NaN</td>
      <td>2012-03-03 09:01:11.88</td>
      <td>2012-04-02 23:59:59</td>
    </tr>
  </tbody>
</table>
</div>


 
# Preprocessing 
 
## Splitting into Training and Testing dataset

currently not using this 

**In [5]:**

{% highlight python %}
users_training = users.loc[users['Split'] == 'Train']
users_testing = users.loc[users['Split'] == 'Test']

apps_training = apps.loc[apps['Split'] == 'Train']
apps_testing = apps.loc[apps['Split'] == 'Test']

user_history_training = user_history.loc[user_history['Split'] == 'Train']
user_history_testing = user_history.loc[user_history['Split'] == 'Test']
{% endhighlight %}
 
### Dataframes
- users_training
- users_testing
- apps_training
- apps_testing
- user_history_training
- user_history_testing 

**In [6]:**

{% highlight python %}
jobs_US = jobs.loc[jobs['Country'] == 'US']
jobs_US_base_line = jobs_US.iloc[0:10000,0:8]
{% endhighlight %}

**In [7]:**

{% highlight python %}
jobs_US_base_line['Title'] = jobs_US_base_line['Title'].fillna('')
jobs_US_base_line['Description'] = jobs_US_base_line['Description'].fillna('')
#jobs_US_base_line['Requirements'] = jobs_US_base_line['Requirements'].fillna('')

jobs_US_base_line['Description'] = jobs_US_base_line['Title'] + jobs_US_base_line['Description']
{% endhighlight %}
 
**From here onwards use `jobs_US_base_line` data frame to work on, which is
selected by `jobs_US.iloc[0:10000,0:8]`.** 
 
## Clean html 

**In [8]:**

{% highlight python %}
import re

def preprocessor(text):
    text = text.replace('\\r', '').replace('&nbsp', '').replace('\n', '')
    text = re.sub('<[^>]*>', '', text)
    emoticons = re.findall('(?::|;|=)(?:-)?(?:\)|\(|D|P)', text)
    text = re.sub('[\W]+', ' ', text.lower()) +\
        ' '.join(emoticons).replace('-', '')
    return text
{% endhighlight %}

**In [9]:**

{% highlight python %}
jobs_US_base_line['Description'] = jobs_US_base_line['Description'].astype(dtype='str').apply(preprocessor)
{% endhighlight %}

**In [10]:**

{% highlight python %}
apps.head()
{% endhighlight %}




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
      <th>UserID</th>
      <th>WindowID</th>
      <th>Split</th>
      <th>ApplicationDate</th>
      <th>JobID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2012-04-04 15:56:23.537</td>
      <td>169528</td>
    </tr>
    <tr>
      <th>1</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2012-04-06 01:03:00.003</td>
      <td>284009</td>
    </tr>
    <tr>
      <th>2</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2012-04-05 02:40:27.753</td>
      <td>2121</td>
    </tr>
    <tr>
      <th>3</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2012-04-05 02:37:02.673</td>
      <td>848187</td>
    </tr>
    <tr>
      <th>4</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2012-04-05 22:44:06.653</td>
      <td>733748</td>
    </tr>
  </tbody>
</table>
</div>



**In [11]:**

{% highlight python %}
apps.info()
{% endhighlight %}

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1603111 entries, 0 to 1603110
    Data columns (total 5 columns):
    UserID             1603111 non-null int64
    WindowID           1603111 non-null int64
    Split              1603111 non-null object
    ApplicationDate    1603111 non-null object
    JobID              1603111 non-null int64
    dtypes: int64(3), object(2)
    memory usage: 61.2+ MB


**In [12]:**

{% highlight python %}
application = apps_training[['UserID', 'JobID']]
{% endhighlight %}

**In [13]:**

{% highlight python %}
grouped_apps = application.groupby(['UserID', 'JobID']).sum().reset_index()
{% endhighlight %}

**In [14]:**

{% highlight python %}
grouped_apps['Quantity'] = 1
{% endhighlight %}

**In [15]:**

{% highlight python %}
grouped_apps.head()
{% endhighlight %}




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
      <th>UserID</th>
      <th>JobID</th>
      <th>Quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7</td>
      <td>309823</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7</td>
      <td>703889</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9</td>
      <td>136489</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9</td>
      <td>617374</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9</td>
      <td>809208</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>


 
## Look-up 

**In [16]:**

{% highlight python %}
jobs_US_base_line.head()
{% endhighlight %}




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
      <th>JobID</th>
      <th>WindowID</th>
      <th>Title</th>
      <th>Description</th>
      <th>Requirements</th>
      <th>City</th>
      <th>State</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Security Engineer/Technical Lead</td>
      <td>security engineer technical leadsecurity clear...</td>
      <td>&lt;p&gt;SKILL SET&lt;/p&gt;\r&lt;p&gt;&amp;nbsp;&lt;/p&gt;\r&lt;p&gt;Network Se...</td>
      <td>Washington</td>
      <td>DC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>1</td>
      <td>SAP Business Analyst / WM</td>
      <td>sap business analyst wmno corp to corp resumes...</td>
      <td>&lt;p&gt;&lt;b&gt;WHAT YOU NEED: &lt;/b&gt;&lt;/p&gt;\r&lt;p&gt;Four year co...</td>
      <td>Charlotte</td>
      <td>NC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7</td>
      <td>1</td>
      <td>P/T HUMAN RESOURCES ASSISTANT</td>
      <td>p t human resources assistant p t human resour...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Winter Park</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>1</td>
      <td>Route Delivery Drivers</td>
      <td>route delivery driverscity beverages come to w...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9</td>
      <td>1</td>
      <td>Housekeeping</td>
      <td>housekeepingi make sure every part of their da...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
  </tbody>
</table>
</div>



**In [17]:**

{% highlight python %}
job_lookup = jobs[['JobID','Title']].drop_duplicates()
job_lookup['Title'] = job_lookup.Title.astype(str)
{% endhighlight %}

**In [18]:**

{% highlight python %}
job_lookup.head()
{% endhighlight %}




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
      <th>JobID</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Security Engineer/Technical Lead</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>SAP Business Analyst / WM</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7</td>
      <td>P/T HUMAN RESOURCES ASSISTANT</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>Route Delivery Drivers</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9</td>
      <td>Housekeeping</td>
    </tr>
  </tbody>
</table>
</div>


 
## Create the sparse ratings matrix of users and items utilizing the code 

**In [19]:**

{% highlight python %}
users = list(np.sort(grouped_apps.UserID.unique()))
jobs = list(grouped_apps.JobID.unique())
application = list(grouped_apps.Quantity)

rows = grouped_apps.UserID.astype('category', categories = users).cat.codes
cols = grouped_apps.JobID.astype('category', categories = jobs).cat.codes

apps_sparse = sparse.csr_matrix((application, (rows, cols)), shape=(len(users), len(jobs)))
{% endhighlight %}

    /home/hectoryee/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:5: FutureWarning: specifying 'categories' or 'ordered' in .astype() is deprecated; pass a CategoricalDtype instead
      """
    /home/hectoryee/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:6: FutureWarning: specifying 'categories' or 'ordered' in .astype() is deprecated; pass a CategoricalDtype instead
      


**In [20]:**

{% highlight python %}
(apps_sparse)
{% endhighlight %}




    <308033x349742 sparse matrix of type '<class 'numpy.int64'>'
    	with 1417514 stored elements in Compressed Sparse Row format>



**In [21]:**

{% highlight python %}
matrix_size = apps_sparse.shape[0] * apps_sparse.shape[1]
num_apps = len(apps_sparse.nonzero()[0])
sparsity = 100 * (1 - (num_apps/matrix_size))
sparsity
{% endhighlight %}




    99.99868422290457


 
98.3% of the interaction matrix is sparse. For collaborative filtering to work,
the maximum sparsity you could get away with would probably be about 99.5% or
so. We are above this, might not get decent results. 
 
## Dataset 

**In [22]:**

{% highlight python %}
def make_train(ratings, pct_test = 0.2):
    '''
    This function will take in the original user-item matrix and "mask" a percentage of the original ratings where a
    user-item interaction has taken place for use as a test set. The test set will contain all of the original ratings, 
    while the training set replaces the specified percentage of them with a zero in the original ratings matrix. 
    
    parameters: 
    
    ratings - the original ratings matrix from which you want to generate a train/test set. Test is just a complete
    copy of the original set. This is in the form of a sparse csr_matrix. 
    
    pct_test - The percentage of user-item interactions where an interaction took place that you want to mask in the 
    training set for later comparison to the test set, which contains all of the original ratings. 
    
    returns:
    
    training_set - The altered version of the original data with a certain percentage of the user-item pairs 
    that originally had interaction set back to zero.
    
    test_set - A copy of the original ratings matrix, unaltered, so it can be used to see how the rank order 
    compares with the actual interactions.
    
    user_inds - From the randomly selected user-item indices, which user rows were altered in the training data.
    This will be necessary later when evaluating the performance via AUC.
    '''
    test_set = ratings.copy() # Make a copy of the original set to be the test set. 
    test_set[test_set != 0] = 1 # Store the test set as a binary preference matrix
    training_set = ratings.copy() # Make a copy of the original data we can alter as our training set. 
    nonzero_inds = training_set.nonzero() # Find the indices in the ratings data where an interaction exists
    nonzero_pairs = list(zip(nonzero_inds[0], nonzero_inds[1])) # Zip these pairs together of user,item index into list
    random.seed(0) # Set the random seed to zero for reproducibility
    num_samples = int(np.ceil(pct_test*len(nonzero_pairs))) # Round the number of samples needed to the nearest integer
    samples = random.sample(nonzero_pairs, num_samples) # Sample a random number of user-item pairs without replacement
    user_inds = [index[0] for index in samples] # Get the user row indices
    item_inds = [index[1] for index in samples] # Get the item column indices
    training_set[user_inds, item_inds] = 0 # Assign all of the randomly chosen user-item pairs to zero
    training_set.eliminate_zeros() # Get rid of zeros in sparse array storage after update to save space
    return training_set, test_set, list(set(user_inds)) # Output the unique list of user rows that were altered  
{% endhighlight %}

**In [23]:**

{% highlight python %}
product_train, product_test, product_users_altered = make_train(apps_sparse, pct_test = 0.2)
{% endhighlight %}
 
## Implement implicit 

**In [24]:**

{% highlight python %}
import implicit
{% endhighlight %}

**In [25]:**

{% highlight python %}
alpha = 15
user_vecs, item_vecs = implicit.alternating_least_squares((product_train*alpha).astype('double'), 
                                                          factors=20, 
                                                          regularization = 0.1, 
                                                         iterations = 50)
{% endhighlight %}

    This method is deprecated. Please use the AlternatingLeastSquares class instead
    WARNING:root:Intel MKL BLAS detected. Its highly recommend to set the environment variable 'export MKL_NUM_THREADS=1' to disable its internal multithreading
    100%|██████████| 50.0/50 [00:30<00:00,  1.44it/s]

 
## A Recommendation Example 

**In [30]:**

{% highlight python %}
users_arr = np.array(users) # Array of customer IDs from the ratings matrix
jobs_arr = np.array(jobs ) # Array of product IDs from the ratings matrix
{% endhighlight %}

**In [31]:**

{% highlight python %}
def get_items_purchased(customer_id, mf_train, customers_list, products_list, item_lookup):
    '''
    This just tells me which items have been already purchased by a specific user in the training set. 
    
    parameters: 
    
    customer_id - Input the customer's id number that you want to see prior purchases of at least once
    
    mf_train - The initial ratings training set used (without weights applied)
    
    customers_list - The array of customers used in the ratings matrix
    
    products_list - The array of products used in the ratings matrix
    
    item_lookup - A simple pandas dataframe of the unique product ID/product descriptions available
    
    returns:
    
    A list of item IDs and item descriptions for a particular customer that were already purchased in the training set
    '''
    cust_ind = np.where(customers_list == customer_id)[0][0] # Returns the index row of our customer id
    purchased_ind = mf_train[cust_ind,:].nonzero()[1] # Get column indices of purchased items
    prod_codes = products_list[purchased_ind] # Get the stock codes for our purchased items
#     return prod_codes
    return item_lookup.loc[item_lookup.JobID.isin(prod_codes)]

{% endhighlight %}

**In [32]:**

{% highlight python %}
users_arr[:5]
{% endhighlight %}




    array([ 7,  9, 14, 16, 18])



**In [33]:**

{% highlight python %}
job_lookup.info()
{% endhighlight %}

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1091923 entries, 0 to 1091922
    Data columns (total 2 columns):
    JobID    1091923 non-null int64
    Title    1091923 non-null object
    dtypes: int64(1), object(1)
    memory usage: 25.0+ MB


**In [34]:**

{% highlight python %}
get_items_purchased(14, product_train, users_arr, jobs_arr, job_lookup)
{% endhighlight %}




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
      <th>JobID</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>605671</th>
      <td>206046</td>
      <td>Materials Manager / Director</td>
    </tr>
    <tr>
      <th>626723</th>
      <td>372423</td>
      <td>Sales Manager</td>
    </tr>
    <tr>
      <th>652060</th>
      <td>574999</td>
      <td>Mortgage Loan Specialist</td>
    </tr>
    <tr>
      <th>663486</th>
      <td>663552</td>
      <td>Mortgage Professionals - All Levels</td>
    </tr>
    <tr>
      <th>678829</th>
      <td>787741</td>
      <td>Planner Specialist</td>
    </tr>
    <tr>
      <th>702882</th>
      <td>978868</td>
      <td>Branch Coordinator</td>
    </tr>
  </tbody>
</table>
</div>



**In [35]:**

{% highlight python %}
from sklearn.preprocessing import MinMaxScaler
{% endhighlight %}

**In [36]:**

{% highlight python %}
def rec_items(customer_id, mf_train, user_vecs, item_vecs, customer_list, item_list, item_lookup, num_items = 10):
    '''
    This function will return the top recommended items to our users 
    
    parameters:
    
    customer_id - Input the customer's id number that you want to get recommendations for
    
    mf_train - The training matrix you used for matrix factorization fitting
    
    user_vecs - the user vectors from your fitted matrix factorization
    
    item_vecs - the item vectors from your fitted matrix factorization
    
    customer_list - an array of the customer's ID numbers that make up the rows of your ratings matrix 
                    (in order of matrix)
    
    item_list - an array of the products that make up the columns of your ratings matrix
                    (in order of matrix)
    
    item_lookup - A simple pandas dataframe of the unique product ID/product descriptions available
    
    num_items - The number of items you want to recommend in order of best recommendations. Default is 10. 
    
    returns:
    
    - The top n recommendations chosen based on the user/item vectors for items never interacted with/purchased
    '''
    
    cust_ind = np.where(customer_list == customer_id)[0][0] # Returns the index row of our customer id
    pref_vec = mf_train[cust_ind,:].toarray() # Get the ratings from the training set ratings matrix
    pref_vec = pref_vec.reshape(-1) + 1 # Add 1 to everything, so that items not purchased yet become equal to 1
    pref_vec[pref_vec > 1] = 0 # Make everything already purchased zero
    rec_vector = user_vecs[cust_ind,:].dot(item_vecs.T) # Get dot product of user vector and all item vectors
    # Scale this recommendation vector between 0 and 1
    min_max = MinMaxScaler()
    rec_vector_scaled = min_max.fit_transform(rec_vector.reshape(-1,1))[:,0] 
    recommend_vector = pref_vec*rec_vector_scaled 
    # Items already purchased have their recommendation multiplied by zero
    product_idx = np.argsort(recommend_vector)[::-1][:num_items] # Sort the indices of the items into order 
    # of best recommendations
    rec_list = [] # start empty list to store items
    for index in product_idx:
        code = item_list[index]
        rec_list.append([code, item_lookup.Title.loc[item_lookup.JobID == code].iloc[0]]) 
        # Append our descriptions to the list
    codes = [item[0] for item in rec_list]
    descriptions = [item[1] for item in rec_list]
    final_frame = pd.DataFrame({'JobId': codes, 'Title': descriptions}) # Create a dataframe 
    return final_frame[['JobId', 'Title']] # Switch order of columns around

{% endhighlight %}

**In [37]:**

{% highlight python %}
get_items_purchased(7, product_train, users_arr, jobs_arr, job_lookup)
{% endhighlight %}




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
      <th>JobID</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>479106</th>
      <td>309823</td>
      <td>Teller</td>
    </tr>
    <tr>
      <th>527966</th>
      <td>703889</td>
      <td>Customer Service Representative, Now Accepting...</td>
    </tr>
  </tbody>
</table>
</div>



**In [38]:**

{% highlight python %}
rec_items(7, product_train, user_vecs, item_vecs, users_arr, jobs_arr, job_lookup,
                       num_items = 10)
{% endhighlight %}




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
      <th>JobId</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1101648</td>
      <td>Customer Service/Receptionist/Front Desk</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12561</td>
      <td>Dental Hygienist - up to $40/ hour!</td>
    </tr>
    <tr>
      <th>2</th>
      <td>802205</td>
      <td>Customer Service Representative</td>
    </tr>
    <tr>
      <th>3</th>
      <td>344145</td>
      <td>Customer Service Representative</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1050711</td>
      <td>Customer Service Representative</td>
    </tr>
    <tr>
      <th>5</th>
      <td>601021</td>
      <td>Customer Service/Receptionist/Call Center Rep</td>
    </tr>
    <tr>
      <th>6</th>
      <td>601126</td>
      <td>Customer Service/Front Desk Help/Receptionist</td>
    </tr>
    <tr>
      <th>7</th>
      <td>394969</td>
      <td>Call Center Customer Service Collections / Credit</td>
    </tr>
    <tr>
      <th>8</th>
      <td>287182</td>
      <td>Administrative Assistant</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1046722</td>
      <td>Data Entry Associate</td>
    </tr>
  </tbody>
</table>
</div>


 
Second Implementation using [implicit
library](https://github.com/benfred/implicit). 

**In [39]:**

{% highlight python %}
from implicit.als import AlternatingLeastSquares
{% endhighlight %}

**In [40]:**

{% highlight python %}
# train model
model = AlternatingLeastSquares(factors=50,
                                regularization=0.01,
                                dtype=np.float64,
                                iterations=50)

confidence = 40
model.fit(confidence * apps_sparse)
{% endhighlight %}

    100%|██████████| 50.0/50 [00:51<00:00,  1.07it/s]


**In [41]:**

{% highlight python %}
# recommend items for a user
user_items = apps_sparse.T.tocsr()
recommendations = model.recommend(7, user_items)

# find related items
# related = model.similar_items(itemid)

{% endhighlight %}

**In [42]:**

{% highlight python %}
recommendations
{% endhighlight %}




    [(57371, 0.029422828555080816),
     (95894, 0.028675761246652852),
     (168777, 0.026280032617673803),
     (186945, 0.025853081049481193),
     (6434, 0.022265122614712515),
     (215515, 0.02175052321349273),
     (286526, 0.02006039079638639),
     (188385, 0.019824223520708895),
     (306468, 0.019819373496054445),
     (122115, 0.019597172209618055)]



**In [43]:**

{% highlight python %}
recommend=[]
for i in range(len(recommendations)):
    recommend.append(recommendations[i][0])
{% endhighlight %}

**In [44]:**

{% highlight python %}
rec_items(7, product_train, user_vecs, item_vecs, users_arr, jobs_arr, job_lookup,
                       num_items = 10)
{% endhighlight %}




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
      <th>JobId</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1101648</td>
      <td>Customer Service/Receptionist/Front Desk</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12561</td>
      <td>Dental Hygienist - up to $40/ hour!</td>
    </tr>
    <tr>
      <th>2</th>
      <td>802205</td>
      <td>Customer Service Representative</td>
    </tr>
    <tr>
      <th>3</th>
      <td>344145</td>
      <td>Customer Service Representative</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1050711</td>
      <td>Customer Service Representative</td>
    </tr>
    <tr>
      <th>5</th>
      <td>601021</td>
      <td>Customer Service/Receptionist/Call Center Rep</td>
    </tr>
    <tr>
      <th>6</th>
      <td>601126</td>
      <td>Customer Service/Front Desk Help/Receptionist</td>
    </tr>
    <tr>
      <th>7</th>
      <td>394969</td>
      <td>Call Center Customer Service Collections / Credit</td>
    </tr>
    <tr>
      <th>8</th>
      <td>287182</td>
      <td>Administrative Assistant</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1046722</td>
      <td>Data Entry Associate</td>
    </tr>
  </tbody>
</table>
</div>



**In [45]:**

{% highlight python %}
job_lookup.loc[job_lookup.JobID.isin(recommend)]
{% endhighlight %}




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
      <th>JobID</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14973</th>
      <td>57371</td>
      <td>Termite Inspector (147-480)</td>
    </tr>
    <tr>
      <th>72738</th>
      <td>286526</td>
      <td>Customer Service Representative/E-Marketing Co...</td>
    </tr>
    <tr>
      <th>308148</th>
      <td>168777</td>
      <td>A/R Medical Biller</td>
    </tr>
    <tr>
      <th>310916</th>
      <td>186945</td>
      <td>Wireless Project Manager</td>
    </tr>
    <tr>
      <th>441140</th>
      <td>6434</td>
      <td>CNC Machinist</td>
    </tr>
    <tr>
      <th>451716</th>
      <td>95894</td>
      <td>Part-Time Customer Service Representative</td>
    </tr>
    <tr>
      <th>466763</th>
      <td>215515</td>
      <td>Product Specialist!!! - Excellent Opportunity!</td>
    </tr>
    <tr>
      <th>594338</th>
      <td>122115</td>
      <td>Maintenance Technician</td>
    </tr>
    <tr>
      <th>893325</th>
      <td>306468</td>
      <td>Data / Business Analyst – Deliver Relevance th...</td>
    </tr>
    <tr>
      <th>996719</th>
      <td>188385</td>
      <td>Assistant Store Manager - Casual Male XL</td>
    </tr>
  </tbody>
</table>
</div>



**In [46]:**

{% highlight python %}
get_items_purchased(7, product_train, users_arr, jobs_arr, job_lookup)
{% endhighlight %}




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
      <th>JobID</th>
      <th>Title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>479106</th>
      <td>309823</td>
      <td>Teller</td>
    </tr>
    <tr>
      <th>527966</th>
      <td>703889</td>
      <td>Customer Service Representative, Now Accepting...</td>
    </tr>
  </tbody>
</table>
</div>


 
## References

- [ALS Implicit Collaborative Filtering](https://medium.com/radon-dev/als-
implicit-collaborative-filtering-5ed653ba39fe)
- [A Gentle Introduction to Recommender Systems with Implicit
Feedback](https://jessesw.com/Rec-System/)
- [Finding Similar Music using Matrix
Factorization](http://www.benfrederickson.com/matrix-factorization/)

## Jupyter Notebook

{% gist cfbacc1773000e91a4d05434b7e99131 job_als.ipynb %}