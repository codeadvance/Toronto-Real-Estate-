Contents
This project involves web scraping real estate data from Realtor.ca using Python. The main steps include:

Setting up the environment and installing necessary packages.
Using Selenium WebDriver to navigate and search Realtor.ca.
Scraping data using Beautiful Soup and Selenium.
Cleaning and manipulating the data using Pandas.
Visualizing the data using Matplotlib, Seaborn, and Plotly.
Getting Started: Installation
First, install the necessary Python packages, primarily Selenium.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
##Real Estate Market Graphs:
## Section 1: Updated: July 2024
<img src="https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/31182e86-64fc-49a9-95ca-658474c3be1c" style= "max-width: 100%; height: 300px;">
<img src="https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/c8818443-707e-4f5f-8253-0c1bd3b4cf8f" style= "max-width: 100%; height: 300px;">
<img src="https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/71273ebc-6098-4046-9df0-87d6a867dbdc" style= "max-width: 100%; height:300px;">
<img src="https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/9e889f55-9474-4dd3-82d2-fb174f7822db" style= "max-width: 100%; height: 300px;">


## Section 2: Real Estate Market Graphs:Feb 2024
![image](https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/02b54440-37d4-42d5-8976-e81e72e4680a)
![image](https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/e687a0c9-a88a-4be4-b5e4-2e6297986300)
![image](https://github.com/codeadvance/Toronto-Real-Estate-/assets/132302205/e4c8a11e-764d-4d93-a0c1-8f39c664821b)

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Coding Preview: 
sh
Copy code
!pip install selenium
Open Realtor.ca Through Webdriver
To interact with Realtor.ca, we'll use Selenium WebDriver.

python
Copy code
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
import time

# Open a webdriver to direct to the website
driver = webdriver.Chrome('E:/chromedriver-win64/chromedriver-win64/chromedriver.exe')

# Go to Realtor.ca
driver.get("https://www.realtor.ca")
Scraping Data Using Beautiful Soup/Selenium
Use Selenium to navigate and Beautiful Soup to parse the HTML content.

python
Copy code
from bs4 import BeautifulSoup

# Finding search bar on Realtor.ca
search_bar = driver.find_element(By.XPATH, '/html/body/form/div[5]/div[2]/span/div/div[1]/div/div[1]/div[1]/div[1]/div[2]/input[2]')
search_bar.send_keys("Toronto")
search_bar.send_keys(Keys.RETURN)

# Scrape the page source
time.sleep(3)
page_source = driver.page_source
soup = BeautifulSoup(page_source, 'html.parser')
Clean/Manipulate Data Using Pandas
Extract and clean the data using Pandas.

python
Copy code
import pandas as pd

price = []
address = []
bedroom = []
room = []
bathroom = []

# Loop through the pages
for page in range(1, 100):
    time.sleep(2)
    driver.execute_script("window.scrollTo(0,document.body.scrollHeight);")
    page_source = driver.page_source
    soup = BeautifulSoup(page_source, 'html.parser')

    for house in soup.find_all('div', class_='cardCon'):
        price.append(house.find('div', class_='smallListingCardPrice').get_text())
        try:
            address.append(house.find('div', class_='smallListingCardAddress').get_text())
        except:
            address.append("No Address")
        rooms = house.find_all('div', class_='smallListingCardIconNum')
        if len(rooms) == 2:
            bedroom.append(rooms[0].get_text().split('+')[0].strip())
            try:
                room.append(rooms[0].get_text().split('+')[1].strip())
            except:
                room.append(0)
            bathroom.append(rooms[1].get_text())
        elif len(rooms) == 1:
            bathroom.append(rooms[0].get_text())
            bedroom.append(0)
            room.append(0)
        else:
            bathroom.append(0)
            bedroom.append(0)
            room.append(0)

    driver.find_element(By.XPATH, '/html/body/form/div[5]/div[2]/span/div/div[3]/div/div[1]/div[2]/div[4]/span/div/a[3]/div').click()

houses = pd.DataFrame({
    'price': price,
    'address': address,
    'bedroom': bedroom,
    'den': room,
    'bathroom': bathroom
})

houses.to_csv('houses.csv', index=False)
Data Visualizations Using Matplotlib/Seaborn/Plotly
Visualize the cleaned data.

python
Copy code
import matplotlib.pyplot as plt
import seaborn as sns

# Load the data
houses = pd.read_csv('houses.csv')

# Extract city from address
houses['city'] = houses['address'].apply(lambda x: x.split(',')[-2].strip() if len(x.split(',')) >= 2 else 'Unknown')

# Remove non-numeric characters from price and convert to float
houses['price'] = houses['price'].replace('[\$,]', '', regex=True).astype(float)

# Group by city and calculate average price
average_prices = houses.groupby('city')['price'].mean()

# Plotting the pie chart
plt.figure(figsize=(8, 6))
average_prices.plot(kind='pie', autopct='%1.1f%%', startangle=140)
plt.title('% House Listing Prices by City')
plt.ylabel('')
plt.show()

# Bar chart of average prices by city
plt.figure(figsize=(10, 6))
average_prices.plot(kind='bar', color='skyblue')
plt.title('Average Prices by City')
plt.xlabel('City')
plt.ylabel('Average Price')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
