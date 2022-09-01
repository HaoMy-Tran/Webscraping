## Webscraping (online shopping website)
### Intro
This is a mini project that I used ```BeautifulSoup4``` to scrape list of products at [ACFC](https://www.acfc.com.vn/) which is an online shopping website. The objective is to extract information of all the items in Women Clothing (name, brand, price, link, and image). 
```BeautifulSoup``` is a simple, user-friendly tool for anyone to learn Web Scraping. For the scope of this project, I will focus on what ```BeautifulSoup``` can do in this website and finally export the results to CSV file.

### File reading guide
Along with this, I also upload a .ipynb file. In this Jupyter Notebook, I scraped the first 4 pages of the Women Clothing. However, more pages and categories could be parsed by changing the generic_url and page_last parameters which I will mention later. Just one more thing to be noticed, the scraping results might change because the owner will change the products and information over time.  

Let's get started.
### Project guide
I had checked this website ahead of time and found out that all the pages for Women's categories have the same structure> Therefore, my code can be reduced for other categories (Shoes, Bags, etc). I picked the first one which is Clothing. 

**Setting up** <br />
First, I imported some of the packages that would be used in this project:
 ```
 from bs4 import BeautifulSoup
 import requests
 import padas as pd
 import csv
 ```
In case you do not have ```BeautifulSoup``` installed in your PC, you can install it by pip install. Then, I let ```BeautifulSoup``` know the URL of the website and what kind of library I wanted to use:
```
url = 'https://www.acfc.com.vn/nu/trang-phuc-nu.html?p=1'
html_text = requests.get(url).text
```
Then I created a soup object with the ```lxml``` library:
```
soup = BeautifulSoup(html_text, 'lxml')
```

After done setting up, I found out what was happening behind the scene of [this website](https://www.acfc.com.vn/nu/trang-phuc-nu.html) by choosing the piece of info and inspecting it. 
I chose the name of a product and here's the elements:

![image](https://user-images.githubusercontent.com/106227875/187222231-1a4204e1-440f-4700-8211-627fb0976d47.png)

We can see the string is nested in a tag named ```<span``` and ```class="brand_name"```. Tracing back, all the information that I needed is in tags named ```<li``` (list) with ```class = 'item product product-item'```. I grabed all of these tags using ```find_all``` method:

```
product_list = soup.find_all('li', class_ = 'item product product-item')
```

**Brand** <br />
The brand names of the products are the text part of ```<span``` tag with ```class="brand-name"```. So I iterated through all text of this list and extract all the results in to a new product_brand list using list comprehension:

```
product_brand = [product.find('span', class_ = 'brand-name').text.replace(' -','') for product in product_list]
```
**Name** <br />
Similar to Brand but just one small thing to be notice. The names can not be extracted alone because there is a ```<span>``` tag, which includes brand name, wrapted in the ```<a>``` tag. Look at this:
```
<a class="product-item-link" href="https://www.acfc.com.vn/ck-underwear-quan-lot-nu-bikini-fit-ck-qf6848ad-20g.html">
    <span class="brand-name">CK UNDERWEAR - </span>
           Áo Kiểu Nữ Tay Dài Ls Cropped Wra </a>
```
So I extracted the string ```'Áo Kiểu Nữ Tay Dài Ls Cropped Wra'``` by using ```content()``` method:
```
product_name = [product.find('a', class_ ='product-item-link').content[2].strip() for product in product_list]
```
**Price** <br />
Similarly, list of products' price was created by:

```
product_price = [product.find('span', class_ = 'price').text.replace('đ','').strip() for product in product_list]
```
**Link** <br />
```
product_link = [product.find('a', class_ = 'product-item-link').get('href') for product in product_list]
```
**Image** <br />
```
product_image = [product.find('img', class_='product-image-photo').get('src') for product in product_list]
```
**Pagination** <br />
All the products are paginated into 213 pages. To grab all the information, the simplest way is to iterate through all the pages by leveraging the number of page appearing at the end of the URL. To do this, I changed the structure of the code a little bit and introduce a ```function``` takes URL, number of web page, and number of the last page as parameters.
First, I created the emty lists of the 5 attributes:
```
product_name = []
product_brand = []
product_price = []
product_link = []
product_image = []
```
I defined a new ```function```:
```
def pages(generic_url, page_number, page_last)
```
```generic_url``` (url without page number) can be other URLs of Women categories because they have the same ```html``` file structure. Then I created a while loop for ```page_number``` and established new rules for setting up objects:
```
def pages(generic_url, page_number, page_last)
   while page_number <= page_last:
     url = generic_url + str(page_number)
     html_text = requests.get(url).text
     soup = BeautifulSoup(html_text, 'lxml')
     product_list = soup.find_all('li', class_ = 'item product product-item')

```
For each of the attribute, I added the new list of string into the innitial emty list. I used ```extend()``` rather than ```append()``` because I did not want the iterations to create new sub-lists. I simply extended the innitial lists:
```
product_brand.extend([product.find('span', class_ = 'brand-name').text.replace(' -','') for product in product list])
product_name.extend([product.find('a', class_='product-item-link').contents[2].strip() for product in product_list])
product_price.extend([product.find('span', class_ = 'price').text.replace('đ','').strip() for product in product_list])
product_link.extend([product.find('a', class_='product-item-link').get('href') for product in product_list])
product_image.extend([product.find('img', class_='product-image-photo').get('src') for product in product_list])
```
Generate the URL for next pages by adding 1 to ```page_number```
```
page_number += 1
```
Finally, I inputed values for the three parameters
```
pages(https://acfc.com.vn/nu/trang-phuc-nu.html?p=',1,213)
```
### Create a data frame and export to CSV file. 
I used ```pandas``` to create a new dataframe and converted it to csv file:
 ```
 product_info = pd.DataFrame({
   'Brand': product_brand,
   'Name': product_name,
   'Price': product_price,
   'Link': product_link,
   'Image': product_image
 })

product_info.to_csv('product_info.csv', index = False, encoding = 'utf-8')
```

### Limits
When looking at the website, on the left side is the filter to maximize the the search results. ```BeautifulSoup``` can not do anything on this part. Rather, some other tools like ```Selenium``` or ```Scrapy``` are suggested.

