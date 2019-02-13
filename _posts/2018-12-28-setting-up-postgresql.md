---
title: Setting Up PostgreSQL
excerpt: Building a database from scratch.
published: true
date: 2018-12-28
categories: project
tags: intern postgres database notebook
---
To install PostgreSQL follow this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04).

### Installing
``` sh
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### Using Postgres
Switch to postgres (default) account on your server type:
``` sh
sudo -i -u postgres
```
Access PostgreSQL prompt:
``` sh
psql
```
Exit interactive Postgres session:
```
\q
```

### New database
Create a new database:
``` sh
sudo -i -u postgres
createdb database-name
```

### Changing user and database
Add user:
``` sh
sudo adduser user-name
```
Switch user and connect to the user database (after creating one with name same as user-name):
``` sh
sudo -i -u user-name
psql
```
Changing it altogether:
```
psql my_database -U postgres
```
Check current connection info:
```
\conninfo
```

### Viewing table and database
List database:
```
\l
```
List table:
```
\dt
\dt *.*
```
List table in a public schema:
```
\dt public.*
```
List schema:
```
\dn
```

### Dumping database for backup
```
pg_dump dbname > \path\outfile
```

## Extension used:
```
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION pg_trgm;
```

Refer [rosette](https://www.rosette.com/blog/overview-fuzzy-name-matching-techniques/) and [wikiversity](https://en.wikiversity.org/wiki/Duplicate_record_detection) for more info.

Create database as user postgres 

**In [1]:**

{% highlight python %}
%%bash
sudo -i -u postgres
dropdb test;
createdb test;
{% endhighlight %}

    sudo: no tty present and no askpass program specified


**In [2]:**

{% highlight python %}
import pandas as pd
from sqlalchemy import create_engine
from sqlalchemy import text

engine = create_engine('postgresql://postgres:postgres@localhost:5432/test')
{% endhighlight %}

    /home/hectoryee/anaconda3/lib/python3.6/site-packages/psycopg2/__init__.py:144: UserWarning: The psycopg2 wheel package will be renamed from release 2.8; in order to keep installing from binary please use "pip install psycopg2-binary" instead. For details see: <http://initd.org/psycopg/docs/install.html#binary-install-from-pypi>.
      """)

 
## Importing data

Read excel file into pandas. 

**In [3]:**

{% highlight python %}
excel_file = "example.xlsx"
xlsx = pd.ExcelFile(excel_file)
print(xlsx.sheet_names)
df = xlsx.parse('Sheet1')

# # from multiple sheets
# excel_file = "example.xlsx"
# xlsx = pd.ExcelFile(excel_file)
# print(xlsx.sheet_names)
# excel_sheets = []
# for sheet in xlsx.sheet_names:
#     excel_sheets.append(xlsx.parse(sheet))
# ds = pd.concat(excel_sheets, ignore_index=True, sort=False)
{% endhighlight %}

    ['Sheet1']

 
### Data Star Student 

**In [4]:**

{% highlight python %}
df.reset_index(inplace =True)
df.columns
{% endhighlight %}




    Index(['index', 'Name', 'Gender', 'Company', 'Role'], dtype='object')



**In [5]:**

{% highlight python %}
df.rename(columns={'Name':'name',
                   'Gender':'gender',
                   'Company':'company',
                   'Role':'role'
                  }, 
          inplace = True)
{% endhighlight %}

**In [6]:**

{% highlight python %}
df.head()
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
      <th>name</th>
      <th>gender</th>
      <th>company</th>
      <th>role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>DIRECTOR</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Arumugam</td>
      <td>Male</td>
      <td>Sinar Bumi Petroleum Sdn Bhd</td>
      <td>DATA ARCHITECT</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Pravin</td>
      <td>Male</td>
      <td>Citi Group</td>
      <td>BUSINESS FRAUD ANALYST</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Tan Pei Ling</td>
      <td>Female</td>
      <td>University of Malaya</td>
      <td>SENIOR LECTURER</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>ENGINEER</td>
    </tr>
  </tbody>
</table>
</div>



**In [7]:**

{% highlight python %}
df[df.duplicated('name') == True].index
{% endhighlight %}




    Int64Index([4], dtype='int64')



**In [8]:**

{% highlight python %}
dup = df[df.duplicated('name') == True]
dup
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
      <th>name</th>
      <th>gender</th>
      <th>company</th>
      <th>role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>ENGINEER</td>
    </tr>
  </tbody>
</table>
</div>


 
### Individual 

**In [9]:**

{% highlight python %}
cleaned_df = df.drop_duplicates(subset=['name'], keep='last')
cleaned_df
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
      <th>name</th>
      <th>gender</th>
      <th>company</th>
      <th>role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Arumugam</td>
      <td>Male</td>
      <td>Sinar Bumi Petroleum Sdn Bhd</td>
      <td>DATA ARCHITECT</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Pravin</td>
      <td>Male</td>
      <td>Citi Group</td>
      <td>BUSINESS FRAUD ANALYST</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Tan Pei Ling</td>
      <td>Female</td>
      <td>University of Malaya</td>
      <td>SENIOR LECTURER</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>ENGINEER</td>
    </tr>
  </tbody>
</table>
</div>


 
## Functions 

**In [10]:**

{% highlight python %}
# get the intersection list of attributes that is to be inserted and table columns
def table(attrib):
    return df[df.columns.intersection(attrib)]

# list columns of table
def list_column(table):
    statement = "select column_name from information_schema.columns \
    where table_schema ='test' and table_name = '" + table + "';"
    df = pd.read_sql(statement, engine)
    return df['column_name'].tolist()

# select all from table
def show(table_name):
    query = 'SELECT * FROM test.' + table_name
    data = pd.read_sql(query, engine)
    return data

# append dataframe to table
def insert(df, table_name):
    df.to_sql(name=table_name, con=engine,schema = 'test', if_exists = 'append', index=False)
    return

# run sql query
def sql_query(statement):
    data = pd.read_sql(statement, engine)
    return data

# wrap up for insert method
def insert_into(table_name, attrib):
    if isinstance(attrib, pd.DataFrame):
        to_be_inserted = attrib
    else:
        to_be_inserted = table(attrib)
    insert(to_be_inserted, table_name)
    result = show(table_name)
    return result.head()
{% endhighlight %}
 
## Create Database

Here I create a database from a sql script.

create_db.sql
``` sql
DROP SCHEMA if exists test cascade;

