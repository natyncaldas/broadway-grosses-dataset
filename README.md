# Broadway Grosses Dataset 
## *Jupyter notebook preview*
> Constructs a **dataset** consisting of box office grosses from all ***closed*** Broadway shows up to this day. Utilizes data from [Broadway World](https://www.broadwayworld.com).
### `SETUP`
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import matplotlib.pyplot as plt
```
## 1. Scraping website data 💻
### 1.1. Requesting HTML data
Sends HTTP GET request to page containing data from all Broadway shows
```python
page_all = requests.get("https://www.broadwayworld.com/grossescumulative.cfm")
page_all
```
Sends HTTP GET request to page containing data from current Broadway shows
```python
page_curr = requests.get("https://www.broadwayworld.com/grosses.cfm")
page_curr
```
### 1.2 Identifying required HTML tags
Retrieves Table element of all Broadway shows
```python
soup_all = BeautifulSoup(page_all.content, 'html.parser')
table_all = soup_all.find('table')
table_all
```
Retrieves list of all Anchor elements containing names of current Broadway shows
```python
soup_curr = BeautifulSoup(page_curr.content, 'html.parser')
list_curr = soup_curr.find_all('a', class_='title')
list_curr
```
## 2. Creating Dataset 🔠
### 2.1. Getting current shows
Creates list containing current show names.
```python
curr_shows = [show.text.strip() for show in list_curr]
curr_shows
```
2.2. Defining Dataframe
- Defines dataframe headers
- Iterates through HTML table containg all shows (starting at the second row to exclude header)
- Gets raw text from each table column and converts numbers to Integer type
- Adds columns to each row (filtering only closed shows)
```python
df = pd.DataFrame(columns=['Show', 'Gross', 'Average ticket price', 'Seats sold', 'Previews', 'Regular shows', 'Total performances'])
for row in table_all.tbody.find_all('tr')[1:]:   
    columns = row.find_all('td')
    if(columns != []):
        show = columns[0].find('strong').text.strip().replace('\n', ' ')
        gross = int(columns[1].text.strip()[1:].replace(',', ''))
        avg_tix = int(columns[2].text.strip()[1:].replace(',', ''))
        seats_sold = int(columns[3].text.strip().replace(',', ''))
        previews = int(columns[4].text.strip().replace(',', ''))
        reg_shows = int(columns[5].text.strip().replace(',', ''))
        total = int(columns[6].text.strip().replace(',', ''))
        if(show not in curr_shows):
            df.loc[len(df.index)] = [show, gross, avg_tix, seats_sold, previews, reg_shows, total]

df.head(n=20)
```
### 2.3. Exporting dataframe as CSV
```python
df.to_csv('./broadway_grosses.csv')
```
## 3. Plotting data 📊
### 3.1. Defining scatter plot
```python
scatter_plot = df.plot.scatter(x='Gross',y='Total performances', s=80, c='Average ticket price', colormap='cubehelix', edgecolors='grey', sharex=False)
plt.show()
```
### 3.2. Exporting plot
```python
figure = scatter_plot.get_figure()
figure.savefig('scatter_plot.png')
```
