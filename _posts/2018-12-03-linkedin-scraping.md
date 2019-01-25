---
title: Scraping Data from LinkedIn
desc: Gather data from LinkedIn using Selenium.
published: true
date: 2018-12-03
categories: project
tags: intern linkedin web-scraping selenium
---

Normal methods are not workable on LinkedIn, which manages to block many forms of scrape bot. I used [Selenium](https://selenium-python.readthedocs.io/locating-elements.html), a framework used for software testing by emulating a user using a browser.


Codes below are implemented in a jupyter notebook.
```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

from random import randint
from bs4 import BeautifulSoup
import pandas as pd
import time
from openpyxl import load_workbook
```
Driver will run a browser. We can set it to Chrome or Firefox. Below is using Chrome.
```python
driver = webdriver.Chrome()
driver.get('http://linkedin.com')
```
Selenium will help to log in to LinkedIn using your details.
```python
## Logging in to LinkedIn
emailElem = driver.find_element_by_id('login-email')
emailElem.send_keys('user@email.com')

passwordElem = driver.find_element_by_id('login-password')
passwordElem.send_keys('somepass')
passwordElem.submit()
```
Load list of name from excel to search. A simple data cleaning to remove "bin" variants in names.
```python
profiles = pd.read_excel('file.xlsx', sheet_name='sheet', usecols=[2])
profiles['Name'] = profiles['Name'].dropna().apply(lambda x: x.lower().replace('/','').replace(
    ' binti ',' bin ').replace(' ap ',' bin ').replace(' al ',' bin ').replace(' bt ',' bin '))
```
Creating 'First Name' and 'Last Name' columns.
```python
new = profiles['Name'].str.split(' bin ', n = 1, expand = True)
profiles['First Name']= new[0]
profiles['Last Name']= new[1]
profiles['Full Name'] = profiles['First Name'] + " " + profiles['Last Name']
profiles['Full Name'].fillna(value=profiles['Name'], inplace=True)

profiles.reset_index(inplace=True)
```

Various functions that we will be using:
```python
# fetch page HTML source
def fetch_html(driver):
    return driver.page_source

def parse_html(driver, html):
    return BeautifulSoup(html,'lxml')

# extract info from HTML
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

# search
def search_num(driver):
    lst_Count= driver.find_elements_by_xpath("//div[@class='search-result__wrapper']")
    return len(lst_Count)

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

# open new tab
def new_tab(name):
    print('searching ' + name + "'s profile")
    url_link = "https://www.linkedin.com/search/results/all/?keywords=" + name + "&origin=GLOBAL_SEARCH_HEADER"
    driver.execute_script("window.open('about:blank', 'tab2');")
    
# open search in new tab
def search_new_tab(name):
    time.sleep(randint(6,20))
    print('searching ' + name + "'s profile")
    url_link = "https://www.linkedin.com/search/results/all/?keywords=" + name + "&origin=GLOBAL_SEARCH_HEADER"
    
    driver.execute_script("window.open('about:blank');")
    window_list = driver.window_handles
    driver.switch_to.window(window_list[-1])
    driver.get(url_link)
    
#     <div class="search-result__wrapper">

    lst_Count= driver.find_elements_by_xpath("//div[@class='search-result__wrapper']")
    if len(lst_Count) == 1:
        time.sleep(randint(0,5))
        search = driver.find_element_by_css_selector('h3.actor-name-with-distance')
        search.click()
    else: scroll()
        
        
# scroll the page
def scroll():
    num = randint(0,2)
    if num == 1:
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(randint(3,5))
        num2 = randint(0,1)
        if num2 == 0:
            driver.execute_script("window.scrollTo(0, 0);")

def close_tab():
    current_window = driver.current_window_handle
    new_window = [window for window in driver.window_handles if window != current_window]
    for i in range(len(new_window)):
        new_window = [window for window in driver.window_handles if window != current_window][0]
        driver.switch_to.window(new_window)
        driver.close()
    driver.switch_to.window(current_window)
    driver.get("https://www.linkedin.com/feed/")
    
def write_to_excel():
    book = load_workbook(path)
    writer = pd.ExcelWriter(path, engine = 'openpyxl')
    writer.book = book

    df.to_excel(writer, sheet_name=str(x))
    writer.save()
    writer.close()
```
Running the functions. Many excel files has many names, so I decided to run a batch of 20 each time.
```python
path = r"output.xlsx"
x = 0

start_time = time.time()
y = 20

i = x*y 
j = i + y

# check if my searches is more than the name list
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
```
Rotate the tab to scrape data individually. Write to excel.
```python
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
```

Get the names of excel sheets.
```python
xls = pd.ExcelFile(path, on_demand = True)
sheets = xls.sheet_names
print(sheets)
```
Close all tabs.
```python
close_tab()
x += 1
print(x)
```