CREATE SCHEMA test
    AUTHORIZATION postgres;
	
drop table if exists test.individual cascade;
create table test.individual(
	name varchar (80) primary key, 
	gender varchar (7),
	company varchar (120),
	role varchar (120),
	ts timestamp default now()
	);
```

**In [11]:**

{% highlight python %}
%%bash
psql -d test
\i /home/hectoryee/project/my_blog/database/create_db.sql 
\q
{% endhighlight %}

    DROP SCHEMA
    CREATE SCHEMA
    DROP TABLE
    CREATE TABLE


    psql:/home/hectoryee/project/my_blog/database/create_db.sql:1: NOTICE:  schema "test" does not exist, skipping
    psql:/home/hectoryee/project/my_blog/database/create_db.sql:6: NOTICE:  table "individual" does not exist, skipping

 
## Insert into 

**In [12]:**

{% highlight python %}
# columns to be inserted
attrib = ['name', 'gender', 'company', 'role']
data = cleaned_df.loc[:,attrib]
insert_into('individual', data)
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
      <th>name</th>
      <th>gender</th>
      <th>company</th>
      <th>role</th>
      <th>ts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arumugam</td>
      <td>Male</td>
      <td>Sinar Bumi Petroleum Sdn Bhd</td>
      <td>DATA ARCHITECT</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pravin</td>
      <td>Male</td>
      <td>Citi Group</td>
      <td>BUSINESS FRAUD ANALYST</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tan Pei Ling</td>
      <td>Female</td>
      <td>University of Malaya</td>
      <td>SENIOR LECTURER</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>ENGINEER</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
  </tbody>
</table>
</div>


 
## Query 

**In [13]:**

{% highlight python %}
individual = 'test.individual'
{% endhighlight %}

**In [14]:**

{% highlight python %}
# -- courses/modules offered by CADS
sql_query('''
select name
from test.individual;
''')
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
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arumugam</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pravin</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tan Pei Ling</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Shaw Kok Hao</td>
    </tr>
  </tbody>
</table>
</div>



**In [15]:**

{% highlight python %}
# -- insert new individual data
sql = '''
INSERT INTO test.individual (name, gender, company, role)
VALUES (:name, :gender, :company, :role);
'''

engine.execute(text(sql), {'name' : 'John Smith', 
                          'gender' : 'trans',
                          'company': 'cads',
                          'role': 'intern'})
{% endhighlight %}




    <sqlalchemy.engine.result.ResultProxy at 0x7f2501733080>



**In [16]:**

{% highlight python %}
show('individual').tail()
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
      <th>name</th>
      <th>gender</th>
      <th>company</th>
      <th>role</th>
      <th>ts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arumugam</td>
      <td>Male</td>
      <td>Sinar Bumi Petroleum Sdn Bhd</td>
      <td>DATA ARCHITECT</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pravin</td>
      <td>Male</td>
      <td>Citi Group</td>
      <td>BUSINESS FRAUD ANALYST</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tan Pei Ling</td>
      <td>Female</td>
      <td>University of Malaya</td>
      <td>SENIOR LECTURER</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>ENGINEER</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>4</th>
      <td>John Smith</td>
      <td>trans</td>
      <td>cads</td>
      <td>intern</td>
      <td>2019-02-03 15:56:14.333465</td>
    </tr>
  </tbody>
</table>
</div>



**In [17]:**

{% highlight python %}
# -- how many data
sql_query('''
SELECT COUNT(*)
FROM test.individual
''')
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
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>



**In [18]:**

{% highlight python %}
# -- update info
sql = '''
UPDATE test.individual
SET role = :role
WHERE name = :name;
'''

engine.execute(text(sql), {'role' : 'permanent staff', 
                       'name' : 'John Smith'})
{% endhighlight %}




    <sqlalchemy.engine.result.ResultProxy at 0x7f2500997b70>



**In [19]:**

{% highlight python %}
show('individual').tail()
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
      <th>name</th>
      <th>gender</th>
      <th>company</th>
      <th>role</th>
      <th>ts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arumugam</td>
      <td>Male</td>
      <td>Sinar Bumi Petroleum Sdn Bhd</td>
      <td>DATA ARCHITECT</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pravin</td>
      <td>Male</td>
      <td>Citi Group</td>
      <td>BUSINESS FRAUD ANALYST</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tan Pei Ling</td>
      <td>Female</td>
      <td>University of Malaya</td>
      <td>SENIOR LECTURER</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Shaw Kok Hao</td>
      <td>Male</td>
      <td>Speedminer Sdn Bhd</td>
      <td>ENGINEER</td>
      <td>2019-02-03 15:56:14.126646</td>
    </tr>
    <tr>
      <th>4</th>
      <td>John Smith</td>
      <td>trans</td>
      <td>cads</td>
      <td>permanent staff</td>
      <td>2019-02-03 15:56:14.333465</td>
    </tr>
  </tbody>
</table>
</div>



**In [20]:**

{% highlight python %}
# -- delete info
sql = '''
DELETE FROM test.individual
WHERE name = :name;
'''

engine.execute(text(sql), {'name' : 'John Smith'})

{% endhighlight %}




    <sqlalchemy.engine.result.ResultProxy at 0x7f25009a1358>


 
## Fuzzy string matching

This is a mini project to work with. Check similar names to ensure there's no
duplicated record. 
 
```sql
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION pg_trgm;
```

References:
- [postgresql](https://www.postgresql.org/docs/current/fuzzystrmatch.html)
- [postgresonline](http://www.postgresonline.com/journal/archives/158-Where-is-
soundex-and-other-warm-and-fuzzy-string-things.html) 
 
#### Soundex

The Soundex system is a method of matching similar-sounding names by converting
them to the same code.

The fuzzystrmatch module provides two functions for working with Soundex codes:
```sql
soundex(text) returns text
difference(text, text) returns int
```

The `soundex` function converts a string to its Soundex code. The `difference`
function converts two strings to their Soundex codes and then reports the number
of matching code positions. Since Soundex codes have four characters, the result
ranges from zero to four, with zero being no match and four being an exact
match. (Thus, the function is misnamed — `similarity` would have been a better
name.) 

**In [21]:**

{% highlight python %}
%%bash
psql -d test
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION pg_trgm;
\q
{% endhighlight %}

    CREATE EXTENSION
    CREATE EXTENSION


**In [22]:**

{% highlight python %}
fuzz = "soundex(name)"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE {fuzz} = soundex('Pravin') 
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>soundex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pravin</td>
      <td>P615</td>
    </tr>
  </tbody>
</table>
</div>



**In [23]:**

{% highlight python %}
fuzz = "difference(name, 'Pravi')"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE {fuzz} > 2
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>difference</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pravin</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>


 
Levenshtein

This function calculates the Levenshtein distance between two strings:
```sql
levenshtein(text source, text target, int ins_cost, int del_cost, int sub_cost)
returns int
levenshtein(text source, text target) returns int
levenshtein_less_equal(text source, text target, int ins_cost, int del_cost, int
sub_cost, int max_d) returns int
levenshtein_less_equal(text source, text target, int max_d) returns int
```

Both `source` and `target` can be any non-null string, with a maximum of 255
characters. The cost parameters specify how much to charge for a character
insertion, deletion, or substitution, respectively. You can omit the cost
parameters, as in the second version of the function; in that case they all
default to 1.

`levenshtein_less_equal` is an accelerated version of the Levenshtein function
for use when only small distances are of interest. If the actual distance is
less than or equal to `max_d`, then `levenshtein_less_equal` returns the correct
distance; otherwise it returns some value greater than `max_d`. If `max_d` is
negative then the behavior is the same as levenshtein. 

**In [24]:**

{% highlight python %}
fuzz = "levenshtein(name, 'Tan Pei Li')"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE {fuzz} <= 6
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>levenshtein</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tan Pei Ling</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



**In [25]:**

{% highlight python %}
fuzz = "levenshtein(name, 'Tan Pei Li',1,0,4)"

sql_query(f'''
SELECT name , {fuzz}
FROM {individual}
WHERE soundex(name) = soundex('Tan Pei Li')
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>levenshtein</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tan Pei Ling</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>


 
#### Metaphone



Metaphone, like Soundex, is based on the idea of constructing a representative
code for an input string. Two strings are then deemed similar if they have the
same codes.

This function calculates the metaphone code of an input string:
```sql
metaphone(text source, int max_output_length) returns text
```

`source` has to be a non-null string with a maximum of 255 characters.
`max_output_length` sets the maximum length of the output metaphone code; if
longer, the output is truncated to this length. 

**In [26]:**

{% highlight python %}
fuzz = "metaphone('Pravi', 4)"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE metaphone(name, 4) = {fuzz}
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>metaphone</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



**In [27]:**

{% highlight python %}
fuzz = "metaphone('Pravi', 3)"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE metaphone(name, 3) = {fuzz}
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>metaphone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pravin</td>
      <td>PRF</td>
    </tr>
  </tbody>
</table>
</div>


 
#### Double Metaphone

The Double Metaphone system computes two “sounds like” strings for a given input
string — a “primary” and an “alternate”. In most cases they are the same, but
for non-English names especially they can be a bit different, depending on
pronunciation. These functions compute the primary and alternate codes:
```sql
dmetaphone(text source) returns text
dmetaphone_alt(text source) returns text
```

There is no length limit on the input strings. 

**In [28]:**

{% highlight python %}
fuzz = "dmetaphone('Pravi')"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE dmetaphone(name) = {fuzz}
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>dmetaphone</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



**In [29]:**

{% highlight python %}
fuzz = "dmetaphone_alt('Pravi')"

sql_query(f'''
SELECT name, {fuzz}
FROM {individual}
WHERE dmetaphone_alt(name) = {fuzz}
ORDER BY {fuzz};
''')
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
      <th>name</th>
      <th>dmetaphone_alt</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>


 
### trigram

[postgesql](https://www.postgresql.org/docs/current/pgtrgm.html)

A trigram is a group of three consecutive characters taken from a string. We can
measure the similarity of two strings by counting the number of trigrams they
share. This simple idea turns out to be very effective for measuring the
similarity of words in many natural languages.
Note

> pg_trgm ignores non-word characters (non-alphanumerics) when extracting
trigrams from a string. Each word is considered to have two spaces prefixed and
one space suffixed when determining the set of trigrams contained in the string.
For example, the set of trigrams in the string “cat” is “ c”, “ ca”, “cat”, and
“at ”. The set of trigrams in the string “foo|bar” is “ f”, “ fo”, “foo”, “oo ”,
“ b”, “ ba”, “bar”, and “ar ”.
 
 
pg_trgm Functions

Function|Returns|Description
:---|:---|:---
similarity(text, text) | real | Returns a number that indicates how similar the
two arguments are. The range of the result is zero (indicating that the two
strings are completely dissimilar) to one (indicating that the two strings are
identical).
show_trgm(text) | text[] | Returns an array of all the trigrams in the given
string. (In practice this is seldom useful except for debugging.)
word_similarity(text, text) | real | Returns a number that indicates the
greatest similarity between the set of trigrams in the first string and any
continuous extent of an ordered set of trigrams in the second string. For
details, see the explanation below.
strict_word_similarity(text, text) | real | Same as word_similarity(text, text),
but forces extent boundaries to match word boundaries. Since we don't have
cross-word trigrams, this function actually returns greatest similarity between
first string and any continuous extent of words of the second string.
show_limit() | real | Returns the current similarity threshold used by the %
operator. This sets the minimum similarity between two words for them to be
considered similar enough to be misspellings of each other, for example
(deprecated).
set_limit(real) | real | Sets the current similarity threshold that is used by
the % operator. The threshold must be between 0 and 1 (default is 0.3). Returns
the same value passed in (deprecated). 

**In [30]:**

{% highlight python %}
sql_query("SELECT word_similarity('word', 'two words');")
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
      <th>word_similarity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.8</td>
    </tr>
  </tbody>
</table>
</div>


 
In the first string, the set of trigrams is {" w"," wo","ord","wor","rd "}. In
the second string, the ordered set of trigrams is {" t"," tw","two","wo "," w","
wo","wor","ord","rds","ds "}. The most similar extent of an ordered set of
trigrams in the second string is {" w"," wo","wor","ord"}, and the similarity is
0.8.

Thus, the strict_word_similarity(text, text) function is useful for finding the
similarity to whole words, while word_similarity(text, text) is more suitable
for finding the similarity for parts of words.

> postgres10 does not support strict_word_similarity 

**In [31]:**

{% highlight python %}
sql_query("SELECT show_trgm('word')")
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
      <th>show_trgm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[  w,  wo, ord, rd , wor]</td>
    </tr>
  </tbody>
</table>
</div>



**In [32]:**

{% highlight python %}
sql_query("SELECT similarity('word', 'two words');")
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
      <th>similarity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.363636</td>
    </tr>
  </tbody>
</table>
</div>


 
pg_trgm Operators

Operator | Returns | Description
--- | --- | ---
text % text | boolean | Returns true if its arguments have a similarity that is
greater than the current similarity threshold set by
pg_trgm.similarity_threshold.
text <% text | boolean | Returns true if the similarity between the trigram set
in the first argument and a continuous extent of an ordered trigram set in the
second argument is greater than the current word similarity threshold set by
pg_trgm.word_similarity_threshold parameter.
text %> text | boolean | Commutator of the <% operator.
text <<% text | boolean | Returns true if its second argument has a continuous
extent of an ordered trigram set that matches word boundaries, and its
similarity to the trigram set of the first argument is greater than the current
strict word similarity threshold set by the
pg_trgm.strict_word_similarity_threshold parameter.
text %>> text | boolean | Commutator of the <<% operator.
text <-> text | real | Returns the “distance” between the arguments, that is one
minus the similarity() value.
text <<-> text | real | Returns the “distance” between the arguments, that is
one minus the word_similarity() value.
text <->> text | real | Commutator of the <<-> operator.
text <<<-> text | real | Returns the “distance” between the arguments, that is
one minus the strict_word_similarity() value.
text <->>> text | real | Commutator of the <<<-> operator. 
 
### GUC Parameters

pg_trgm.similarity_threshold (real)

    Sets the current similarity threshold that is used by the % operator. The
threshold must be between 0 and 1 (default is 0.3).

pg_trgm.word_similarity_threshold (real)

    Sets the current word similarity threshold that is used by <% and %>
operators. The threshold must be between 0 and 1 (default is 0.6).

 
 
### Index Support 
 
    CREATE TABLE test_trgm (t text);
    CREATE INDEX trgm_idx ON test_trgm USING GIST (t gist_trgm_ops);

or

    CREATE INDEX trgm_idx ON test_trgm USING GIN (t gin_trgm_ops); 

**In [33]:**

{% highlight python %}
sql = f'''
CREATE INDEX trgm_idx ON {individual} USING GIN (name gin_trgm_ops);
'''
engine.execute(text(sql))
{% endhighlight %}




    <sqlalchemy.engine.result.ResultProxy at 0x7f25009972e8>



**In [34]:**

{% highlight python %}
search_term = 'Pravi'
sql_query(f'''
SELECT name, similarity(name, '{search_term}') AS sml
FROM {individual}
WHERE name %% '{search_term}'
ORDER BY sml DESC, name
LIMIT 5;
''')
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
      <th>name</th>
      <th>sml</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pravin</td>
      <td>0.625</td>
    </tr>
  </tbody>
</table>
</div>


