---
title: Job Recommendation Engine
desc: Recommendsation engine.
published: true
date: 2018-11-23
categories: project
tags: intern notebook recommendation data-science
---
## Introduction to Recommender
Mainly there's two types of recommender, collaborative and content-based recommender.

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

## Building a Job Recommender
### Dataset
Download from [Careerbuilder](https://www.kaggle.com/c/job-recommendation)
https://github.com/blennon/kaggle/tree/master/careerbuilder

### TOC
* [Import dependencies](#import)
* [Load dataset](#dataset)
* [EDA and Preprocessing](#EDA)
 - [split into training and testing dataset](#split)
 - [location](#location)
 - [preprocessing](#preprocessing)
* [Building model](#model)
* [Clean html](#html)
* [Content based filtering](#content)
 - [job description based recommender](#jobdesc)
 - [similar user based recommender](#similarusers)
* [Collaborative filtering](#collab) 
 
### Import dependencies <a class="anchor" id="import"></a> 

**In [1]:**

{% highlight python %}
%matplotlib inline
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np
{% endhighlight %}
 
### Load dataset <a class="anchor" id="dataset"></a> 

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


 
### EDA and Preprocessing <a class="anchor" id="EDA"></a> 
 
#### Splitting into Training and Testing dataset <a class="anchor"
id="split"></a> 
 
with attribute split:
- users
- apps
- user_history 
 
#### users 

**In [11]:**

{% highlight python %}
users_training = users.loc[users['Split'] == 'Train']
{% endhighlight %}

**In [12]:**

{% highlight python %}
users_testing = users.loc[users['Split'] == 'Test']
{% endhighlight %}
 
#### apps 

**In [13]:**

{% highlight python %}
apps_training = apps.loc[apps['Split'] == 'Train']
{% endhighlight %}

**In [14]:**

{% highlight python %}
apps_testing = apps.loc[apps['Split'] == 'Test']
{% endhighlight %}
 
#### user_history 

**In [15]:**

{% highlight python %}
user_history_training = user_history.loc[user_history['Split'] == 'Train']
{% endhighlight %}

**In [16]:**

{% highlight python %}
user_history_testing = user_history.loc[user_history['Split'] == 'Test']
{% endhighlight %}
 
##### Dataframes
- users_training
- users_testing
- apps_training
- apps_testing
- user_history_training
- user_history_testing 
 
### Preprocessing <a class="anchor" id="preprocessing"></a>
- Considering only US
- Removing data with empty state 

**In [17]:**

{% highlight python %}
jobs_US = jobs.loc[jobs['Country'] == 'US']
{% endhighlight %}

**In [18]:**

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



**In [19]:**

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



**In [20]:**

{% highlight python %}
statewise_jobs = jobs_US.groupby(['State']).size().reset_index(name = 'Statewise').sort_values('Statewise', ascending = False)
{% endhighlight %}

**In [21]:**

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



**In [22]:**

{% highlight python %}
citywise_jobs = jobs_US.groupby(['City']).size().reset_index(name='Citywise').sort_values('Citywise', ascending=False)
{% endhighlight %}

**In [23]:**

{% highlight python %}
citywise_jobs_top = citywise_jobs.loc[citywise_jobs['Citywise']>=12]
{% endhighlight %}
 
- jobs_US
- statewise_jobs
- citywise_jobs
- citywise_jobs_top 
 
### User profile based on location 

**In [24]:**

{% highlight python %}
users_training_US = users_training.loc[users_training['Country'] == 'US']
{% endhighlight %}

**In [25]:**

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



**In [26]:**

{% highlight python %}
users_training_statewise_top = users_training_statewise.loc[users_training_statewise['statewise'] >= 12]
{% endhighlight %}

**In [27]:**

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



**In [28]:**

{% highlight python %}
users_training_citywise_top = users_training_citywise.loc[users_training_citywise['citywise'] >= 12]
{% endhighlight %}
 
- users_training_US
- users_training_statewise
- users_training_citywise
- users_training_citywise_top 

**In [29]:**

{% highlight python %}
import ast 
from scipy import stats
from ast import literal_eval
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
from sklearn.metrics.pairwise import linear_kernel, cosine_similarity
# from nltk.stem.snowball import SnowballStemmer
# from nltk.stem.wordnet import WordNetLemmatizer
# from nltk.corpus import wordnet
{% endhighlight %}
 
## Building model <a class="anchor" id="model"></a> 

**In [30]:**

{% highlight python %}
jobs_US.columns
{% endhighlight %}




    Index(['JobID', 'WindowID', 'Title', 'Description', 'Requirements', 'City',
           'State', 'Country', 'Zip5', 'StartDate', 'EndDate'],
          dtype='object')



**In [31]:**

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



**In [32]:**

{% highlight python %}
jobs_US_base_line = jobs_US.iloc[0:10000,0:8]
{% endhighlight %}

**In [33]:**

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



**In [34]:**

{% highlight python %}
jobs_US_base_line['Title'] = jobs_US_base_line['Title'].fillna('')
jobs_US_base_line['Description'] = jobs_US_base_line['Description'].fillna('')
#jobs_US_base_line['Requirements'] = jobs_US_base_line['Requirements'].fillna('')

jobs_US_base_line['Description'] = jobs_US_base_line['Title'] + jobs_US_base_line['Description']
{% endhighlight %}
 
## Clean html <a class="anchor" id="html"></a> 

**In [35]:**

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

**In [36]:**

{% highlight python %}
jobs_US_base_line['Description'] = jobs_US_base_line['Description'].astype(dtype='str').apply(preprocessor)
{% endhighlight %}

**In [37]:**

{% highlight python %}
jobs_US_base_line.loc[0,'Description']
{% endhighlight %}




    'security engineer technical leadsecurity clearance required top secret job number tmr 447location of job washington dctmr inc is an equal employment opportunity companyfor more job opportunities with tmr visit our website www tmrhq comsend resumes to hr tmrhq2 com job summary leads the customer rsquo s overall cyber security strategy formalizes service offerings consisted with itil best practices and provides design and architecture support provide security design architecture support for ojp rsquo s it security division itsd leads the secops team in the day to day ojp security operations support provides direction when needed in a security incident or technical issues works in concert with network operations on design integration for best security posture supports business development functions including capture management proposal development and responses and other initiatives to include conferences trade shows webinars developing white papers and the like identifies resources and mentors in house talent to ensure tmr remains responsive to growing initiatives and contracts with qualified personnel '


 
## Dataset 
 
**From here onwards use `jobs_US_base_line` data frame to work on, which is
selected by `jobs_US.iloc[0:10000,0:8]`.** 

**In [38]:**

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


 
#### Dataframes
- users_training
- users_testing
- apps_training
- apps_testing
- user_history_training
- user_history_testing

##### Location
- jobs_US
- statewise_jobs
- citywise_jobs
- citywise_jobs_top 
 
# Content based filtering <a class="anchor" id="content"></a> 
 
## job description based recommender <a class="anchor" id="jobdesc"></a>
using term frequency-inverse document frequency 

**In [39]:**

{% highlight python %}
tf = TfidfVectorizer(analyzer='word',ngram_range=(1, 2),min_df=0, stop_words='english')
tfidf_matrix = tf.fit_transform(jobs_US_base_line['Description'])
{% endhighlight %}

**In [40]:**

{% highlight python %}
tfidf_matrix.shape
{% endhighlight %}




    (10000, 535561)



**In [41]:**

{% highlight python %}
print(tfidf_matrix)
{% endhighlight %}

      (0, 441695)	0.22505879122815065
      (0, 169589)	0.030669382747046916
      (0, 488697)	0.04752972531305034
      (0, 264028)	0.07576675940802603
      (0, 91384)	0.043127331709931944
      (0, 415767)	0.020558424763840996
      (0, 441253)	0.04981586018912077
      (0, 253884)	0.08610417293518174
      (0, 326351)	0.03628758749940522
      (0, 498708)	0.23741486018184604
      (0, 12862)	0.07913828672728201
      (0, 522879)	0.04488762967640074
      (0, 133507)	0.07913828672728201
      (0, 176599)	0.028902410192437344
      (0, 167677)	0.03274166426535237
      (0, 337345)	0.021001362288507987
      (0, 104352)	0.07000309538665801
      (0, 336738)	0.023882963162787138
      (0, 520158)	0.03215125588973581
      (0, 524056)	0.036239661087183676
      (0, 532905)	0.026723078362849224
      (0, 498712)	0.07913828672728201
      (0, 108785)	0.07913828672728201
      (0, 425609)	0.0371210673841218
      (0, 226031)	0.03469578910963611
      :	:
      (9999, 344616)	0.05017478681815903
      (9999, 467781)	0.05017478681815903
      (9999, 74829)	0.05017478681815903
      (9999, 523365)	0.05017478681815903
      (9999, 260759)	0.05017478681815903
      (9999, 74696)	0.05017478681815903
      (9999, 96875)	0.05017478681815903
      (9999, 150349)	0.05017478681815903
      (9999, 373161)	0.05017478681815903
      (9999, 390419)	0.05017478681815903
      (9999, 129078)	0.05017478681815903
      (9999, 203951)	0.05017478681815903
      (9999, 317361)	0.05017478681815903
      (9999, 385716)	0.05017478681815903
      (9999, 79158)	0.05017478681815903
      (9999, 492689)	0.05017478681815903
      (9999, 220490)	0.05017478681815903
      (9999, 414561)	0.05017478681815903
      (9999, 184487)	0.05017478681815903
      (9999, 492732)	0.05017478681815903
      (9999, 94472)	0.05017478681815903
      (9999, 351340)	0.05017478681815903
      (9999, 97193)	0.05017478681815903
      (9999, 447009)	0.05017478681815903
      (9999, 351336)	0.05339311347511375


**In [42]:**

{% highlight python %}
jobs_US_base_line.loc[0,'Description']
{% endhighlight %}




    'security engineer technical leadsecurity clearance required top secret job number tmr 447location of job washington dctmr inc is an equal employment opportunity companyfor more job opportunities with tmr visit our website www tmrhq comsend resumes to hr tmrhq2 com job summary leads the customer rsquo s overall cyber security strategy formalizes service offerings consisted with itil best practices and provides design and architecture support provide security design architecture support for ojp rsquo s it security division itsd leads the secops team in the day to day ojp security operations support provides direction when needed in a security incident or technical issues works in concert with network operations on design integration for best security posture supports business development functions including capture management proposal development and responses and other initiatives to include conferences trade shows webinars developing white papers and the like identifies resources and mentors in house talent to ensure tmr remains responsive to growing initiatives and contracts with qualified personnel '



**In [43]:**

{% highlight python %}
cosine_sim = linear_kernel(tfidf_matrix, tfidf_matrix)
{% endhighlight %}

**In [44]:**

{% highlight python %}
cosine_sim[0]
{% endhighlight %}




    array([1.        , 0.03241652, 0.00838853, ..., 0.01491531, 0.01491531,
           0.01491531])



**In [45]:**

{% highlight python %}
jobs_US_base_line = jobs_US_base_line.reset_index()
titles = jobs_US_base_line['Title']
indices = pd.Series(jobs_US_base_line.index, index=jobs_US_base_line['Title'])
{% endhighlight %}

**In [46]:**

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
      <th>index</th>
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
      <td>0</td>
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
      <td>1</td>
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
      <td>2</td>
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
      <td>3</td>
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
      <td>4</td>
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



**In [47]:**

{% highlight python %}
print(indices)
{% endhighlight %}

    Title
    Security Engineer/Technical Lead                                 0
    SAP Business Analyst / WM                                        1
    P/T HUMAN RESOURCES ASSISTANT                                    2
    Route Delivery Drivers                                           3
    Housekeeping                                                     4
    SALON/SPA COORDINATOR                                            5
    SUPERINTENDENT                                                   6
    ELECTRONIC PRE-PRESS PROFESSIONAL                                7
    UTILITY LINE TRUCK OPERATOR/ DIGGER DERRICK                      8
    CONSTRUCTION PROJECT MGR & PM TRAINEE                            9
    Administrative Assistant                                        10
    ACCOUNT EXECUTIVES                                              11
    COMMERCIAL ESTIMATOR                                            12
    Immediate Opening                                               13
    TESL Adjunct                                                    14
    Salon Manager/Hairstylists                                      15
    VOCATIONAL COUNSELOR                                            16
    GALLERY SALES POSITIONS                                         17
    SURGICAL SCRUB TECH                                             18
    Real Estate Agent                                               19
    LPN, RN, CNA, TECHS                                             20
    Vacation Sales Representatives                                  21
    Top Sales Agent                                                 22
    Quick Service Food & Beverage                                   23
    CREDIT/COLLECTIONS ASSISTANT                                    24
    POOL TECH                                                       25
    EXPERIENCED ROOFERS                                             26
    ARIZA TALENT & MODELING                                         27
    CDL CLASS A DRIVER                                              28
    Skilled Tradesman                                               29
                                                                  ... 
    Sales Representative / Account Manager /  Customer Service    9970
    Sales Representative / Account Manager /  Customer Service    9971
    Sales Representative / Account Manager /  Customer Service    9972
    Sales Representative / Account Manager /  Customer Service    9973
    Sales Representative / Account Manager /  Customer Service    9974
    Sales Representative / Account Manager /  Customer Service    9975
    Sales Representative / Account Manager /  Customer Service    9976
    Sales Representative / Account Manager /  Customer Service    9977
    Sales Representative / Account Manager /  Customer Service    9978
    Sales Representative / Account Manager /  Customer Service    9979
    Sales Representative / Account Manager /  Customer Service    9980
    Sales Representative / Account Manager /  Customer Service    9981
    Sales Representative / Account Manager /  Customer Service    9982
    Sales Representative / Account Manager /  Customer Service    9983
    Sales Representative / Account Manager /  Customer Service    9984
    Sales Representative / Account Manager /  Customer Service    9985
    Sales Representative / Account Manager /  Customer Service    9986
    Sales Representative / Account Manager /  Customer Service    9987
    Sales Representative / Account Manager /  Customer Service    9988
    Sales Representative / Account Manager /  Customer Service    9989
    Sales Representative / Account Manager /  Customer Service    9990
    Sales Representative / Account Manager /  Customer Service    9991
    Sales Representative / Account Manager /  Customer Service    9992
    Sales Representative / Account Manager /  Customer Service    9993
    Sales Representative / Account Manager /  Customer Service    9994
    Sales Representative / Account Manager /  Customer Service    9995
    Sales Representative / Account Manager /  Customer Service    9996
    Sales Representative / Account Manager /  Customer Service    9997
    Sales Representative / Account Manager /  Customer Service    9998
    Sales Representative / Account Manager /  Customer Service    9999
    Length: 10000, dtype: int64


**In [48]:**

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

**In [49]:**

{% highlight python %}
get_recommendations('SAP Business Analyst / WM').head(10)
{% endhighlight %}




    1                           SAP Business Analyst / WM
    6051                    SAP FI/CO Business Consultant
    5159                          SAP Basis Administrator
    5868                       SAP FI/CO Business Analyst
    5351    SAP Sales and Distribution Solution Architect
    4796       Senior Specialist - SAP Configuration - SD
    5117                       SAP Integration Specialist
    4290           SAP FICO Functional -2years experience
    4728           SAP ABAP Developer with PRA experience
    5244                                 Business Analyst
    Name: Title, dtype: object



**In [50]:**

{% highlight python %}
get_recommendations('Security Engineer/Technical Lead').head(10)
{% endhighlight %}




    0                        Security Engineer/Technical Lead
    5906                             Senior Security Engineer
    6380                Security Technology - SIEM Consultant
    3248                Senior Lead Systems Security Engineer
    1302                       Information Security Architect
    5525                   Sr. Information Security Architect
    6873              Integrated System Service Engineer - CA
    3230                    Computer Systems Security Manager
    1568                 Senior Information Security Engineer
    4901    Cloud Services Security Application Administrator
    Name: Title, dtype: object


 
## similar user based recommender <a class="anchor" id="similarusers"></a>

- degree type, majors and total years of experience
- `users_training` dataset 

**In [51]:**

{% highlight python %}
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



**In [52]:**

{% highlight python %}
user_based_approach_US = users_training.loc[users_training['Country']=='US']
{% endhighlight %}

**In [53]:**

{% highlight python %}
user_based_approach = user_based_approach_US.iloc[0:10000,:].copy()
{% endhighlight %}

**In [54]:**

{% highlight python %}
user_based_approach.head()
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



**In [55]:**

{% highlight python %}
user_based_approach['DegreeType'] = user_based_approach['DegreeType'].fillna('')
user_based_approach['Major'] = user_based_approach['Major'].fillna('')
user_based_approach['TotalYearsExperience'] = str(user_based_approach['TotalYearsExperience'].fillna(''))

user_based_approach['DegreeType'] = user_based_approach['DegreeType'] + user_based_approach['Major'] + \
                                    user_based_approach['TotalYearsExperience']

{% endhighlight %}

**In [56]:**

{% highlight python %}
tf = TfidfVectorizer(analyzer='word',ngram_range=(1, 2),min_df=0, stop_words='english')
tfidf_matrix = tf.fit_transform(user_based_approach['DegreeType'])
{% endhighlight %}

**In [57]:**

{% highlight python %}
tfidf_matrix.shape
{% endhighlight %}




    (10000, 7337)



**In [58]:**

{% highlight python %}
cosine_sim = linear_kernel(tfidf_matrix,tfidf_matrix)
{% endhighlight %}

**In [59]:**

{% highlight python %}
cosine_sim[0]
{% endhighlight %}




    array([1.        , 0.67053882, 0.84759861, ..., 0.43990417, 0.79335895,
           0.69670809])



**In [60]:**

{% highlight python %}
user_based_approach = user_based_approach.reset_index()
userid = user_based_approach['UserID']
indices = pd.Series(user_based_approach.index, index=user_based_approach['UserID'])
indices.head(2)
{% endhighlight %}




    UserID
    47    0
    72    1
    dtype: int64



**In [61]:**

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

**In [62]:**

{% highlight python %}
get_recommendations_userwise(123)
{% endhighlight %}




    [4, 150, 1594, 5560, 2464, 2846, 7945, 8125, 1171, 11, 24]



**In [63]:**

{% highlight python %}
def get_job_id(usrid_list):
    jobs_userwise = apps_training['UserID'].isin(usrid_list) #
    df1 = pd.DataFrame(data = apps_training[jobs_userwise], columns=['JobID'])
    joblist = df1['JobID'].tolist()
    Job_list = jobs['JobID'].isin(joblist) #[1083186, 516837, 507614, 754917, 686406, 1058896, 335132])
    df_temp = pd.DataFrame(data = jobs[Job_list], columns=['JobID','Title','Description','City','State'])
    return df_temp
{% endhighlight %}

**In [64]:**

{% highlight python %}
get_job_id(get_recommendations_userwise(47))
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
      <th>Description</th>
      <th>City</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>905894</th>
      <td>428902</td>
      <td>Aircraft Servicer</td>
      <td>&lt;b&gt;Job Classification: &lt;/b&gt; Direct Hire \r\n\r...</td>
      <td>Memphis</td>
      <td>TN</td>
    </tr>
    <tr>
      <th>975525</th>
      <td>1098447</td>
      <td>Automotive Service Advisor</td>
      <td>&lt;div&gt;\r&lt;div&gt;Briggs Nissan in Lawrence Kansas h...</td>
      <td>Lawrence</td>
      <td>KS</td>
    </tr>
    <tr>
      <th>980507</th>
      <td>37309</td>
      <td>Medical Lab Technician - High Volume Lab</td>
      <td>&lt;span&gt;Position Title:&lt;span&gt;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;...</td>
      <td>Fort Myers</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>986244</th>
      <td>83507</td>
      <td>Nurse Tech (CNA/STNA)</td>
      <td>&lt;p align="center"&gt;&lt;b&gt;Purpose of Your Job Posit...</td>
      <td>Englewood</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>987452</th>
      <td>93883</td>
      <td>Nurse Tech II (CNA/STNA)</td>
      <td>&lt;B&gt;Nurse Tech II (CNA/STNA)&lt;/B&gt; &lt;BR&gt;\r&lt;BR&gt;\rTh...</td>
      <td>Fort Myers</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1000910</th>
      <td>228284</td>
      <td>REGISTERED NURSE – ICU</td>
      <td>&lt;p&gt;&lt;strong&gt;&lt;span&gt;&lt;font face=""&gt;Registered Nurs...</td>
      <td>Punta Gorda</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1007140</th>
      <td>284840</td>
      <td>Certified Nursing Assistant / CNA</td>
      <td>&lt;hr&gt;\r&lt;p style="text-align: center"&gt;&lt;strong&gt;Ce...</td>
      <td>Saint Petersburg</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1007141</th>
      <td>284841</td>
      <td>Home Health Aide / HHA</td>
      <td>&lt;hr&gt;\r&lt;p style="text-align: center"&gt;&lt;strong&gt;Ho...</td>
      <td>Saint Petersburg</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1009455</th>
      <td>312536</td>
      <td>Secretary II</td>
      <td>&lt;br&gt;&lt;br&gt;&lt;b&gt;Department: &lt;/b&gt;COMM Maryland Cardi...</td>
      <td>Baltimore</td>
      <td>MD</td>
    </tr>
    <tr>
      <th>1011978</th>
      <td>341662</td>
      <td>Medical Assistant</td>
      <td>Certified Medical Assistant for busy Pain Clin...</td>
      <td>Fort Myers</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1034578</th>
      <td>551375</td>
      <td>Phlebotomist</td>
      <td>&lt;p&gt;Every day All Medical Personnel helps excep...</td>
      <td>Clearwater</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1048060</th>
      <td>684278</td>
      <td>Sales Representative / Customer Service / Acco...</td>
      <td>&lt;P&gt;Central Payment offers limitless opportunit...</td>
      <td>Bonita Springs</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1066952</th>
      <td>867194</td>
      <td>Hospital Liaison and Pharmaceutical</td>
      <td>Hospital Liaison with Pharmaceutical exp&lt;br /&gt;...</td>
      <td>Fort Myers</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1070785</th>
      <td>910932</td>
      <td>Nursing:  CNA (PRN)</td>
      <td>&lt;p&gt;&amp;nbsp;&lt;/p&gt;\r&lt;p&gt;Take advantage of this great...</td>
      <td>Fort Myers</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1076051</th>
      <td>960285</td>
      <td>All college grads apply! Entry level sales and...</td>
      <td>&lt;div&gt; &lt;span&gt;\r&lt;div&gt;\r&lt;div&gt;&lt;strong&gt;All college ...</td>
      <td>Fort Myers</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>1091311</th>
      <td>1108709</td>
      <td>Certified Nursing Assistant / CNA / HHA</td>
      <td>&lt;hr&gt;\r&lt;p style="text-align: center"&gt;&lt;strong&gt;Ce...</td>
      <td>Sarasota</td>
      <td>FL</td>
    </tr>
  </tbody>
</table>
</div>