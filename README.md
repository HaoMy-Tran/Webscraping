# Webscarping
This is a small project that I use BeautifulSoup4 to scrape list of product in [ACFC](https://www.acfc.com.vn/) which is an online shopping website. The objective is to extract information of all the items in Women Clothing (name, brand, and price). 

BeautifulSoup is a simple, user-friendly tool for anyone to learn Web Scraping. However, it also has some limits. 


It is usually used in scraping static websites. When the data become more complex (dynamic), or we require a higher degree of accruracy (data wrapted in check box, radio bottom or search box), other tools like Selenium and Scrapy are recommended. 
For the scope of this project, I will focus on what Beautiful Soup can do in this website and finally export as a CSV file.

Let's get started.
I have checked this website beforehand and found out that the pages for Women's categories have the same structure, so I will pick Clothing without intention. Because of that, my code can be reduced for other categories (Shoes, Bags, etc). 

Fist, let import some of the packages that will be used in this project:
 ```
 from bs4 import BeautifulSoup
 import requests
 import padas as pd
 ```
BeautifulSoup can be installed by pip install. Then, you should let BeautifulSoup to know the URL of the website and what kind of library you want to use:
```
url = 'https://www.acfc.com.vn/nu/trang-phuc-nu.html?p=1'
html_text = requests.get(url).text
```
Then create a soup object with the lxml library:
```
soup = BeautifulSoup(html_text, 'lxml')
```

After done setting up, we need to inspect what is behind the scene of this website https://www.acfc.com.vn/nu/trang-phuc-nu.html by choosing the piece of info and inspecting it. 
Let choose "TOMMY JEANS - Áo Thun Nữ Tay Ngắn Tjw Classic Essential Logo 2 Ss", here's the elements:

```
<li class="item product product-item">
                    <div class="product-item-info" data-container="product-grid">
                        <div class="product-labels">
                                                    </div>
                                                                        <a href="https://www.acfc.com.vn/tommy-jeans-ao-thun-nu-tay-ngan-tjw-classic-essential-logo-2-ss-thj-dw0dw12853-ybr.html" class="product photo product-item-photo" tabindex="-1">
                            
<span class="product-image-container" style="width:546px;">
    <span class="product-image-wrapper" style="padding-bottom: 132.96703296703%;">
        <img loading="lazy" class="product-image-photo" src="https://exacdn.acfc.com.vn/media/catalog/product/cache/e0eb9f74eaaa356eac6b0ce899de6ac0/d/w/dw0dw12853ybr_f_fcjjrcxvpjxxmh1i.png" max-width="546" max-height="726" alt=""><div class="product-label-container bot-mid" style="display: none"></div></span></span>                        </a>
                        <div class="product details product-item-details">
                                                        <strong class="product name product-item-name">
                                <a class="product-item-link" href="https://www.acfc.com.vn/tommy-jeans-ao-thun-nu-tay-ngan-tjw-classic-essential-logo-2-ss-thj-dw0dw12853-ybr.html">
                                    <span class="brand-name">TOMMY JEANS - </span>
                                    Áo Thun Nữ Tay Ngắn Tjw Classic Essential Logo 2 Ss</a>
                            </strong>
                            <div class="price-box price-final_price" data-role="priceBox" data-product-id="719688" data-price-box="product-id-719688"><span class="normal-price">
    

<span class="price-container price-final_price tax weee">
    <span id="product-price-719688" data-price-amount="999000" data-price-type="finalPrice" class="price-wrapper "><span class="price">999.000&nbsp;đ</span></span>
        </span>
</span>

</div>                        </div>
                    </div>
                </li>

```
We can see the string is wrap in a tag named 'span' and class is brand_name'. Tracing back, we can see all the information that we need is in tags named 'li' (list) with class = 'item product product-item'. Let grab all of these tags using find_all method:

```
product_list = soup.find_all('li', class_ = 'item product product-item')
```


The brand names of the products are the text part of 'span' tag with class = 'brand-name'. So I looped through all text of this list and extract all the results in to a new product_brand list using list comprehention:

```
product_brand = [product.find('span', class_ = 'brand-name').text.replace(' -','') for product in product_list]
```
Next, I continue with name of the products. At this point, there is just small thing needed to be noticed. Look at the html file:
```
 <a class="product-item-link" href="https://www.acfc.com.vn/tommy-jeans-ao-thun-nu-tay-ngan-tjw-classic-essential-logo-2-ss-thj-dw0dw12853-ybr.html">
                                    <span class="brand-name">TOMMY JEANS - </span>
                                    Áo Thun Nữ Tay Ngắn Tjw Classic Essential Logo 2 Ss</a>
```
In <a tag, if I used list comprehension here, it would return both products' brand and products' name. Therefore, I needed to subtract  

