# Amazon-Listing-Scraping-and-Analysis_

![image](https://user-images.githubusercontent.com/61817305/166145550-103bde75-fa2b-4121-9276-605f7cf1573a.png)


## Tools Used
* Python, Excel and Power BI

## Libraries Used
* Pandas, BeautifulSoup, requests, urllib and random

### Task
* Create data from scratch.
* Scrape the Amazon Listing Data in Electronics category and create a DataFrame from it.

* Scraped Data includes - 

    - ASIN (Product unique identifier)
    - Category 
    - Whether the Product is Sponsored or not 
    - Does the Product have any Badge like [Amazon Choice, Best Seller etc..]
    - Title
    - Total number of Stars
    - Total Ratings
    - Product List Price
    - Product Original Price
    - Discount offered on the Prouct
    - Next Day Delivery for Prime Customers or not 

* The data is cleaned in Python and Excel, post which further Transformation is done in Power BI.
* In the last stage we have crated an automated dashboard in Power BI which shows 100% correct results, which can be verified on the Amazon website.
* The biggest realization we found in the analysis is that Brands only offer discount if their product is not doing well in comparision to their competition.



### Step 1

Import BeautifulSoup
Make GET request to fetch page data
Parse HTML
Filter relevalnt Parts

1. To scrape the data from Flipkart website we are going to use the Python package called beautifulsoup.

To scrape the data we use request librairy. One of the librairy is called 'urllib'
We wil be making a get request to fetch the data from the website 
The web page contains HTML, CSS JAVASCRIPT and all of that comes to us via get request.
So similarly If we want to get a webpage in python we are going to make a get request using this librairy.
    (We are on a machine client and we want to access some data which is on the internet, so we are going to make a get request to the server > we hit a particular url and in return we get a HTTP response.)
Our task would be to make a get request and get some response and parse that response to extract the exact the data.

```
from bs4 import BeautifulSoup as soup
import pandas as pd
import requests
import urllib # given a web url we want to open that via urllib
import time
import requests, random
```
```
def getdata (url):
    user_agents = [
      "chrome/5.0 (Windows NT 6.0; Win64; x64",
      "chrome/5.0 (Windows NT 6.0; Win64; x32",
      "chrome/5.0 (Windows NT 6.1; Win64; x64",
      "chrome/5.0 (Windows NT 6.1; Win64; x32",
      "safari/5.0 (Windows NT 6.1; Win64; x64",
      "safari/5.0 (Windows NT 6.1; Win64; x32",
      'Mozilla/5.0',
      'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:77.0) Gecko/20100101 Firefox/77.0',
      'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36',
      'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:77.0) Gecko/20100101 Firefox/77.0',
      'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36'
    ]
    user_agent = random.choice(user_agents) #Choose a random user_agent from the user_agents list
    header_ = {'User-Agent': user_agent}
```
- The HTTP headers User-Agent is a request header that allows a characteristic string that allows network protocol peers to identify the Operating System and Browser of the web-server
```
    header = { 'User-Agent' : 'chrome/5.0 (Windows NT 6.1; Win64; x64)' } 
    req = urllib.request.Request(url, headers=header_)
```
- The urllib.request module defines functions and classes which help in opening URLs (mostly HTTP), req will return a HTTP response which needs to be parsed.
```
    amazon_html = urllib.request.urlopen(req).read()  # we call the read method to read the HTTP data
```
- It basically contains the entire HTML of the Amazon page, there are different tags in it like div,h2,a,span,h3
- Now that we have fetched the page, the next part would be to parse the HTML, that's where Beautiful soup comes into picture  
- BeautifulSoup is a class and we import that above and now we will use the soup class to make a soup object which will help us to parse the HTML.
```
    a_soup = soup(amazon_html,'html.parser')
```
- soup() objects the html, we have to specify what kind of parsing we have to do: 
        - example: whether it's a json kind of data, whether it's HTML kind or something else.
        - In our case it's HTML kind of data so we give html parse as the second argument

###  Now our next task is to extract the relevant details form a_soup
```
    for e in a_soup.select('div[data-component-type="s-search-result"]'):
```
![image](https://user-images.githubusercontent.com/61817305/166100096-f9d03de0-c94d-424b-81bc-f241f13da74b.png)


```
        try:
            asin = e.find('a')['href'].replace('dp%2F', '/dp/').split('/dp/')[1].replace('%2','/ref').split('/ref')[0]
        except:
            asin = 'No ASIN Found'
```
![image](https://user-images.githubusercontent.com/61817305/166100202-e70de609-d952-400a-a70c-9e502ee89a4f.png)


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Example URL   
```
e.find('a')['href']  # RETURNS US THE BELOW URL
```
https://www.amazon.in/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_aps_sr_pg1_1?ie=UTF8&amp;adId=A006342821IBNABDN01DS&amp;url=%2FNew-Apple-iPhone-11-64GB%2Fdp%2FB08L89J9G3%2Fref%3Dsr_1_1_sspa%3Fcrid%3D3G56ZQQTTKYU7%26keywords%3Diphone%26qid%3D1651310661%26sprefix%3Diphon%252Caps%252C262%26sr%3D8-1-spons%26psc%3D1&amp;qualifier=1651310662&amp;id=1778621225268483&amp;widgetName=sp_atf

- Our aim is to get the unique identifier [ B08L89J9G3 ] within the url 
- So we  replace 'dp%2F' inside the link  with '/dp/'  
- Next we split the new link on '/dp/' and return the 2 index (Index starts from 0 in python)
- Once we do that we replace '%2'with '/ref' 
- Finally we split the new link on '/ref' and return the first index [0]  which gives us the ASIN B08L89J9G3

-------------------------------------------------------------------------------------------------------------------------------------------------

In case the code doesn't find any results it will return 'No ASIN Found'  that's what the below code will return.
```
        except:
            asin = 'No ASIN Found'
```
--------------------------------------------------------------------------------------------------------------------------------------------------
## Sponsored
```
        try:
            sponsored = e.find('span',{'class':'a-color-secondary'}).text
        except:
            sponsored = "Non Sponsored"
```
![image](https://user-images.githubusercontent.com/61817305/166101323-f22785a8-a8a4-47cc-a326-720ce680ac72.png)

---------------------------------------------------------------------------------------------------------------------------------------------------

## Badge
```
        try:
            tag = e.find('span',{'class':'a-badge-label-inner a-text-ellipsis'}).find('span', {'class': 'a-badge-text'}).text
        except:
            tag = "No Tag"

```
![image](https://user-images.githubusercontent.com/61817305/166101504-289195a5-13fe-45cb-9adc-c43ae98ecd78.png)

------------------------------------------------------------------------------------------------------------------------------------------------------------
## Title 
```
        try:
            title = e.find('h2').text
        except:
            title = None
```
![image](https://user-images.githubusercontent.com/61817305/166101607-e113e797-e4bf-43a1-a366-71b878461575.png)

-----------------------------------------------------------------------------------------------------------------------------------------------------------
## Stars

```
         try:
            stars = e.find('span',{'class':'a-icon-alt'}).text.split(' ')[0]
        except:
            stars = 0
```
![image](https://user-images.githubusercontent.com/61817305/166101674-6b1a88ca-7a11-40c1-b332-06e0f8a481f6.png)

 -----------------------------------------------------------------------------------------------------------------------------------------------------------

## Ratings
```            
        try:
            rating = e.find('span',{'class':'a-size-base s-underline-text'}).text
        except:
            rating = 0
```
![image](https://user-images.githubusercontent.com/61817305/166101705-b67cf87b-c74f-4309-8015-196a53c12a92.png)

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## List Price
```
        try:
            list_price = e.find('span',{'class':'a-price-whole'}).text
        except:
            list_price = 0
```
![image](https://user-images.githubusercontent.com/61817305/166101739-b1111902-b241-45c3-9832-7a0cb71d1c5d.png)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------
## Original Price

```                                       
        try:
            original_price = e.find('span',{'class':'a-price a-text-price'}).find('span', {'class': 'a-offscreen'}).text
        except:
            original_price = list_price   # IF original price is missing make that equal to the listing price
```
![image](https://user-images.githubusercontent.com/61817305/166101783-26bbbf83-a56c-4c2e-8b04-d9f66e93cbbb.png)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Discount
```
        try:
            discount = e.select_one('.a-letter-space +span').text.split('%')[0].replace('(','')
        except:
            discount = 0
            
```
![image](https://user-images.githubusercontent.com/61817305/166101836-fee616a2-aef4-47b8-b20e-d6c0ffe43b5b.png)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Next Day Delivery Or not
```
        try:
            date = e.find('span',{'class':'a-color-base a-text-bold'}).text
        except:
            date = 'No Prime Offer'
```
![image](https://user-images.githubusercontent.com/61817305/166101881-d24b33e1-e8a1-4407-a461-b61dbf347cef.png)


-----------------------------------------------------------------------------------------------------------------------------------------------------------------
```
                                       
cat = k  # k is the keyword that we are getting the results for 
data =[] # Empty list 
# Below for every result we are appending that to data
data.append({
            'ASIN': asin,  # We ar naming the column ASIN which stores the values of asin
            'Category': cat,    # we are also creating a new column which takes in value of k or cat
            'ASIN': asin,
            'Sponsored':sponsored,
            'Tag':tag,
            'Title':title,
            'Stars':stars,
            'Rating':rating,
            'List_price':list_price,
            'Original_price':original_price,
            'Discount':discount,
            'By_date': date
        })
        
    return a_soup
```
```                                       
def getnextpage(a_soup):
    
    try:
        page = a_soup.find('a',attrs={"class": 's-pagination-item s-pagination-next s-pagination-button s-pagination-separator'})['href']
        url =  'http://www.amazon.in'+ str(page)
    except:
        url = None
    return url

```
![image](https://user-images.githubusercontent.com/61817305/166101998-7d75adfc-9aae-4506-908c-9aadc36a4e05.png)

If we found no URL return Nothing and filally  return URL

```

--------------------------------------------------------------------------------------------------------------------------------------------------------
```
## Keywords
```
We pass the keywords as a list

```
keywords = ['bluetooth+earphone','earbuds','hard+drive','headphone','iphone','laptop',
                       'mixer+grinder','mobile','monitor','power+bank','printer','refrigerator','router',
                       'smartwatch','speaker', 'tablet','tv','vaccume+cleaner','washing+machine',
                       'apple+watch','macbook']
```

----------------------------------------------------------------------------------------------------------------------------------------------------------

```                                       
for k in keywords:
```
for every keyword in keywords 
```
    url = 'https://www.amazon.in/s?k='+k
```
URL will become 
- 'https://www.amazon.in/s?k='+ bluetooth+earphone
- 'https://www.amazon.in/s?k='+ earbuds

and so on .. 

```
    while True:
```

Till you find the last keyword run the below functions

```
        geturl = getdata(url)
```
- It will go to the top and run the function getdata(url)
- Once it it is finished running that function 
- The getdata(url) returns [ a_soup  ] 
- It will run the below function
- Which will accept the f_soup 
```
        url = getnextpage(geturl)
```
Go to the next page and return the URL of that page and give that to the getdata(url) function and this will continue till all results are found for a particular keyword and so on..
```       
        if not url:
            print('N 0   M O R E   R E S U L T S    F O U N D   F O R   T H I S   K E Y W O R D ')
            print('       S H O W I N G   R E S U L T S  F O R   N E X T   K E Y W O R D       ')
            break
        print(url)
        

Below we are creating the DataFrame from data
   
```
output = pd.DataFrame(data)

```
We are also adding an Extra column ['Data'] to the Dataframe with the date

```
output['Date'] = '24-04-2022' # Adding Date colum to the dataframe with the date value
```
```
output.head()  # Displaying the head of the created DataFrame 
```

Finally save the results as a CSV file

```
output.to_csv('scraped_data_14_april.csv',index=False) 

```

```
import os   # Importing the os library to merge all days data

```

```
df = pd.read_csv('./scraped_data_14_april.csv')
df.head(5)

```
We are just reading the data that we saved as CSV file

Note: The data that I have scraped over the days is stored in a folder "all_data"
"all_data" contains files having the scraped data over the days.

```
files = [file for file in os.listdir('../all_data/')] # get all files data from all_data

for file in files:
    print (file)

```
Output
```
scraped_data_14_april.csv
scraped_data_15_april.csv
scraped_data_16_april.csv
scraped_data_17_april.csv
scraped_data_18_april.csv
scraped_data_19_april.csv
scraped_data_20_april.csv
scraped_data_21_april.csv
scraped_data_22_april.csv
scraped_data_23_april.csv
scraped_data_24_april.csv
scraped_data_25_april.csv
```
```
all_data = pd.DataFrame()  # Creating a DataFrame
```

```
files = [file for file in os.listdir('../all_data/')]

for file in files:
    df = pd.read_csv('../all_data/' + file)   
    df['Title'].str.strip()
    df['Title'] = df['Title'].str.replace(r'\s+'," ") 
    # instead of giving the complete file path we write + file which also gives the file path
    # '../all_data/' is the folder and + file is the name of scraped data file for over the days
    
    all_data = pd.concat([all_data, df])   # adding days scraped data to the dataframe
    all_data.head()
```
```
# Power BI Transformation
```  
```
= Table.TransformColumnTypes(#
"Promoted Headers",{{"Category", type text}, {"Sponsored", type text}, 
{"Tag", type text}, {"Title", type text}, {"Stars", type number}, {"Rating", Int64.Type}, {"List_price", Int64.Type},\
 {"Original_price", Currency.Type}, {"Discount", Int64.Type}, {"By_date", type text}, {"Date", type date}, {"ASIN", type text}})

= Table.SplitColumn(#
"Duplicated Title", "Title - Copy", Splitter.SplitTextByEachDelimiter({" "},
 QuoteStyle.Csv, false), {"Title - Copy.1", "Title - Copy.2"})
 
 = Table.SplitColumn(#
"Split Title to get Brand", "Title - Copy.2", Splitter.SplitTextByEachDelimiter({" "},
 QuoteStyle.Csv, false), {"Title - Copy.2.1", "Title - Copy.2.2"})
 
 
 = Table.ReplaceValue(#
"Removed Columns2","(Renewed)",each[Custom],Replacer.ReplaceText,{"Brand"})
 
= Table.ReplaceValue(#
"Replace Renewed with R+Brand","(Refurbished)",each[Custom],Replacer.ReplaceText,{"Brand"}) 
 
 
 = Table.ReplaceValue(#
"Removed Columns3","2020",each[Custom.2],Replacer.ReplaceText,{"Brand"})
 
 
 = Table.ReplaceValue(#
"Replace 2020 with the brand name","2021",each[Custom.2],Replacer.ReplaceText,{"Brand"})
 
 
 = Table.ReplaceValue(#
"Replaced 2021 with brand","2022",each[Custom.2],Replacer.ReplaceText,{"Brand"})
 
 = Table.ReplaceValue(#
"Replaced 2022 with brand","2.5 320GB Portable External Hard Drive Sengert USB3.0 Mobile HDD Storage for PC, Mac, Desktop, Laptop and MacBook 3.0 for PC,Mac, Laptop, Xbox One, Xbox 360(Black)",
"Sengert",Replacer.ReplaceText,{"Brand"})

= Table.ReplaceValue(#
"Replaced 2.5 with brand Sengert","2.5 SAS SATA SSD Hard Drive Carrier Tray Caddy 651687 001 Hard Drive Adapter Bracket for HP DL380 DL560 ML350 ML310E",
"Dpofirs",Replacer.ReplaceText,{"Brand"})


= Table.ReplaceValue(#
"Replaced 2.5 with brand Dpofirs","2.5 USB 3.0 HDD External Mobile Hard Drive Portable HDD Storage Compatible for PC Mac Desktop Laptop Silver 160GB",
"Tooarts",Replacer.ReplaceText,{"Brand"})

= Table.ReplaceValue(#
"Replaced 2.5 with brand Tooarts","Infinity","Infinity JBL",Replacer.ReplaceText,{"Brand"})


= Table.ReplaceValue(#
"Replaced Infinity brand with Infinity JBL","(","",Replacer.ReplaceText,{"Brand"})


= Table.ReplaceValue(#
"Replaced Value ( in brand",")","",Replacer.ReplaceText,{"Brand"})


= Table.ReplaceValue(#
"Replaced Value ) in brand","iphone","mobile",Replacer.ReplaceText,{"Category"})

 = Table.ReplaceValue(#
"Replaced Value iphone with mobile in category","apple+watch","smartwatch",Replacer.ReplaceText,{"Category"})

= Table.ReplaceValue(#
"Replaced Value apple+watch with smart watch in category","macbook","laptop",Replacer.ReplaceText,{"Category"})

= Table.ReplaceValue(#
"Replaced Value macbook with laptop in category","","NO ASIN FOUND",Replacer.ReplaceValue,{"ASIN"})

= Table.ReplaceErrorValues(#
"Removed Custom column from where we got the brand for 2020 and so on", {{"Discount", 0}})

= Table.ReplaceValue(#
"Renamed Original Title to Full Title","New","Apple",Replacer.ReplaceText,{"Brand"})
```

## Top Ratings Table 

 ```
= Table.Group(Source, {"Date", "Full Title", "Rating", "List_price", "Original_price"}, {{"Count", each Table.RowCount(_), Int64.Type}})

= Table.Group(#
"Grouped Rows", {"Date", "Rating", "Full Title"}, {{"Count", each Table.RowCount(_), Int64.Type}})

= Table.Distinct(#
"Grouped Rows1", {"Rating", "Date"})

 ```
## Top Stars Table
 ```
= Table.RemoveColumns(Source,{"Sponsored", "Tag", "By_date"})

= Table.Group(#
"Removed Columns", {"Date", "Full Title", "Rating", "List_price", "Original_price", "Stars","Title"}, 
{{"Count", each Table.RowCount(_), Int64.Type}})


 Table.Group(#
 "Grouped Rows", {"Date", "Rating", "Full Title", "Stars","Title"}, {{"Count", each Table.RowCount(_), Int64.Type}})

 = Table.Distinct(#
 "Grouped Rows1", {"Rating", "Date"})

 = Table.Distinct(#
 "Removed Duplicates2", {"Title"})
 
 ```


# Final Results Verification
![image](https://user-images.githubusercontent.com/61817305/166145562-38c91d9e-f23a-48e2-a6b4-ccf593c164c7.png)

 
 The Top products have a filter on! Displaying only those products having minimum 1000 rating and Stars greater than or equal to 4.5 
 The products in this category are indeed the best products, because even after 1000 ratings they have more than 4.5 Stars. (Stars are Amazon's way of showing how good a product is, which is takes into account different features)
 
 
 Further We also noticed that the Brands that have good rating for a particular product seem to offer very less or no discount while as their competitors are offering decent amount of discount.
 
 Example below:
 ![image](https://user-images.githubusercontent.com/61817305/166146006-e84bfb0f-16bd-4743-8110-16cad6041f50.png)
 Notice that MI brand is doing good in TV category and it's offering very little discount compared to its competitor LG which is not doing that good in TV category compared to MI so to attract more buyers and in pursuit of staying at the top it is offering the highest discount.

### Note: The list price showing up in the Dashboard is without variation.
A single product can have multiple variations.
* Example: Iphone 11 can have multiple variations as 
* Iphone 11 (BLACK)
* Iphone 11 (RED)
* Iphone 11 (YELLOW)
* Iphone 11 (PURPLE)
* and so on, but the list price showing up in the dashboard is just of one product and not all the products combined in the variation.
* All the products in the analysis are unique, without any variation. (Reason: When products are in variation the have same ratings and stars)
*

 # Rating Verification.
### Notice iphone 11 in the Top Products with a rating of 4.60 
* First it follows the filter function in the Top products which only displayes those products having rating more than 1000 and Stars greater than or equal to 4.5
 ![image](https://user-images.githubusercontent.com/61817305/166864822-4e33e5e9-a4e3-4177-931d-af19af245099.png)
 
 ## The reating of Apple Iphone 11 on 17 of April is same as showing in our dashboard
![image](https://user-images.githubusercontent.com/61817305/166864932-9bae3167-7c97-4e96-89f8-ca2a2949bcde.png)
![image](https://user-images.githubusercontent.com/61817305/166865042-7e4a8b09-1327-4ab3-9496-671cf1265b07.png)

 ## The reating of Apple Iphone 11 on 18 of April is same as showing in our dashboard
![image](https://user-images.githubusercontent.com/61817305/166865246-3893356a-2bf9-48f4-84d2-84f266cf112c.png)

![image](https://user-images.githubusercontent.com/61817305/166865284-8458ccd0-b540-4f6a-b73e-e56378bd3144.png)

 ## The reating of Apple Iphone 11 on 22 of April is same as showing in our dashboard
![image](https://user-images.githubusercontent.com/61817305/166865331-21c1767e-0cd9-4bea-968a-0d6e190bb7bb.png)
![image](https://user-images.githubusercontent.com/61817305/166865384-48980028-a8ae-485d-81d7-71d440663f55.png)

 ## The reating of Apple Iphone 11 on 25 of April is same as showing in our dashboard [Apple Iphone 11 went out of stock on 25th of April]
![image](https://user-images.githubusercontent.com/61817305/166865482-19b9713a-0465-4ec8-9406-4b7f430b1832.png)

![image](https://user-images.githubusercontent.com/61817305/166865507-62e2582a-63c3-4428-a58a-15f710ddae33.png)


