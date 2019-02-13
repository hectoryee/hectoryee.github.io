---
title: Scraping Data from LinkedIn
excerpt: Gather data from LinkedIn using Selenium.
published: true
date: 2018-12-03
categories: project
tags: intern linkedin web-scraping selenium
---

Normal methods are not workable on LinkedIn, which manages to block many forms of scrape bot. I used [Selenium](https://selenium-python.readthedocs.io/locating-elements.html), a framework used for software testing by emulating a user using a browser.

## Import dependencies 

**In [2]:**

{% highlight python %}
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

from random import randint
from bs4 import BeautifulSoup
import pandas as pd
import time
from openpyxl import load_workbook
{% endhighlight %}
 
### Profiles to scrape 
 
Get the list of names to search for. Names are processed to remove "bin", "a/l"
etc. 

**In [17]:**

{% highlight python %}
# load data
profiles = pd.read_excel('../example.xlsx', sheet_name='Sheet1', usecols=[0])
profiles['Name'] = profiles['Name'].dropna().apply(lambda x: x.lower().replace('/','').replace(
    ' binti ',' bin ').replace(' ap ',' bin ').replace(' al ',' bin ').replace(' bt ',' bin '))

# separate names
new = profiles['Name'].str.split(' bin ', n = 1, expand = True)
profiles['First Name']= new[0]
profiles['Last Name']= new[1]
profiles['Full Name'] = profiles['First Name'] + " " + profiles['Last Name']
profiles['Full Name'].fillna(value=profiles['Name'], inplace=True)

# profiles.reset_index(inplace=True)
profiles
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
      <th>Name</th>
      <th>First Name</th>
      <th>Last Name</th>
      <th>Full Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>shaw kok hao</td>
      <td>shaw kok hao</td>
      <td>None</td>
      <td>shaw kok hao</td>
    </tr>
    <tr>
      <th>1</th>
      <td>arumugam bin pillay</td>
      <td>arumugam</td>
      <td>pillay</td>
      <td>arumugam pillay</td>
    </tr>
    <tr>
      <th>2</th>
      <td>pravin</td>
      <td>pravin</td>
      <td>None</td>
      <td>pravin</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tan pei ling</td>
      <td>tan pei ling</td>
      <td>None</td>
      <td>tan pei ling</td>
    </tr>
    <tr>
      <th>4</th>
      <td>shaw kok hao</td>
      <td>shaw kok hao</td>
      <td>None</td>
      <td>shaw kok hao</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ashraf bin asli</td>
      <td>ashraf</td>
      <td>asli</td>
      <td>ashraf asli</td>
    </tr>
  </tbody>
</table>
</div>


 
## Methods 

**In [4]:**

{% highlight python %}
# fetch page html source
def fetch_html(driver):
    return driver.page_source

# parse html source to beautifulsoup
def parse_html(driver, html):
    return BeautifulSoup(html,'lxml')


# extract info from html using beautifulsoup
def extract_basic_info(soup, url):
    
    if soup.find('h1', {'class':'pv-top-card-section__name inline t-24 t-black t-normal'}):
        name = soup.find('h1', {'class':'pv-top-card-section__name inline t-24 t-black t-normal'}).getText().strip()
    else:
        name = None
        url = None
        
    if soup.find('h2',{'class':'pv-top-card-section__headline mt1 t-18 t-black t-normal'}):
        self_intro1 = soup.find('h2',{'class':'pv-top-card-section__headline mt1 t-18 t-black t-normal'}).getText().strip()
    else: self_intro1 = None
    
    if soup.find('h3', {'class':'pv-entity__school-name t-16 t-black t-bold'}):
        school = soup.find('h3', {'class':'pv-entity__school-name t-16 t-black t-bold'}).getText()
    else: school = None
        
    if soup.find('span', {'class':'pv-entity__comma-item'}):
        degree = soup.find('span', {'class':'pv-entity__comma-item'}).getText()
    else: degree = None

    if soup.find('span',{'class':'pv-entity__secondary-title'}):
        current_company = soup.find('span',{'class':'pv-entity__secondary-title'}).getText()
    else: current_company = None
        
    if soup.find('h3',{'class':'t-16 t-black t-bold'}):
        current_title = soup.find('h3',{'class':'t-16 t-black t-bold'}).getText()
    else: current_title = None
    
    if soup.find('p',{'class':'pv-entity__description t-14 t-black t-normal ember-view'}):
        desc = soup.find('p',{'class':'pv-entity__description t-14 t-black t-normal ember-view'}).getText()
    else: desc = None
        
    return {"url": [url],
            "name": [name],
            "school": [school],
            "degree": [degree],
            "company": [current_company],
            "title": [current_title],
            "desc": [desc]}

# get the number of results per page
def search_num(driver):
    lst_Count= driver.find_elements_by_xpath("//div[@class='search-result__wrapper']")
    return len(lst_Count)

# search for name in linkedin
def search(name, driver):
    searchElem = driver.find_element_by_xpath("//div[@id='nav-typeahead-wormhole']//input[1]")
    searchElem.clear()
    searchElem.send_keys(name)

    searchElem.send_keys(Keys.ENTER);
    lst_Count= driver.find_elements_by_xpath("//div[@class='search-result__wrapper']")
    
    if len(lst_Count) == 1:
        search = driver.find_element_by_css_selector('h3.actor-name-with-distance')
        search.click()
#     else:
#         print("Lot's of searches")
    return

# open new tab
def new_tab(name):
    print('searching ' + name + "'s profile")
    url_link = "https://www.linkedin.com/search/results/all/?keywords=" + name + "&origin=GLOBAL_SEARCH_HEADER"
    driver.execute_script("window.open('about:blank', 'tab2');")
    return
    
# open search in new tab
def search_new_tab(name):
    time.sleep(randint(6,20))
    print('searching ' + name + "'s profile")
    url_link = "https://www.linkedin.com/search/results/all/?keywords=" + name + "&origin=GLOBAL_SEARCH_HEADER"
    
    driver.execute_script("window.open('about:blank');")
    window_list = driver.window_handles
    driver.switch_to.window(window_list[-1])
    driver.get(url_link)

    lst_Count= driver.find_elements_by_xpath("//div[@class='search-result__wrapper']")
    if len(lst_Count) == 1:
        time.sleep(randint(0,5))
        search = driver.find_element_by_css_selector('h3.actor-name-with-distance')
        search.click()
    else: scroll()
    return
        
        
# scroll the page
def scroll():
    num = randint(0,2)
    if num == 1:
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(randint(3,5))
        num2 = randint(0,1)
        if num2 == 0:
            driver.execute_script("window.scrollTo(0, 0);")
    return

# close all tabs
def close_tab():
    current_window = driver.current_window_handle
    new_window = [window for window in driver.window_handles if window != current_window]
    for i in range(len(new_window)):
        new_window = [window for window in driver.window_handles if window != current_window][0]
        driver.switch_to.window(new_window)
        driver.close()
    driver.switch_to.window(current_window)
    driver.get("https://www.linkedin.com/feed/")
    return

# output collected data to excel
def write_to_excel():
    book = load_workbook(path)
    writer = pd.ExcelWriter(path, engine = 'openpyxl')
    writer.book = book

    df.to_excel(writer, sheet_name=str(x))
    writer.save()
    writer.close()
    return
{% endhighlight %}
 
## Main 
 
Logging in to LinkedIn automated by Selenium 

**In [21]:**

{% highlight python %}
driver = webdriver.Chrome()
driver.get('http://linkedin.com')

## Logging in to LinkedIn
emailElem = driver.find_element_by_id('login-email')
emailElem.send_keys('@mail.com')

passwordElem = driver.find_element_by_id('login-password')
passwordElem.send_keys('password')
passwordElem.submit()
{% endhighlight %}

**In [5]:**

{% highlight python %}
path = r"managers.xlsx"
x = 29
{% endhighlight %}
 
Since I have a big list to search, I decided to divide the name to batches of y.
x means the nth batch. 

**In [42]:**

{% highlight python %}
start_time = time.time()

y = 20

i = x*y 
j = i + y

if j > len(profiles.index):
    print('''
    ================
    more than enough
    ================
              ''')


search = profiles[i:j]
for index, row in search.iterrows():
    print(row['index'], end=' ')
    search_new_tab(row['Full Name'])
    
print("--- %s seconds ---" % (time.time() - start_time))
print("%i%% done" % (j/len(profiles.index) * 100))
{% endhighlight %}

    
        ================
        more than enough
        ================
                  
    680 searching jim daryl teo jin liang's profile
    681 searching mohd najib mohammad's profile
    682 searching nik zurin nik mohamed's profile
    683 searching mohamed bachik mohamed hussain's profile
    684 searching chua kiow kiow's profile
    685 searching nicky lim hui siang's profile
    686 searching woo choon kong's profile
    687 searching abd wahid idris's profile
    688 searching muniandy maruthiah's profile
    689 searching choi vincent's profile
    690 searching michael loh tet fook's profile
    691 searching see beng keh's profile
    --- 193.57338523864746 seconds ---
    101% done

 
## Collect data 
 
Collect HTML from all tabs and append to data frame. Write to excel new sheet. 

**In [None]:**

{% highlight python %}
window_list = driver.window_handles
df = pd.DataFrame()

for i in range(len(window_list)):
    driver.switch_to.window(window_list[i])
#     time.sleep(randint(0,5))
#     scroll()
    
    html = driver.page_source
    soup = BeautifulSoup(html,'lxml')
    url = driver.current_url

    dict_profile = extract_basic_info(soup, url)
    df2 = pd.DataFrame.from_dict(dict_profile)
    df = df.append(df2, ignore_index=True)
    
df.head()
write_to_excel()
{% endhighlight %}
 
Check new sheet is added. 

**In [40]:**

{% highlight python %}
xls = pd.ExcelFile(path, on_demand = True)
sheets = xls.sheet_names
print(sheets)
{% endhighlight %}

    ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29', '30', '31', '32', '33']

 
Close all tabs. 

**In [41]:**

{% highlight python %}
close_tab()
x += 1
print(x)
{% endhighlight %}

    34

