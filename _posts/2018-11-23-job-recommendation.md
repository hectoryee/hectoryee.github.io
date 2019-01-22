---
title: Job Recommendation Engine
desc: Recommendsation engine.
published: true
date: 2018-11-23
categories: project
tags: notebook recommendation data-science
---

### Collaborative filtering
analyse information on user'd behaviours and predict what users will like based on their similarity to other users
algo: k-NN approach, Pearson Correlation
info: explicit and implicit data
suffers from cold start, scalability, sparsity
matrix factorisation

### Content-based filtering (personality-based approach)
based on description of item and  profile of user's preference, recommend similar items
tf-idf representation (item representation algo)
info: user's preference, user's history
recommendation is narrow

### Hybrid recommender systems
combine collaborative filtering and content-based filtering, might be more effective sometimes

### Job Recommendation with dataset from [Careerbuilder](https://www.kaggle.com/c/job-recommendation)
https://github.com/blennon/kaggle/tree/master/careerbuilder

# Job Recommendation

with dataset from Careerbuilder 
 
## TOC
* [Import dependencies](#import)
* [Load dataset](#dataset)
* [EDA and Preprocessing](#EDA)
 - [split into training and testing dataset](#split)
 - [location](#location)
 - [preprocessing](#preprocessing)
* [Building model](#model)
* [Clean html](#html)
 - [Tokenising](#token)
* [Text processing](#textprocessing)
 - [BOW model](#bow)
 - [Term frequency-inverse document frequency](#tfidf)

* [similar users](#similarusers) 
 
## Import dependencies <a class="anchor" id="import"></a> 

**In [1]:**

{% highlight python %}
%matplotlib inline
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np
{% endhighlight %}
 
## Load dataset <a class="anchor" id="dataset"></a> 

**In [2]:**

{% highlight python %}
!ls data/*.tsv
{% endhighlight %}

    data/apps.tsv  data/test_users.tsv    data/users.tsv
    data/jobs.tsv  data/user_history.tsv  data/window_dates.tsv

 
- users
- jobs
- apps
- users_history
- test_users 

**In [3]:**

{% highlight python %}
users = pd.read_csv('data/users.tsv', sep='\t', encoding='utf-8')
jobs = pd.read_csv('data/jobs.tsv', sep='\t', encoding='utf-8', error_bad_lines=False)
apps = pd.read_csv('data/apps.tsv', sep='\t', encoding='utf-8')
user_history = pd.read_csv('data/user_history.tsv', sep='\t', encoding='utf-8')
test_users = pd.read_csv('data/test_users.tsv', sep='\t', encoding='utf-8')

# jobs = pd.read_csv('data/jobs.tsv', sep='\t')
{% endhighlight %}

    b'Skipping line 122433: expected 11 fields, saw 12\n'
    b'Skipping line 602576: expected 11 fields, saw 12\n'
    b'Skipping line 990950: expected 11 fields, saw 12\n'
    /home/hectoryee/anaconda3/lib/python3.6/site-packages/IPython/core/interactiveshell.py:2785: DtypeWarning: Columns (8) have mixed types. Specify dtype option on import or set low_memory=False.
      interactivity=interactivity, compiler=compiler, result=result)


**In [4]:**

{% highlight python %}
users.head()
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>ZipCode</th>
      <th>DegreeType</th>
      <th>Major</th>
      <th>GraduationDate</th>
      <th>WorkHistoryCount</th>
      <th>TotalYearsExperience</th>
      <th>CurrentlyEmployed</th>
      <th>ManagedOthers</th>
      <th>ManagedHowMany</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>Paramount</td>
      <td>CA</td>
      <td>US</td>
      <td>90723</td>
      <td>High School</td>
      <td>NaN</td>
      <td>1999-06-01 00:00:00</td>
      <td>3</td>
      <td>10.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>La Mesa</td>
      <td>CA</td>
      <td>US</td>
      <td>91941</td>
      <td>Master's</td>
      <td>Anthropology</td>
      <td>2011-01-01 00:00:00</td>
      <td>10</td>
      <td>8.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>80</td>
      <td>1</td>
      <td>Train</td>
      <td>Williamstown</td>
      <td>NJ</td>
      <td>US</td>
      <td>08094</td>
      <td>High School</td>
      <td>Not Applicable</td>
      <td>1985-06-01 00:00:00</td>
      <td>5</td>
      <td>11.0</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>98</td>
      <td>1</td>
      <td>Train</td>
      <td>Astoria</td>
      <td>NY</td>
      <td>US</td>
      <td>11105</td>
      <td>Master's</td>
      <td>Journalism</td>
      <td>2007-05-01 00:00:00</td>
      <td>3</td>
      <td>3.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>123</td>
      <td>1</td>
      <td>Train</td>
      <td>Baton Rouge</td>
      <td>LA</td>
      <td>US</td>
      <td>70808</td>
      <td>Bachelor's</td>
      <td>Agricultural Business</td>
      <td>2011-05-01 00:00:00</td>
      <td>1</td>
      <td>9.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



**In [5]:**

{% highlight python %}
users.columns
{% endhighlight %}




    Index(['UserID', 'WindowID', 'Split', 'City', 'State', 'Country', 'ZipCode',
           'DegreeType', 'Major', 'GraduationDate', 'WorkHistoryCount',
           'TotalYearsExperience', 'CurrentlyEmployed', 'ManagedOthers',
           'ManagedHowMany'],
          dtype='object')



**In [6]:**

{% highlight python %}
users.info()
{% endhighlight %}

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 389708 entries, 0 to 389707
    Data columns (total 15 columns):
    UserID                  389708 non-null int64
    WindowID                389708 non-null int64
    Split                   389708 non-null object
    City                    389708 non-null object
    State                   389218 non-null object
    Country                 389708 non-null object
    ZipCode                 387974 non-null object
    DegreeType              389708 non-null object
    Major                   292468 non-null object
    GraduationDate          269477 non-null object
    WorkHistoryCount        389708 non-null int64
    TotalYearsExperience    375528 non-null float64
    CurrentlyEmployed       347632 non-null object
    ManagedOthers           389708 non-null object
    ManagedHowMany          389708 non-null int64
    dtypes: float64(1), int64(4), object(10)
    memory usage: 44.6+ MB


**In [7]:**

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



**In [8]:**

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



**In [9]:**

{% highlight python %}
user_history.head()
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
      <th>Sequence</th>
      <th>JobTitle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>1</td>
      <td>National Space Communication Programs-Special ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2</td>
      <td>Detention Officer</td>
    </tr>
    <tr>
      <th>2</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>3</td>
      <td>Passenger Screener, TSA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>1</td>
      <td>Lecturer, Department of Anthropology</td>
    </tr>
    <tr>
      <th>4</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>2</td>
      <td>Student Assistant</td>
    </tr>
  </tbody>
</table>
</div>



**In [10]:**

{% highlight python %}
test_users.head()
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>767</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>769</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>861</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1006</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1192</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>


 
## EDA and Preprocessing <a class="anchor" id="EDA"></a> 
 
### Splitting into Training and Testing dataset <a class="anchor"
id="split"></a> 
 
with attribute split:
- users
- apps
- user_history 
 
### users 

**In [11]:**

{% highlight python %}
users_training = users.loc[users['Split'] == 'Train']
users_training.head()
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>ZipCode</th>
      <th>DegreeType</th>
      <th>Major</th>
      <th>GraduationDate</th>
      <th>WorkHistoryCount</th>
      <th>TotalYearsExperience</th>
      <th>CurrentlyEmployed</th>
      <th>ManagedOthers</th>
      <th>ManagedHowMany</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>Paramount</td>
      <td>CA</td>
      <td>US</td>
      <td>90723</td>
      <td>High School</td>
      <td>NaN</td>
      <td>1999-06-01 00:00:00</td>
      <td>3</td>
      <td>10.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>La Mesa</td>
      <td>CA</td>
      <td>US</td>
      <td>91941</td>
      <td>Master's</td>
      <td>Anthropology</td>
      <td>2011-01-01 00:00:00</td>
      <td>10</td>
      <td>8.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>80</td>
      <td>1</td>
      <td>Train</td>
      <td>Williamstown</td>
      <td>NJ</td>
      <td>US</td>
      <td>08094</td>
      <td>High School</td>
      <td>Not Applicable</td>
      <td>1985-06-01 00:00:00</td>
      <td>5</td>
      <td>11.0</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>98</td>
      <td>1</td>
      <td>Train</td>
      <td>Astoria</td>
      <td>NY</td>
      <td>US</td>
      <td>11105</td>
      <td>Master's</td>
      <td>Journalism</td>
      <td>2007-05-01 00:00:00</td>
      <td>3</td>
      <td>3.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>123</td>
      <td>1</td>
      <td>Train</td>
      <td>Baton Rouge</td>
      <td>LA</td>
      <td>US</td>
      <td>70808</td>
      <td>Bachelor's</td>
      <td>Agricultural Business</td>
      <td>2011-05-01 00:00:00</td>
      <td>1</td>
      <td>9.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



**In [12]:**

{% highlight python %}
users_training.shape
{% endhighlight %}




    (366870, 15)



**In [13]:**

{% highlight python %}
users_testing = users.loc[users['Split'] == 'Test']
{% endhighlight %}

**In [14]:**

{% highlight python %}
users_testing.head()
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>ZipCode</th>
      <th>DegreeType</th>
      <th>Major</th>
      <th>GraduationDate</th>
      <th>WorkHistoryCount</th>
      <th>TotalYearsExperience</th>
      <th>CurrentlyEmployed</th>
      <th>ManagedOthers</th>
      <th>ManagedHowMany</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>31</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>Murrieta</td>
      <td>CA</td>
      <td>US</td>
      <td>92562</td>
      <td>Bachelor's</td>
      <td>University Studies/Business</td>
      <td>2008-05-01 00:00:00</td>
      <td>5</td>
      <td>16.0</td>
      <td>No</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>769</td>
      <td>1</td>
      <td>Test</td>
      <td>Roselle</td>
      <td>IL</td>
      <td>US</td>
      <td>60172</td>
      <td>Bachelor's</td>
      <td>Radio-Television</td>
      <td>2011-05-01 00:00:00</td>
      <td>5</td>
      <td>5.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>33</th>
      <td>861</td>
      <td>1</td>
      <td>Test</td>
      <td>Morris</td>
      <td>IL</td>
      <td>US</td>
      <td>60450</td>
      <td>High School</td>
      <td>General Studies</td>
      <td>1989-05-01 00:00:00</td>
      <td>7</td>
      <td>21.0</td>
      <td>NaN</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>38</th>
      <td>1006</td>
      <td>1</td>
      <td>Test</td>
      <td>West Chester</td>
      <td>PA</td>
      <td>US</td>
      <td>19382</td>
      <td>High School</td>
      <td>Not Applicable</td>
      <td>2008-06-01 00:00:00</td>
      <td>3</td>
      <td>6.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>44</th>
      <td>1192</td>
      <td>1</td>
      <td>Test</td>
      <td>Cincinnati</td>
      <td>OH</td>
      <td>US</td>
      <td>45255</td>
      <td>Bachelor's</td>
      <td>Marketing</td>
      <td>NaN</td>
      <td>5</td>
      <td>6.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>


 
### apps 

**In [15]:**

{% highlight python %}
apps_training = apps.loc[apps['Split'] == 'Train']
apps_training.head()
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



**In [16]:**

{% highlight python %}
apps_training.shape
{% endhighlight %}




    (1417514, 5)



**In [17]:**

{% highlight python %}
apps_testing = apps.loc[apps['Split'] == 'Test']
apps_testing.head()
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
      <th>126</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>2012-04-01 14:37:20.023</td>
      <td>85377</td>
    </tr>
    <tr>
      <th>127</th>
      <td>769</td>
      <td>1</td>
      <td>Test</td>
      <td>2012-04-16 22:36:52.48</td>
      <td>853328</td>
    </tr>
    <tr>
      <th>128</th>
      <td>769</td>
      <td>1</td>
      <td>Test</td>
      <td>2012-04-09 18:59:28.193</td>
      <td>86106</td>
    </tr>
    <tr>
      <th>129</th>
      <td>769</td>
      <td>1</td>
      <td>Test</td>
      <td>2012-04-09 18:59:31.127</td>
      <td>327571</td>
    </tr>
    <tr>
      <th>130</th>
      <td>769</td>
      <td>1</td>
      <td>Test</td>
      <td>2012-04-08 21:29:11.993</td>
      <td>119161</td>
    </tr>
  </tbody>
</table>
</div>



**In [18]:**

{% highlight python %}
apps_testing.shape
{% endhighlight %}




    (185597, 5)


 
### user_history 

**In [19]:**

{% highlight python %}
user_history_training = user_history.loc[user_history['Split'] == 'Train']
user_history_training.head()
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
      <th>Sequence</th>
      <th>JobTitle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>1</td>
      <td>National Space Communication Programs-Special ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>2</td>
      <td>Detention Officer</td>
    </tr>
    <tr>
      <th>2</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>3</td>
      <td>Passenger Screener, TSA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>1</td>
      <td>Lecturer, Department of Anthropology</td>
    </tr>
    <tr>
      <th>4</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>2</td>
      <td>Student Assistant</td>
    </tr>
  </tbody>
</table>
</div>



**In [20]:**

{% highlight python %}
user_history_training.shape
{% endhighlight %}




    (1652513, 5)



**In [21]:**

{% highlight python %}
user_history_testing = user_history.loc[user_history['Split'] == 'Test']
user_history_testing.head()
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
      <th>Sequence</th>
      <th>JobTitle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>144</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>1</td>
      <td>Claims Adjuster</td>
    </tr>
    <tr>
      <th>145</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>2</td>
      <td>Professional Baseball Player</td>
    </tr>
    <tr>
      <th>146</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>3</td>
      <td>Professional Baseball Player</td>
    </tr>
    <tr>
      <th>147</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>4</td>
      <td>Professional Baseball Player</td>
    </tr>
    <tr>
      <th>148</th>
      <td>767</td>
      <td>1</td>
      <td>Test</td>
      <td>5</td>
      <td>Professional Baseball Player</td>
    </tr>
  </tbody>
</table>
</div>



**In [22]:**

{% highlight python %}
user_history_testing.shape
{% endhighlight %}




    (101388, 5)


 
#### Dataframes
- users_training
- users_testing
- apps_training
- apps_testing
- user_history_training
- user_history_testing 
 
### Location <a class="anchor" id="location"></a> 

**In [23]:**

{% highlight python %}
jobs.groupby(['City','State','Country']).size().reset_index(name='Locationwise')
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>Locationwise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>&amp;#60;&amp;#47;b&amp;#62; Brno &amp;#60;b&amp;#62;</td>
      <td></td>
      <td>CZ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>&amp;#60;&amp;#47;b&amp;#62; Praha &amp;#60;b&amp;#62;</td>
      <td></td>
      <td>CZ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>- Any</td>
      <td></td>
      <td>CZ</td>
      <td>13</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29 Palms</td>
      <td>CA</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>&lt;</td>
      <td></td>
      <td>HU</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>&lt;</td>
      <td></td>
      <td>TR</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>AMF O'Hare</td>
      <td>IL</td>
      <td>US</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>APAC-Australia</td>
      <td></td>
      <td>AU</td>
      <td>68</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Aaron</td>
      <td>IN</td>
      <td>US</td>
      <td>6</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Abanaka</td>
      <td>OH</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Abbeville</td>
      <td>LA</td>
      <td>US</td>
      <td>34</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Abbeville</td>
      <td>SC</td>
      <td>US</td>
      <td>10</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Abbot</td>
      <td>VA</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Abbotsford</td>
      <td>WI</td>
      <td>US</td>
      <td>14</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Abbott Park</td>
      <td>IL</td>
      <td>US</td>
      <td>22</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Abbottstown</td>
      <td>PA</td>
      <td>US</td>
      <td>11</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Abell</td>
      <td>MD</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Abercrombie</td>
      <td>ND</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Aberdeen</td>
      <td>ID</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Aberdeen</td>
      <td>MD</td>
      <td>US</td>
      <td>214</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Aberdeen</td>
      <td>MS</td>
      <td>US</td>
      <td>5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Aberdeen</td>
      <td>NC</td>
      <td>US</td>
      <td>27</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Aberdeen</td>
      <td>NJ</td>
      <td>US</td>
      <td>13</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Aberdeen</td>
      <td>OH</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Aberdeen</td>
      <td>PA</td>
      <td>US</td>
      <td>3</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Aberdeen</td>
      <td>SD</td>
      <td>US</td>
      <td>101</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Aberdeen</td>
      <td>WA</td>
      <td>US</td>
      <td>42</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Aberdeen Gardens</td>
      <td>WA</td>
      <td>US</td>
      <td>2</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Aberdeen Proving Ground</td>
      <td>MD</td>
      <td>US</td>
      <td>98</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Abeyta</td>
      <td>CO</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>15664</th>
      <td>Zephyr Cove</td>
      <td>NV</td>
      <td>US</td>
      <td>5</td>
    </tr>
    <tr>
      <th>15665</th>
      <td>Zephyrhills</td>
      <td>FL</td>
      <td>US</td>
      <td>69</td>
    </tr>
    <tr>
      <th>15666</th>
      <td>Zillah</td>
      <td>WA</td>
      <td>US</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15667</th>
      <td>Zimmerdale</td>
      <td>KS</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15668</th>
      <td>Zimmerman</td>
      <td>MN</td>
      <td>US</td>
      <td>6</td>
    </tr>
    <tr>
      <th>15669</th>
      <td>Zinc</td>
      <td>AR</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15670</th>
      <td>Zion</td>
      <td>IL</td>
      <td>US</td>
      <td>39</td>
    </tr>
    <tr>
      <th>15671</th>
      <td>Zion</td>
      <td>VA</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15672</th>
      <td>Zionsville</td>
      <td>IN</td>
      <td>US</td>
      <td>26</td>
    </tr>
    <tr>
      <th>15673</th>
      <td>Zullinger</td>
      <td>PA</td>
      <td>US</td>
      <td>6</td>
    </tr>
    <tr>
      <th>15674</th>
      <td>Zumbrota</td>
      <td>MN</td>
      <td>US</td>
      <td>6</td>
    </tr>
    <tr>
      <th>15675</th>
      <td>Zuni</td>
      <td>NM</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15676</th>
      <td>Zurich</td>
      <td>MT</td>
      <td>US</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15677</th>
      <td>Zwolle</td>
      <td>LA</td>
      <td>US</td>
      <td>5</td>
    </tr>
    <tr>
      <th>15678</th>
      <td>adasd</td>
      <td></td>
      <td>BT</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15679</th>
      <td>asd</td>
      <td></td>
      <td>UG</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15680</th>
      <td>bainbridge island</td>
      <td>WA</td>
      <td>US</td>
      <td>11</td>
    </tr>
    <tr>
      <th>15681</th>
      <td>bhut</td>
      <td></td>
      <td>BT</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15682</th>
      <td>cafee</td>
      <td></td>
      <td>BT</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15683</th>
      <td>cali</td>
      <td></td>
      <td>BT</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15684</th>
      <td>california</td>
      <td></td>
      <td>AO</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15685</th>
      <td>casdd</td>
      <td></td>
      <td>BT</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15686</th>
      <td>mexico city</td>
      <td></td>
      <td>MX</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15687</th>
      <td>mexio city</td>
      <td></td>
      <td>MX</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15688</th>
      <td>moscow</td>
      <td></td>
      <td>RU</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15689</th>
      <td>puebla</td>
      <td></td>
      <td>MX</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15690</th>
      <td>san jose</td>
      <td></td>
      <td>BJ</td>
      <td>6</td>
    </tr>
    <tr>
      <th>15691</th>
      <td>san jose</td>
      <td></td>
      <td>UA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15692</th>
      <td>seoul</td>
      <td></td>
      <td>KR</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15693</th>
      <td>ulsoor</td>
      <td></td>
      <td>BT</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>15694 rows × 4 columns</p>
</div>



**In [24]:**

{% highlight python %}
jobs.groupby(['Country']).size().reset_index(name='Locationwise').sort_values('Locationwise',ascending=False).head()
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
      <th>Country</th>
      <th>Locationwise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>59</th>
      <td>US</td>
      <td>1090462</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AF</td>
      <td>560</td>
    </tr>
    <tr>
      <th>19</th>
      <td>CZ</td>
      <td>193</td>
    </tr>
    <tr>
      <th>40</th>
      <td>MX</td>
      <td>93</td>
    </tr>
    <tr>
      <th>52</th>
      <td>TR</td>
      <td>81</td>
    </tr>
  </tbody>
</table>
</div>



**In [25]:**

{% highlight python %}
Country_wise_job = jobs.groupby(['Country']).size().reset_index(name='Locationwise').sort_values('Locationwise', 
                                                                                                 ascending=False)
{% endhighlight %}

**In [26]:**

{% highlight python %}
plt.figure(figsize=(12,6))
ax = sns.barplot(x="Country", y="Locationwise", data=Country_wise_job)
ax.set_xticklabels(ax.get_xticklabels(), rotation=90, ha="right")
ax.set_title('Country wise job openings')
plt.tight_layout()
plt.show()
{% endhighlight %}

 
![png]({{ BASE_PATH }}/images/job_recommender_38_0.png) 

 
## Preprocessing <a class="anchor" id="preprocessing"></a>
- Considering only US
- Removing data with empty state 

**In [27]:**

{% highlight python %}
jobs_US = jobs.loc[jobs['Country'] == 'US']
{% endhighlight %}

**In [28]:**

{% highlight python %}
jobs_US[['City','State','Country']]
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Washington</td>
      <td>DC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Charlotte</td>
      <td>NC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Winter Park</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Ormond Beach</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Winter Park</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Los Angeles</td>
      <td>CA</td>
      <td>US</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Longwood</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Altamonte Springs</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Daytona Beach</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Oviedo</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Altamonte Springs</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Windermere</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Leesburg</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Longwood</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Apopka</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1091893</th>
      <td>Yonkers</td>
      <td>NY</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091894</th>
      <td>Newark</td>
      <td>NJ</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091895</th>
      <td>Charlotte</td>
      <td>NC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091896</th>
      <td>Columbus</td>
      <td>MN</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091897</th>
      <td>Las Vegas</td>
      <td>NV</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091898</th>
      <td>Westminster</td>
      <td>CO</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091899</th>
      <td>New York</td>
      <td>NY</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091900</th>
      <td>Columbia</td>
      <td>SC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091901</th>
      <td>Suffolk</td>
      <td>VA</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091902</th>
      <td>Chicago</td>
      <td>IL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091903</th>
      <td>Belleville</td>
      <td>MI</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091904</th>
      <td>Kalamazoo</td>
      <td>MI</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091905</th>
      <td>Durham</td>
      <td>NC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091906</th>
      <td>Waco</td>
      <td>TX</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091907</th>
      <td>Atlanta</td>
      <td>GA</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091908</th>
      <td>Columbia</td>
      <td>SC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091909</th>
      <td>Darlington</td>
      <td>SC</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091910</th>
      <td>Reynoldsburg</td>
      <td>OH</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091911</th>
      <td>Canton</td>
      <td>MS</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091912</th>
      <td>Saint Louis</td>
      <td>MO</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091913</th>
      <td>Appleton</td>
      <td>WI</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091914</th>
      <td>Nashville</td>
      <td>TN</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091915</th>
      <td>Albuquerque</td>
      <td>NM</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091916</th>
      <td>Chicago</td>
      <td>IL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091917</th>
      <td>Schaumburg</td>
      <td>IL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091918</th>
      <td>Amsterdam</td>
      <td>NY</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091919</th>
      <td>Birmingham</td>
      <td>AL</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091920</th>
      <td>Carthage</td>
      <td>MS</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091921</th>
      <td>Warren</td>
      <td>MI</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1091922</th>
      <td>Syracuse</td>
      <td>NY</td>
      <td>US</td>
    </tr>
  </tbody>
</table>
<p>1090462 rows × 3 columns</p>
</div>



**In [29]:**

{% highlight python %}
jobs_US.groupby(['City', 'State', 'Country']).size().reset_index(name = 'Locationwise').sort_values('Locationwise', ascending = False).head()
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>Locationwise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6601</th>
      <td>Houston</td>
      <td>TX</td>
      <td>US</td>
      <td>19306</td>
    </tr>
    <tr>
      <th>9835</th>
      <td>New York</td>
      <td>NY</td>
      <td>US</td>
      <td>18395</td>
    </tr>
    <tr>
      <th>2651</th>
      <td>Chicago</td>
      <td>IL</td>
      <td>US</td>
      <td>17806</td>
    </tr>
    <tr>
      <th>3475</th>
      <td>Dallas</td>
      <td>TX</td>
      <td>US</td>
      <td>13139</td>
    </tr>
    <tr>
      <th>610</th>
      <td>Atlanta</td>
      <td>GA</td>
      <td>US</td>
      <td>12352</td>
    </tr>
  </tbody>
</table>
</div>



**In [30]:**

{% highlight python %}
statewise_jobs = jobs_US.groupby(['State']).size().reset_index(name = 'Statewise').sort_values('Statewise', ascending = False)
{% endhighlight %}

**In [31]:**

{% highlight python %}
statewise_jobs
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
      <th>State</th>
      <th>Statewise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>CA</td>
      <td>109630</td>
    </tr>
    <tr>
      <th>48</th>
      <td>TX</td>
      <td>98071</td>
    </tr>
    <tr>
      <th>10</th>
      <td>FL</td>
      <td>71024</td>
    </tr>
    <tr>
      <th>16</th>
      <td>IL</td>
      <td>58743</td>
    </tr>
    <tr>
      <th>38</th>
      <td>NY</td>
      <td>53998</td>
    </tr>
    <tr>
      <th>42</th>
      <td>PA</td>
      <td>48999</td>
    </tr>
    <tr>
      <th>39</th>
      <td>OH</td>
      <td>45048</td>
    </tr>
    <tr>
      <th>34</th>
      <td>NJ</td>
      <td>35175</td>
    </tr>
    <tr>
      <th>30</th>
      <td>NC</td>
      <td>34553</td>
    </tr>
    <tr>
      <th>11</th>
      <td>GA</td>
      <td>33453</td>
    </tr>
    <tr>
      <th>50</th>
      <td>VA</td>
      <td>32339</td>
    </tr>
    <tr>
      <th>24</th>
      <td>MI</td>
      <td>29995</td>
    </tr>
    <tr>
      <th>22</th>
      <td>MD</td>
      <td>28523</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AZ</td>
      <td>27911</td>
    </tr>
    <tr>
      <th>21</th>
      <td>MA</td>
      <td>26756</td>
    </tr>
    <tr>
      <th>17</th>
      <td>IN</td>
      <td>26020</td>
    </tr>
    <tr>
      <th>47</th>
      <td>TN</td>
      <td>24773</td>
    </tr>
    <tr>
      <th>54</th>
      <td>WI</td>
      <td>23019</td>
    </tr>
    <tr>
      <th>25</th>
      <td>MN</td>
      <td>22697</td>
    </tr>
    <tr>
      <th>53</th>
      <td>WA</td>
      <td>21999</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CO</td>
      <td>21021</td>
    </tr>
    <tr>
      <th>26</th>
      <td>MO</td>
      <td>20394</td>
    </tr>
    <tr>
      <th>7</th>
      <td>CT</td>
      <td>16775</td>
    </tr>
    <tr>
      <th>19</th>
      <td>KY</td>
      <td>16328</td>
    </tr>
    <tr>
      <th>45</th>
      <td>SC</td>
      <td>15142</td>
    </tr>
    <tr>
      <th>18</th>
      <td>KS</td>
      <td>13184</td>
    </tr>
    <tr>
      <th>14</th>
      <td>IA</td>
      <td>12260</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AL</td>
      <td>12040</td>
    </tr>
    <tr>
      <th>20</th>
      <td>LA</td>
      <td>11762</td>
    </tr>
    <tr>
      <th>41</th>
      <td>OR</td>
      <td>10120</td>
    </tr>
    <tr>
      <th>40</th>
      <td>OK</td>
      <td>9761</td>
    </tr>
    <tr>
      <th>36</th>
      <td>NV</td>
      <td>7507</td>
    </tr>
    <tr>
      <th>8</th>
      <td>DC</td>
      <td>7387</td>
    </tr>
    <tr>
      <th>49</th>
      <td>UT</td>
      <td>7212</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AR</td>
      <td>6186</td>
    </tr>
    <tr>
      <th>28</th>
      <td>MS</td>
      <td>5770</td>
    </tr>
    <tr>
      <th>35</th>
      <td>NM</td>
      <td>5193</td>
    </tr>
    <tr>
      <th>9</th>
      <td>DE</td>
      <td>5153</td>
    </tr>
    <tr>
      <th>32</th>
      <td>NE</td>
      <td>4876</td>
    </tr>
    <tr>
      <th>55</th>
      <td>WV</td>
      <td>3616</td>
    </tr>
    <tr>
      <th>33</th>
      <td>NH</td>
      <td>3514</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ID</td>
      <td>3200</td>
    </tr>
    <tr>
      <th>31</th>
      <td>ND</td>
      <td>2788</td>
    </tr>
    <tr>
      <th>46</th>
      <td>SD</td>
      <td>2556</td>
    </tr>
    <tr>
      <th>44</th>
      <td>RI</td>
      <td>2525</td>
    </tr>
    <tr>
      <th>13</th>
      <td>HI</td>
      <td>2476</td>
    </tr>
    <tr>
      <th>23</th>
      <td>ME</td>
      <td>2089</td>
    </tr>
    <tr>
      <th>0</th>
      <td>AK</td>
      <td>2041</td>
    </tr>
    <tr>
      <th>52</th>
      <td>VT</td>
      <td>1840</td>
    </tr>
    <tr>
      <th>29</th>
      <td>MT</td>
      <td>1463</td>
    </tr>
    <tr>
      <th>56</th>
      <td>WY</td>
      <td>1072</td>
    </tr>
    <tr>
      <th>43</th>
      <td>PR</td>
      <td>406</td>
    </tr>
    <tr>
      <th>51</th>
      <td>VI</td>
      <td>33</td>
    </tr>
    <tr>
      <th>12</th>
      <td>GU</td>
      <td>32</td>
    </tr>
    <tr>
      <th>37</th>
      <td>NW</td>
      <td>11</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AS</td>
      <td>2</td>
    </tr>
    <tr>
      <th>27</th>
      <td>MP</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



**In [32]:**

{% highlight python %}
plt.figure(figsize=(12,6))
ax = sns.barplot(x="State", y="Statewise",data=statewise_jobs)
ax.set_xticklabels(ax.get_xticklabels(), rotation=90, ha = "right")
ax.set_title("State wise job openings")
plt.tight_layout
plt.show()
{% endhighlight %}

 
![png]({{ BASE_PATH }}/images/job_recommender_45_0.png) 


**In [33]:**

{% highlight python %}
jobs_US.groupby(['City']).size().reset_index(name='Citywise').sort_values('Citywise', ascending=False)
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
      <th>City</th>
      <th>Citywise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4564</th>
      <td>Houston</td>
      <td>19323</td>
    </tr>
    <tr>
      <th>6809</th>
      <td>New York</td>
      <td>18402</td>
    </tr>
    <tr>
      <th>1782</th>
      <td>Chicago</td>
      <td>17806</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>Dallas</td>
      <td>13202</td>
    </tr>
    <tr>
      <th>408</th>
      <td>Atlanta</td>
      <td>12365</td>
    </tr>
    <tr>
      <th>7650</th>
      <td>Phoenix</td>
      <td>12297</td>
    </tr>
    <tr>
      <th>1709</th>
      <td>Charlotte</td>
      <td>10419</td>
    </tr>
    <tr>
      <th>2056</th>
      <td>Columbus</td>
      <td>9323</td>
    </tr>
    <tr>
      <th>4684</th>
      <td>Indianapolis</td>
      <td>9235</td>
    </tr>
    <tr>
      <th>5632</th>
      <td>Los Angeles</td>
      <td>8878</td>
    </tr>
    <tr>
      <th>7641</th>
      <td>Philadelphia</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>1846</th>
      <td>Cincinnati</td>
      <td>7650</td>
    </tr>
    <tr>
      <th>10229</th>
      <td>Washington</td>
      <td>7619</td>
    </tr>
    <tr>
      <th>440</th>
      <td>Austin</td>
      <td>7294</td>
    </tr>
    <tr>
      <th>2518</th>
      <td>Denver</td>
      <td>7121</td>
    </tr>
    <tr>
      <th>8574</th>
      <td>San Antonio</td>
      <td>7107</td>
    </tr>
    <tr>
      <th>9551</th>
      <td>Tampa</td>
      <td>7076</td>
    </tr>
    <tr>
      <th>7291</th>
      <td>Orlando</td>
      <td>7008</td>
    </tr>
    <tr>
      <th>6299</th>
      <td>Minneapolis</td>
      <td>6811</td>
    </tr>
    <tr>
      <th>6634</th>
      <td>Nashville</td>
      <td>6751</td>
    </tr>
    <tr>
      <th>1031</th>
      <td>Boston</td>
      <td>6730</td>
    </tr>
    <tr>
      <th>5653</th>
      <td>Louisville</td>
      <td>6717</td>
    </tr>
    <tr>
      <th>4895</th>
      <td>Kansas City</td>
      <td>6717</td>
    </tr>
    <tr>
      <th>8585</th>
      <td>San Diego</td>
      <td>6699</td>
    </tr>
    <tr>
      <th>8589</th>
      <td>San Francisco</td>
      <td>6665</td>
    </tr>
    <tr>
      <th>6182</th>
      <td>Miami</td>
      <td>6659</td>
    </tr>
    <tr>
      <th>517</th>
      <td>Baltimore</td>
      <td>6472</td>
    </tr>
    <tr>
      <th>3513</th>
      <td>Fort Worth</td>
      <td>6157</td>
    </tr>
    <tr>
      <th>8783</th>
      <td>Seattle</td>
      <td>6112</td>
    </tr>
    <tr>
      <th>8524</th>
      <td>Saint Louis</td>
      <td>5582</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2265</th>
      <td>Cross Junction</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2263</th>
      <td>Cross</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7718</th>
      <td>Pinon</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7663</th>
      <td>Pierce City</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7662</th>
      <td>Pierce</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2284</th>
      <td>Crystal Bay</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2307</th>
      <td>Cuney</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7603</th>
      <td>Pequot Indian Res</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7605</th>
      <td>Percy</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7608</th>
      <td>Peridot</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7610</th>
      <td>Perkins</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7611</th>
      <td>Perrine</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2316</th>
      <td>Cushman</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2313</th>
      <td>Curtiss</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7622</th>
      <td>Pescadero</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2310</th>
      <td>Curtice</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2308</th>
      <td>Cunningham</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7635</th>
      <td>Peyton</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7658</th>
      <td>Pickwick</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2304</th>
      <td>Cumberland Foreside</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2300</th>
      <td>Culver</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7642</th>
      <td>Philippi</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7646</th>
      <td>Philmont</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7647</th>
      <td>Philo</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7649</th>
      <td>Philomont</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2293</th>
      <td>Cuchillo</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7652</th>
      <td>Picabo</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2290</th>
      <td>Crystola</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2286</th>
      <td>Crystal Hill</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>29 Palms</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>10913 rows × 2 columns</p>
</div>



**In [34]:**

{% highlight python %}
citywise_jobs = jobs_US.groupby(['City']).size().reset_index(name='Citywise').sort_values('Citywise', ascending=False)
{% endhighlight %}

**In [35]:**

{% highlight python %}
citywise_jobs
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
      <th>City</th>
      <th>Citywise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4564</th>
      <td>Houston</td>
      <td>19323</td>
    </tr>
    <tr>
      <th>6809</th>
      <td>New York</td>
      <td>18402</td>
    </tr>
    <tr>
      <th>1782</th>
      <td>Chicago</td>
      <td>17806</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>Dallas</td>
      <td>13202</td>
    </tr>
    <tr>
      <th>408</th>
      <td>Atlanta</td>
      <td>12365</td>
    </tr>
    <tr>
      <th>7650</th>
      <td>Phoenix</td>
      <td>12297</td>
    </tr>
    <tr>
      <th>1709</th>
      <td>Charlotte</td>
      <td>10419</td>
    </tr>
    <tr>
      <th>2056</th>
      <td>Columbus</td>
      <td>9323</td>
    </tr>
    <tr>
      <th>4684</th>
      <td>Indianapolis</td>
      <td>9235</td>
    </tr>
    <tr>
      <th>5632</th>
      <td>Los Angeles</td>
      <td>8878</td>
    </tr>
    <tr>
      <th>7641</th>
      <td>Philadelphia</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>1846</th>
      <td>Cincinnati</td>
      <td>7650</td>
    </tr>
    <tr>
      <th>10229</th>
      <td>Washington</td>
      <td>7619</td>
    </tr>
    <tr>
      <th>440</th>
      <td>Austin</td>
      <td>7294</td>
    </tr>
    <tr>
      <th>2518</th>
      <td>Denver</td>
      <td>7121</td>
    </tr>
    <tr>
      <th>8574</th>
      <td>San Antonio</td>
      <td>7107</td>
    </tr>
    <tr>
      <th>9551</th>
      <td>Tampa</td>
      <td>7076</td>
    </tr>
    <tr>
      <th>7291</th>
      <td>Orlando</td>
      <td>7008</td>
    </tr>
    <tr>
      <th>6299</th>
      <td>Minneapolis</td>
      <td>6811</td>
    </tr>
    <tr>
      <th>6634</th>
      <td>Nashville</td>
      <td>6751</td>
    </tr>
    <tr>
      <th>1031</th>
      <td>Boston</td>
      <td>6730</td>
    </tr>
    <tr>
      <th>5653</th>
      <td>Louisville</td>
      <td>6717</td>
    </tr>
    <tr>
      <th>4895</th>
      <td>Kansas City</td>
      <td>6717</td>
    </tr>
    <tr>
      <th>8585</th>
      <td>San Diego</td>
      <td>6699</td>
    </tr>
    <tr>
      <th>8589</th>
      <td>San Francisco</td>
      <td>6665</td>
    </tr>
    <tr>
      <th>6182</th>
      <td>Miami</td>
      <td>6659</td>
    </tr>
    <tr>
      <th>517</th>
      <td>Baltimore</td>
      <td>6472</td>
    </tr>
    <tr>
      <th>3513</th>
      <td>Fort Worth</td>
      <td>6157</td>
    </tr>
    <tr>
      <th>8783</th>
      <td>Seattle</td>
      <td>6112</td>
    </tr>
    <tr>
      <th>8524</th>
      <td>Saint Louis</td>
      <td>5582</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2265</th>
      <td>Cross Junction</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2263</th>
      <td>Cross</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7718</th>
      <td>Pinon</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7663</th>
      <td>Pierce City</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7662</th>
      <td>Pierce</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2284</th>
      <td>Crystal Bay</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2307</th>
      <td>Cuney</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7603</th>
      <td>Pequot Indian Res</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7605</th>
      <td>Percy</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7608</th>
      <td>Peridot</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7610</th>
      <td>Perkins</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7611</th>
      <td>Perrine</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2316</th>
      <td>Cushman</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2313</th>
      <td>Curtiss</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7622</th>
      <td>Pescadero</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2310</th>
      <td>Curtice</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2308</th>
      <td>Cunningham</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7635</th>
      <td>Peyton</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7658</th>
      <td>Pickwick</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2304</th>
      <td>Cumberland Foreside</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2300</th>
      <td>Culver</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7642</th>
      <td>Philippi</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7646</th>
      <td>Philmont</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7647</th>
      <td>Philo</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7649</th>
      <td>Philomont</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2293</th>
      <td>Cuchillo</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7652</th>
      <td>Picabo</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2290</th>
      <td>Crystola</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2286</th>
      <td>Crystal Hill</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>29 Palms</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>10913 rows × 2 columns</p>
</div>



**In [36]:**

{% highlight python %}
citywise_top = citywise_jobs.loc[citywise_jobs['Citywise']>=12]
{% endhighlight %}

**In [37]:**

{% highlight python %}
citywise_top
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
      <th>City</th>
      <th>Citywise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4564</th>
      <td>Houston</td>
      <td>19323</td>
    </tr>
    <tr>
      <th>6809</th>
      <td>New York</td>
      <td>18402</td>
    </tr>
    <tr>
      <th>1782</th>
      <td>Chicago</td>
      <td>17806</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>Dallas</td>
      <td>13202</td>
    </tr>
    <tr>
      <th>408</th>
      <td>Atlanta</td>
      <td>12365</td>
    </tr>
    <tr>
      <th>7650</th>
      <td>Phoenix</td>
      <td>12297</td>
    </tr>
    <tr>
      <th>1709</th>
      <td>Charlotte</td>
      <td>10419</td>
    </tr>
    <tr>
      <th>2056</th>
      <td>Columbus</td>
      <td>9323</td>
    </tr>
    <tr>
      <th>4684</th>
      <td>Indianapolis</td>
      <td>9235</td>
    </tr>
    <tr>
      <th>5632</th>
      <td>Los Angeles</td>
      <td>8878</td>
    </tr>
    <tr>
      <th>7641</th>
      <td>Philadelphia</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>1846</th>
      <td>Cincinnati</td>
      <td>7650</td>
    </tr>
    <tr>
      <th>10229</th>
      <td>Washington</td>
      <td>7619</td>
    </tr>
    <tr>
      <th>440</th>
      <td>Austin</td>
      <td>7294</td>
    </tr>
    <tr>
      <th>2518</th>
      <td>Denver</td>
      <td>7121</td>
    </tr>
    <tr>
      <th>8574</th>
      <td>San Antonio</td>
      <td>7107</td>
    </tr>
    <tr>
      <th>9551</th>
      <td>Tampa</td>
      <td>7076</td>
    </tr>
    <tr>
      <th>7291</th>
      <td>Orlando</td>
      <td>7008</td>
    </tr>
    <tr>
      <th>6299</th>
      <td>Minneapolis</td>
      <td>6811</td>
    </tr>
    <tr>
      <th>6634</th>
      <td>Nashville</td>
      <td>6751</td>
    </tr>
    <tr>
      <th>1031</th>
      <td>Boston</td>
      <td>6730</td>
    </tr>
    <tr>
      <th>5653</th>
      <td>Louisville</td>
      <td>6717</td>
    </tr>
    <tr>
      <th>4895</th>
      <td>Kansas City</td>
      <td>6717</td>
    </tr>
    <tr>
      <th>8585</th>
      <td>San Diego</td>
      <td>6699</td>
    </tr>
    <tr>
      <th>8589</th>
      <td>San Francisco</td>
      <td>6665</td>
    </tr>
    <tr>
      <th>6182</th>
      <td>Miami</td>
      <td>6659</td>
    </tr>
    <tr>
      <th>517</th>
      <td>Baltimore</td>
      <td>6472</td>
    </tr>
    <tr>
      <th>3513</th>
      <td>Fort Worth</td>
      <td>6157</td>
    </tr>
    <tr>
      <th>8783</th>
      <td>Seattle</td>
      <td>6112</td>
    </tr>
    <tr>
      <th>8524</th>
      <td>Saint Louis</td>
      <td>5582</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1098</th>
      <td>Brandenburg</td>
      <td>12</td>
    </tr>
    <tr>
      <th>7699</th>
      <td>Pine Hills</td>
      <td>12</td>
    </tr>
    <tr>
      <th>8635</th>
      <td>Sandpoint</td>
      <td>12</td>
    </tr>
    <tr>
      <th>4920</th>
      <td>Keasbey</td>
      <td>12</td>
    </tr>
    <tr>
      <th>8033</th>
      <td>Quitman</td>
      <td>12</td>
    </tr>
    <tr>
      <th>698</th>
      <td>Bel Aire</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2874</th>
      <td>East Walpole</td>
      <td>12</td>
    </tr>
    <tr>
      <th>6120</th>
      <td>Mendota</td>
      <td>12</td>
    </tr>
    <tr>
      <th>5004</th>
      <td>Key Largo</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1337</th>
      <td>Bushnell</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2988</th>
      <td>Eldersburg</td>
      <td>12</td>
    </tr>
    <tr>
      <th>8862</th>
      <td>Sharpsburg</td>
      <td>12</td>
    </tr>
    <tr>
      <th>9815</th>
      <td>Tucumcari</td>
      <td>12</td>
    </tr>
    <tr>
      <th>8170</th>
      <td>Republic</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1364</th>
      <td>Cadiz</td>
      <td>12</td>
    </tr>
    <tr>
      <th>5580</th>
      <td>Logan Township</td>
      <td>12</td>
    </tr>
    <tr>
      <th>8809</th>
      <td>Selmer</td>
      <td>12</td>
    </tr>
    <tr>
      <th>5464</th>
      <td>Lewisboro</td>
      <td>12</td>
    </tr>
    <tr>
      <th>10768</th>
      <td>Woodbine</td>
      <td>12</td>
    </tr>
    <tr>
      <th>7842</th>
      <td>Pontotoc</td>
      <td>12</td>
    </tr>
    <tr>
      <th>5909</th>
      <td>Marriottsville</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3347</th>
      <td>Fleetwood</td>
      <td>12</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Albertson</td>
      <td>12</td>
    </tr>
    <tr>
      <th>5463</th>
      <td>Lewisberry</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3499</th>
      <td>Fort Sill</td>
      <td>12</td>
    </tr>
    <tr>
      <th>6838</th>
      <td>Newport Coast</td>
      <td>12</td>
    </tr>
    <tr>
      <th>4655</th>
      <td>Ijamsville</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2510</th>
      <td>Denmark</td>
      <td>12</td>
    </tr>
    <tr>
      <th>9904</th>
      <td>University Heights</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1192</th>
      <td>Brodheadsville</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
<p>4430 rows × 2 columns</p>
</div>



**In [38]:**

{% highlight python %}
plt.figure(figsize=(12,6))
ax = sns.barplot(x="City", y="Citywise",data=citywise_top.head(50))
ax.set_xticklabels(ax.get_xticklabels(), rotation=90, ha="right")
ax.set_title('City wise job openings')
plt.tight_layout()
plt.show()
{% endhighlight %}

 
![png]({{ BASE_PATH }}/images/job_recommender_51_0.png) 

 
- jobs_US
- statewise_jobs
- citywise_jobs
- citywise_jobs_top 
 
### User profile based on location 

**In [39]:**

{% highlight python %}
users_training.groupby(['Country']).size().reset_index(name='Locationwise').sort_values('Locationwise', ascending=False)
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
      <th>Country</th>
      <th>Locationwise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>105</th>
      <td>US</td>
      <td>365740</td>
    </tr>
    <tr>
      <th>47</th>
      <td>IN</td>
      <td>236</td>
    </tr>
    <tr>
      <th>18</th>
      <td>CA</td>
      <td>108</td>
    </tr>
    <tr>
      <th>81</th>
      <td>PH</td>
      <td>59</td>
    </tr>
    <tr>
      <th>82</th>
      <td>PK</td>
      <td>59</td>
    </tr>
    <tr>
      <th>71</th>
      <td>MX</td>
      <td>48</td>
    </tr>
    <tr>
      <th>103</th>
      <td>UK</td>
      <td>39</td>
    </tr>
    <tr>
      <th>31</th>
      <td>ES</td>
      <td>37</td>
    </tr>
    <tr>
      <th>33</th>
      <td>FR</td>
      <td>31</td>
    </tr>
    <tr>
      <th>50</th>
      <td>IT</td>
      <td>29</td>
    </tr>
    <tr>
      <th>0</th>
      <td>AE</td>
      <td>24</td>
    </tr>
    <tr>
      <th>89</th>
      <td>SA</td>
      <td>19</td>
    </tr>
    <tr>
      <th>30</th>
      <td>EG</td>
      <td>19</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BR</td>
      <td>18</td>
    </tr>
    <tr>
      <th>92</th>
      <td>SG</td>
      <td>17</td>
    </tr>
    <tr>
      <th>113</th>
      <td>ZA</td>
      <td>14</td>
    </tr>
    <tr>
      <th>21</th>
      <td>CO</td>
      <td>14</td>
    </tr>
    <tr>
      <th>108</th>
      <td>VE</td>
      <td>13</td>
    </tr>
    <tr>
      <th>38</th>
      <td>GR</td>
      <td>13</td>
    </tr>
    <tr>
      <th>25</th>
      <td>DE</td>
      <td>12</td>
    </tr>
    <tr>
      <th>52</th>
      <td>JO</td>
      <td>11</td>
    </tr>
    <tr>
      <th>49</th>
      <td>IR</td>
      <td>11</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BD</td>
      <td>10</td>
    </tr>
    <tr>
      <th>27</th>
      <td>DO</td>
      <td>10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AR</td>
      <td>10</td>
    </tr>
    <tr>
      <th>91</th>
      <td>SE</td>
      <td>10</td>
    </tr>
    <tr>
      <th>86</th>
      <td>RO</td>
      <td>9</td>
    </tr>
    <tr>
      <th>20</th>
      <td>CN</td>
      <td>9</td>
    </tr>
    <tr>
      <th>53</th>
      <td>JP</td>
      <td>9</td>
    </tr>
    <tr>
      <th>84</th>
      <td>QA</td>
      <td>8</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>BA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>BB</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BE</td>
      <td>1</td>
    </tr>
    <tr>
      <th>94</th>
      <td>TC</td>
      <td>1</td>
    </tr>
    <tr>
      <th>45</th>
      <td>IE</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>BW</td>
      <td>1</td>
    </tr>
    <tr>
      <th>64</th>
      <td>LT</td>
      <td>1</td>
    </tr>
    <tr>
      <th>42</th>
      <td>HT</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>IQ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>41</th>
      <td>HR</td>
      <td>1</td>
    </tr>
    <tr>
      <th>37</th>
      <td>GM</td>
      <td>1</td>
    </tr>
    <tr>
      <th>55</th>
      <td>KH</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AF</td>
      <td>1</td>
    </tr>
    <tr>
      <th>59</th>
      <td>KZ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>60</th>
      <td>LA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>61</th>
      <td>LB</td>
      <td>1</td>
    </tr>
    <tr>
      <th>62</th>
      <td>LC</td>
      <td>1</td>
    </tr>
    <tr>
      <th>65</th>
      <td>LU</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BY</td>
      <td>1</td>
    </tr>
    <tr>
      <th>66</th>
      <td>LV</td>
      <td>1</td>
    </tr>
    <tr>
      <th>68</th>
      <td>MK</td>
      <td>1</td>
    </tr>
    <tr>
      <th>69</th>
      <td>MR</td>
      <td>1</td>
    </tr>
    <tr>
      <th>70</th>
      <td>MU</td>
      <td>1</td>
    </tr>
    <tr>
      <th>34</th>
      <td>GA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>73</th>
      <td>NC</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>DK</td>
      <td>1</td>
    </tr>
    <tr>
      <th>79</th>
      <td>PA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>CY</td>
      <td>1</td>
    </tr>
    <tr>
      <th>85</th>
      <td>RE</td>
      <td>1</td>
    </tr>
    <tr>
      <th>80</th>
      <td>PE</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>115 rows × 2 columns</p>
</div>



**In [40]:**

{% highlight python %}
users_training_US = users_training.loc[users_training['Country'] == 'US']
users_training_US.head()
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
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>ZipCode</th>
      <th>DegreeType</th>
      <th>Major</th>
      <th>GraduationDate</th>
      <th>WorkHistoryCount</th>
      <th>TotalYearsExperience</th>
      <th>CurrentlyEmployed</th>
      <th>ManagedOthers</th>
      <th>ManagedHowMany</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>1</td>
      <td>Train</td>
      <td>Paramount</td>
      <td>CA</td>
      <td>US</td>
      <td>90723</td>
      <td>High School</td>
      <td>NaN</td>
      <td>1999-06-01 00:00:00</td>
      <td>3</td>
      <td>10.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>72</td>
      <td>1</td>
      <td>Train</td>
      <td>La Mesa</td>
      <td>CA</td>
      <td>US</td>
      <td>91941</td>
      <td>Master's</td>
      <td>Anthropology</td>
      <td>2011-01-01 00:00:00</td>
      <td>10</td>
      <td>8.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>80</td>
      <td>1</td>
      <td>Train</td>
      <td>Williamstown</td>
      <td>NJ</td>
      <td>US</td>
      <td>08094</td>
      <td>High School</td>
      <td>Not Applicable</td>
      <td>1985-06-01 00:00:00</td>
      <td>5</td>
      <td>11.0</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>98</td>
      <td>1</td>
      <td>Train</td>
      <td>Astoria</td>
      <td>NY</td>
      <td>US</td>
      <td>11105</td>
      <td>Master's</td>
      <td>Journalism</td>
      <td>2007-05-01 00:00:00</td>
      <td>3</td>
      <td>3.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>123</td>
      <td>1</td>
      <td>Train</td>
      <td>Baton Rouge</td>
      <td>LA</td>
      <td>US</td>
      <td>70808</td>
      <td>Bachelor's</td>
      <td>Agricultural Business</td>
      <td>2011-05-01 00:00:00</td>
      <td>1</td>
      <td>9.0</td>
      <td>Yes</td>
      <td>No</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



**In [41]:**

{% highlight python %}
users_training_US.shape
{% endhighlight %}




    (365740, 15)



**In [42]:**

{% highlight python %}
users_training_statewise = users_training_US.groupby('State').size().reset_index(
    name='statewise').sort_values('statewise',ascending=False)
users_training_statewise.head()
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
      <th>State</th>
      <th>statewise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11</th>
      <td>FL</td>
      <td>40381</td>
    </tr>
    <tr>
      <th>47</th>
      <td>TX</td>
      <td>33260</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CA</td>
      <td>31141</td>
    </tr>
    <tr>
      <th>17</th>
      <td>IL</td>
      <td>22557</td>
    </tr>
    <tr>
      <th>37</th>
      <td>NY</td>
      <td>19299</td>
    </tr>
  </tbody>
</table>
</div>



**In [43]:**

{% highlight python %}
users_training_statewise_top = users_training_statewise.loc[users_training_statewise['statewise'] >= 12]
{% endhighlight %}

**In [44]:**

{% highlight python %}
plt.figure(figsize=(12,6))
ax = sns.barplot(x='State', y='statewise', data=users_training_statewise_top)
ax.set_xticklabels(ax.get_xticklabels(), rotation=90, ha='right')
ax.set_title('State wise job seekers')
plt.tight_layout()
plt.show()
{% endhighlight %}

 
![png]({{ BASE_PATH }}/images/job_recommender_59_0.png) 


**In [45]:**

{% highlight python %}
users_training_citywise = users_training_US.groupby(['City']).size().reset_index(
    name='citywise').sort_values('citywise',ascending=False)
users_training_citywise.head()
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
      <th>City</th>
      <th>citywise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1528</th>
      <td>Chicago</td>
      <td>6964</td>
    </tr>
    <tr>
      <th>4066</th>
      <td>Houston</td>
      <td>5487</td>
    </tr>
    <tr>
      <th>4177</th>
      <td>Indianapolis</td>
      <td>4450</td>
    </tr>
    <tr>
      <th>5604</th>
      <td>Miami</td>
      <td>4359</td>
    </tr>
    <tr>
      <th>6965</th>
      <td>Philadelphia</td>
      <td>4347</td>
    </tr>
  </tbody>
</table>
</div>



**In [46]:**

{% highlight python %}
users_training_citywise_top = users_training_citywise.loc[users_training_citywise['citywise'] >= 12]
users_training_citywise_top.head()
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
      <th>City</th>
      <th>citywise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1528</th>
      <td>Chicago</td>
      <td>6964</td>
    </tr>
    <tr>
      <th>4066</th>
      <td>Houston</td>
      <td>5487</td>
    </tr>
    <tr>
      <th>4177</th>
      <td>Indianapolis</td>
      <td>4450</td>
    </tr>
    <tr>
      <th>5604</th>
      <td>Miami</td>
      <td>4359</td>
    </tr>
    <tr>
      <th>6965</th>
      <td>Philadelphia</td>
      <td>4347</td>
    </tr>
  </tbody>
</table>
</div>



**In [47]:**

{% highlight python %}
plt.figure(figsize=(12,6))
ax = sns.barplot(x='City', y='citywise', data=users_training_citywise_top.head(50))
ax.set_xticklabels(ax.get_xticklabels(), rotation=90, ha="right")
ax.set_title('City wise job seekers')
plt.tight_layout()
plt.show()
{% endhighlight %}

 
![png]({{ BASE_PATH }}/images/job_recommender_62_0.png) 

 
- users_training_US
- users_training_statewise
- users_training_citywise
- users_training_citywise_top 

**In [48]:**

{% highlight python %}
import ast 
from scipy import stats
from ast import literal_eval
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
from sklearn.metrics.pairwise import linear_kernel, cosine_similarity
# from nltk.stem.snowball import SnowballStemmer
# from nltk.stem.wordnet import WordNetLemmatizer
# from nltk.corpus import wordnet
# from surprise import Reader, Dataset, SVD, evaluate
{% endhighlight %}
 
## Building model <a class="anchor" id="model"></a> 

**In [49]:**

{% highlight python %}
jobs_US.columns
{% endhighlight %}




    Index(['JobID', 'WindowID', 'Title', 'Description', 'Requirements', 'City',
           'State', 'Country', 'Zip5', 'StartDate', 'EndDate'],
          dtype='object')



**In [50]:**

{% highlight python %}
jobs_US.head().transpose()
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>JobID</th>
      <td>1</td>
      <td>4</td>
      <td>7</td>
      <td>8</td>
      <td>9</td>
    </tr>
    <tr>
      <th>WindowID</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Title</th>
      <td>Security Engineer/Technical Lead</td>
      <td>SAP Business Analyst / WM</td>
      <td>P/T HUMAN RESOURCES ASSISTANT</td>
      <td>Route Delivery Drivers</td>
      <td>Housekeeping</td>
    </tr>
    <tr>
      <th>Description</th>
      <td>&lt;p&gt;Security Clearance Required:&amp;nbsp; Top Secr...</td>
      <td>&lt;strong&gt;NO Corp. to Corp resumes&amp;nbsp;are bein...</td>
      <td>&lt;b&gt;    &lt;b&gt; P/T HUMAN RESOURCES ASSISTANT&lt;/b&gt; &lt;...</td>
      <td>CITY BEVERAGES Come to work for the best in th...</td>
      <td>I make  sure every part of their day is magica...</td>
    </tr>
    <tr>
      <th>Requirements</th>
      <td>&lt;p&gt;SKILL SET&lt;/p&gt;\r&lt;p&gt;&amp;nbsp;&lt;/p&gt;\r&lt;p&gt;Network Se...</td>
      <td>&lt;p&gt;&lt;b&gt;WHAT YOU NEED: &lt;/b&gt;&lt;/p&gt;\r&lt;p&gt;Four year co...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Please refer to the Job Description to view th...</td>
    </tr>
    <tr>
      <th>City</th>
      <td>Washington</td>
      <td>Charlotte</td>
      <td>Winter Park</td>
      <td>Orlando</td>
      <td>Orlando</td>
    </tr>
    <tr>
      <th>State</th>
      <td>DC</td>
      <td>NC</td>
      <td>FL</td>
      <td>FL</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>Country</th>
      <td>US</td>
      <td>US</td>
      <td>US</td>
      <td>US</td>
      <td>US</td>
    </tr>
    <tr>
      <th>Zip5</th>
      <td>20531</td>
      <td>28217</td>
      <td>32792</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>StartDate</th>
      <td>2012-03-07 13:17:01.643</td>
      <td>2012-03-21 02:03:44.137</td>
      <td>2012-03-02 16:36:55.447</td>
      <td>2012-03-03 09:01:10.077</td>
      <td>2012-03-03 09:01:11.88</td>
    </tr>
    <tr>
      <th>EndDate</th>
      <td>2012-04-06 23:59:59</td>
      <td>2012-04-20 23:59:59</td>
      <td>2012-04-01 23:59:59</td>
      <td>2012-04-02 23:59:59</td>
      <td>2012-04-02 23:59:59</td>
    </tr>
  </tbody>
</table>
</div>



**In [51]:**

{% highlight python %}
jobs_US_base_line = jobs_US.iloc[0:10000,0:8]
{% endhighlight %}

**In [52]:**

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
      <td>&lt;p&gt;Security Clearance Required:&amp;nbsp; Top Secr...</td>
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
      <td>&lt;strong&gt;NO Corp. to Corp resumes&amp;nbsp;are bein...</td>
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
      <td>&lt;b&gt;    &lt;b&gt; P/T HUMAN RESOURCES ASSISTANT&lt;/b&gt; &lt;...</td>
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
      <td>CITY BEVERAGES Come to work for the best in th...</td>
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
      <td>I make  sure every part of their day is magica...</td>
      <td>Please refer to the Job Description to view th...</td>
      <td>Orlando</td>
      <td>FL</td>
      <td>US</td>
    </tr>
  </tbody>
</table>
</div>



**In [53]:**

{% highlight python %}
jobs_US_base_line['Title'] = jobs_US_base_line['Title'].fillna('')
jobs_US_base_line['Description'] = jobs_US_base_line['Description'].fillna('')
#jobs_US_base_line['Requirements'] = jobs_US_base_line['Requirements'].fillna('')

jobs_US_base_line['Description'] = jobs_US_base_line['Title'] + jobs_US_base_line['Description']
{% endhighlight %}
 
## Clean html <a class="anchor" id="html"></a> 

**In [54]:**

{% highlight python %}
jobs_US_base_line.loc[0, 'Requirements']
{% endhighlight %}




    '<p>SKILL SET</p>\\r<p>&nbsp;</p>\\r<p>Network Security tools:</p>\\r<p>&nbsp;</p>\\r<p>Webdefend Web Application Firewall (WAF), Cisco Routers, Fortigate 3800 Firewall series, Palo Alto 4000 firewall series, Cisco ASA 5xx Firewall Platform, Cisco&nbsp; FWSM,&nbsp; SourceFire Defense Center, SourceFire IP Sensor Platform, BlueCoat SG Appliance, F5 BigIP(reverse proxy).</p>\\r<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </p>\\r<p>Web Application tools:&nbsp; </p>\\r<p>&nbsp;</p>\\r<p>AppDective, Fortify SCA, HP WebInspect, and the like.</p>\\r<p>&nbsp;</p>\\r<p>Network Vulnerability tools:&nbsp; </p>\\r<p>&nbsp;</p>\\r<p>Tenable Security Center, McAfee Foundstone scanner, Cain and Able, L0phtcrack - Password Cracker, Nessus Vulnerability Scanner, NMAP &ndash; Port Scanner, and other scanning and vulnerability mapping tools.&nbsp; </p>\\r<p>&nbsp;</p>\\r<p>&nbsp;</p>\\r<p>DESIRABLE SKILLS:</p>\\r<p>&nbsp;</p>\\r<p>CISSP and/or related Certifications</p>\\r<p>&nbsp;</p>\\r<p>EDUCATION AND YEARS OF EXPERIENCE:</p>\\r<p>&nbsp;</p>\\r<p>BS Computer Science or related discipline; minimum of 8 years in IT Security; minimum 4 years in Senior/Lead position</p>\\r<p>&nbsp;</p>\\r<p><a href="https://www.tmrhq.com/jobapplicationstep1.aspx"><span>APPLY HERE</span></a></p>'



**In [55]:**

{% highlight python %}
import re

# def cleanhtml(raw_html):
#     cleanr = re.compile('<.*?>')
#     cleantext = re.sub(cleanr, '', raw_html)
#     return cleantext

# cleanhtml(jobs.loc[0, 'Requirements'])


def preprocessor(text):
    text = text.replace('\\r', '').replace('&nbsp', '').replace('\n', '')
    text = re.sub('<[^>]*>', '', text)
    emoticons = re.findall('(?::|;|=)(?:-)?(?:\)|\(|D|P)', text)
    text = re.sub('[\W]+', ' ', text.lower()) +\
        ' '.join(emoticons).replace('-', '')
    return text

preprocessor(jobs_US_base_line.loc[0, 'Requirements'])
{% endhighlight %}




    'skill set network security tools webdefend web application firewall waf cisco routers fortigate 3800 firewall series palo alto 4000 firewall series cisco asa 5xx firewall platform cisco fwsm sourcefire defense center sourcefire ip sensor platform bluecoat sg appliance f5 bigip reverse proxy web application tools appdective fortify sca hp webinspect and the like network vulnerability tools tenable security center mcafee foundstone scanner cain and able l0phtcrack password cracker nessus vulnerability scanner nmap ndash port scanner and other scanning and vulnerability mapping tools desirable skills cissp and or related certifications education and years of experience bs computer science or related discipline minimum of 8 years in it security minimum 4 years in senior lead position apply here;D'



**In [56]:**

{% highlight python %}
jobs_US_base_line['Description'] = jobs_US_base_line['Description'].astype(dtype='str').apply(preprocessor)
{% endhighlight %}

**In [57]:**

{% highlight python %}
jobs_US_base_line.loc[0,'Description']
{% endhighlight %}




    'security engineer technical leadsecurity clearance required top secret job number tmr 447location of job washington dctmr inc is an equal employment opportunity companyfor more job opportunities with tmr visit our website www tmrhq comsend resumes to hr tmrhq2 com job summary leads the customer rsquo s overall cyber security strategy formalizes service offerings consisted with itil best practices and provides design and architecture support provide security design architecture support for ojp rsquo s it security division itsd leads the secops team in the day to day ojp security operations support provides direction when needed in a security incident or technical issues works in concert with network operations on design integration for best security posture supports business development functions including capture management proposal development and responses and other initiatives to include conferences trade shows webinars developing white papers and the like identifies resources and mentors in house talent to ensure tmr remains responsive to growing initiatives and contracts with qualified personnel '


 
## Dataset 
 
**From here onwards use `jobs_US_base_line` data frame to work on, which is
selected by `jobs_US.iloc[0:10000,0:8]`.** 

**In [58]:**

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


 
### Tokenising <a class="anchor" id="token"></a> 

**In [59]:**

{% highlight python %}
from nltk.stem.porter import PorterStemmer
porter = PorterStemmer()

def tokenizer_porter(text):
    return [porter.stem(word) for word in text.split()]

tokenizer_porter('runners like running and thus they run')
{% endhighlight %}




    ['runner', 'like', 'run', 'and', 'thu', 'they', 'run']



**In [60]:**

{% highlight python %}
jobs_US_base_line_token = jobs_US_base_line.copy()
{% endhighlight %}

**In [61]:**

{% highlight python %}
jobs_US_base_line_token['Description'] = jobs_US_base_line_token['Description'].apply(tokenizer_porter)
{% endhighlight %}

**In [62]:**

{% highlight python %}
jobs_US_base_line_token.loc[0, 'Description']
{% endhighlight %}




    ['secur',
     'engin',
     'technic',
     'leadsecur',
     'clearanc',
     'requir',
     'top',
     'secret',
     'job',
     'number',
     'tmr',
     '447locat',
     'of',
     'job',
     'washington',
     'dctmr',
     'inc',
     'is',
     'an',
     'equal',
     'employ',
     'opportun',
     'companyfor',
     'more',
     'job',
     'opportun',
     'with',
     'tmr',
     'visit',
     'our',
     'websit',
     'www',
     'tmrhq',
     'comsend',
     'resum',
     'to',
     'hr',
     'tmrhq2',
     'com',
     'job',
     'summari',
     'lead',
     'the',
     'custom',
     'rsquo',
     's',
     'overal',
     'cyber',
     'secur',
     'strategi',
     'formal',
     'servic',
     'offer',
     'consist',
     'with',
     'itil',
     'best',
     'practic',
     'and',
     'provid',
     'design',
     'and',
     'architectur',
     'support',
     'provid',
     'secur',
     'design',
     'architectur',
     'support',
     'for',
     'ojp',
     'rsquo',
     's',
     'it',
     'secur',
     'divis',
     'itsd',
     'lead',
     'the',
     'secop',
     'team',
     'in',
     'the',
     'day',
     'to',
     'day',
     'ojp',
     'secur',
     'oper',
     'support',
     'provid',
     'direct',
     'when',
     'need',
     'in',
     'a',
     'secur',
     'incid',
     'or',
     'technic',
     'issu',
     'work',
     'in',
     'concert',
     'with',
     'network',
     'oper',
     'on',
     'design',
     'integr',
     'for',
     'best',
     'secur',
     'postur',
     'support',
     'busi',
     'develop',
     'function',
     'includ',
     'captur',
     'manag',
     'propos',
     'develop',
     'and',
     'respons',
     'and',
     'other',
     'initi',
     'to',
     'includ',
     'confer',
     'trade',
     'show',
     'webinar',
     'develop',
     'white',
     'paper',
     'and',
     'the',
     'like',
     'identifi',
     'resourc',
     'and',
     'mentor',
     'in',
     'hous',
     'talent',
     'to',
     'ensur',
     'tmr',
     'remain',
     'respons',
     'to',
     'grow',
     'initi',
     'and',
     'contract',
     'with',
     'qualifi',
     'personnel']


 
### Word2vec 

**In [63]:**

{% highlight python %}
from gensim.models import word2vec
import logging
{% endhighlight %}


    ---------------------------------------------------------------------------

    ModuleNotFoundError                       Traceback (most recent call last)

    <ipython-input-63-17f6cbda934f> in <module>()
    ----> 1 from gensim.models import word2vec
          2 import logging


    ModuleNotFoundError: No module named 'gensim'


**In [None]:**

{% highlight python %}
logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
{% endhighlight %}
 
## Text processing <a class="anchor" id="textprocessing"></a> 
 
consider [modifying count vectoriser](https://towardsdatascience.com/hacking-
scikit-learns-vectorizers-9ef26a7170af). 
 
### Bag-of-words (BOW) model <a class="anchor" id="bow"></a> 
 
#### unigram 

**In [None]:**

{% highlight python %}
count = CountVectorizer()
bag = count.fit_transform(jobs_US_base_line['Description'])
{% endhighlight %}

**In [None]:**

{% highlight python %}
# print(count.vocabulary_)
{% endhighlight %}

**In [None]:**

{% highlight python %}
print(bag.toarray())
{% endhighlight %}
 
#### 2-gram 

**In [None]:**

{% highlight python %}
# Try 2-gram representation (use another instance of CountVectorizer)

count2 = CountVectorizer(ngram_range = (2, 2))
test_base = jobs_US_base_line.iloc[:100]
bag = count2.fit_transform(test_base['Description'])
{% endhighlight %}

**In [None]:**

{% highlight python %}
# print(count2.vocabulary_)
{% endhighlight %}

**In [None]:**

{% highlight python %}
print(bag.toarray())
{% endhighlight %}
 
Build [classifier](https://towardsdatascience.com/natural-language-processing-
count-vectorization-with-scikit-learn-e7804269bb5e). Remove stopwords. 
 
## Job description based recommender 
 
### Term frequency-inverse document frequency <a class="anchor" id="tfidf"></a> 

**In [None]:**

{% highlight python %}
tf = TfidfVectorizer(analyzer='word',ngram_range=(1, 2),min_df=0, stop_words='english')
tfidf_matrix = tf.fit_transform(jobs_US_base_line['Description'])
{% endhighlight %}

**In [None]:**

{% highlight python %}
tfidf_matrix.shape
{% endhighlight %}

**In [None]:**

{% highlight python %}
print(tfidf_matrix)
{% endhighlight %}

**In [None]:**

{% highlight python %}
jobs_US_base_line.loc[0,'Description']
{% endhighlight %}

**In [None]:**

{% highlight python %}
cosine_sim = linear_kernel(tfidf_matrix, tfidf_matrix)
{% endhighlight %}

**In [None]:**

{% highlight python %}
cosine_sim[0]
{% endhighlight %}

**In [None]:**

{% highlight python %}
jobs_US_base_line = jobs_US_base_line.reset_index()
titles = jobs_US_base_line['Title']
indices = pd.Series(jobs_US_base_line.index, index=jobs_US_base_line['Title'])
{% endhighlight %}

**In [None]:**

{% highlight python %}
jobs_US_base_line.head()
{% endhighlight %}

**In [None]:**

{% highlight python %}
print(indices)
{% endhighlight %}

**In [None]:**

{% highlight python %}
def get_recommendations(title):
    idx = indices[title]
    #print (idx)
    sim_scores = list(enumerate(cosine_sim[idx]))
    #print (sim_scores)
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    job_indices = [i[0] for i in sim_scores]
    return titles.iloc[job_indices]
{% endhighlight %}

**In [None]:**

{% highlight python %}
get_recommendations('SAP Business Analyst / WM').head(10)
{% endhighlight %}

**In [None]:**

{% highlight python %}
get_recommendations('Security Engineer/Technical Lead').head(10)
{% endhighlight %}
 
### similar users <a class="anchor" id="similarusers"></a>

- degree type, majors and total years of experience
- `users_training` dataset 

**In [None]:**

{% highlight python %}
users_training.head()
{% endhighlight %}

**In [None]:**

{% highlight python %}
user_based_approach_US = users_training.loc[users_training['Country']=='US']
{% endhighlight %}

**In [None]:**

{% highlight python %}
user_based_approach = user_based_approach_US.iloc[0:10000,:].copy()
{% endhighlight %}

**In [None]:**

{% highlight python %}
user_based_approach.head()
{% endhighlight %}

**In [None]:**

{% highlight python %}
user_based_approach['DegreeType'] = user_based_approach['DegreeType'].fillna('')
user_based_approach['Major'] = user_based_approach['Major'].fillna('')
user_based_approach['TotalYearsExperience'] = str(user_based_approach['TotalYearsExperience'].fillna(''))

user_based_approach['DegreeType'] = user_based_approach['DegreeType'] + user_based_approach['Major'] + \
                                    user_based_approach['TotalYearsExperience']

{% endhighlight %}

**In [None]:**

{% highlight python %}
tf = TfidfVectorizer(analyzer='word',ngram_range=(1, 2),min_df=0, stop_words='english')
tfidf_matrix = tf.fit_transform(user_based_approach['DegreeType'])
{% endhighlight %}

**In [None]:**

{% highlight python %}
tfidf_matrix.shape
{% endhighlight %}

**In [None]:**

{% highlight python %}
cosine_sim = linear_kernel(tfidf_matrix,tfidf_matrix)
{% endhighlight %}

**In [None]:**

{% highlight python %}
cosine_sim[0]
{% endhighlight %}

**In [None]:**

{% highlight python %}
user_based_approach = user_based_approach.reset_index()
userid = user_based_approach['UserID']
indices = pd.Series(user_based_approach.index, index=user_based_approach['UserID'])
indices.head(2)
{% endhighlight %}

**In [None]:**

{% highlight python %}
def get_recommendations_userwise(userid):
    idx = indices[userid]
    #print (idx)
    sim_scores = list(enumerate(cosine_sim[idx]))
    #print (sim_scores)
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    user_indices = [i[0] for i in sim_scores]
    #print (user_indices)
    return user_indices[0:11]
{% endhighlight %}

**In [None]:**

{% highlight python %}
get_recommendations_userwise(123)
{% endhighlight %}

**In [None]:**

{% highlight python %}
def get_job_id(usrid_list):
    jobs_userwise = apps_training['UserID'].isin(usrid_list) #
    df1 = pd.DataFrame(data = apps_training[jobs_userwise], columns=['JobID'])
    joblist = df1['JobID'].tolist()
    Job_list = jobs['JobID'].isin(joblist) #[1083186, 516837, 507614, 754917, 686406, 1058896, 335132])
    df_temp = pd.DataFrame(data = jobs[Job_list], columns=['JobID','Title','Description','City','State'])
    return df_temp
{% endhighlight %}

**In [None]:**

{% highlight python %}
get_job_id(get_recommendations_userwise(47))
{% endhighlight %}
 
## Benchmark

Predicts that a user will apply to the most popular jobs in his/her city, and
then to the most popular jobs in his/her state. 

**In [None]:**

{% highlight python %}
import sys

# These are the usual ipython objects, including this one you are creating
ipython_vars = ['In', 'Out', 'exit', 'quit', 'get_ipython', 'ipython_vars']

# Get a sorted list of the objects and their sizes
sorted([(x, sys.getsizeof(globals().get(x))) for x in dir() if not x.startswith('_') and x not in sys.modules and x not in ipython_vars], key=lambda x: x[1], reverse=True)
{% endhighlight %}
 
## Bad lines 

**In [None]:**

{% highlight python %}
line = pd.read_csv('data/jobs.tsv', sep='\t', skiprows=122432, nrows=1, header=None)
line.head()
{% endhighlight %}

**In [None]:**

{% highlight python %}
line[4]
{% endhighlight %}

**In [None]:**

{% highlight python %}
# file = open("double.txt","w")
np.savetxt("double.txt", line.values, fmt='%s')
{% endhighlight %}

**In [None]:**

{% highlight python %}
line = pd.read_csv('data/jobs.tsv', sep='\t', skiprows=1, nrows=1, header=None)
line.head()
{% endhighlight %}
