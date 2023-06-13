## Configure Virtual Environment and Install Scrapy
- python3 -m venv venv
- ource venv/bin/activate
- pip install scrapy
- scrapy

## Create Scrapy Project and Spider
- Create Scrapy project
```
scrapy startproject bookscraper
```
- Create Spider on spiders folder
```
scrapy genspider bookspider books.toscrape.com
```
- Install and configure ipython
```
pip install ipython
on scrappy.cfg add:
[settings]
default = bookscraper.settings
shell = ipython
```
- Open scrapy shell
```
scrapy shell
fetch('https://books.toscrape.com')
response.css('article.product_pod')
response.css('article.product_pod').get()
books = response.css('article.product_pod')
len(books)
```
- Get a single book information on shell to test
```
book = books[0]
book.css('h3 a::text').get()
book.css('.product_price .price_color').get()
book.css('.h3 a').attrib['href']
```
- Add Functions on Spider (on parse function of spider)

```
class BookspiderSpider(scrapy.Spider):
    name = "bookspider"
    allowed_domains = ["books.toscrape.com"]
    start_urls = ["https://books.toscrape.com"]

    def parse(self, response):
        books = response.css('article.product_pod')
        
        for book in books:
            yield{
                'name' : book.css('h3 a::text').get(),
                'price' : book.css('.product_price .price_color').get(),
                'url' : book.css('h3 a').attrib['href'],
            }
```
- Test functionality by executing
```
scrapy crawl bookspider
```

## Check on other paginated pages
- Enter scrapy shell
```
scrapy shell
fetch('https://books.toscrape.com')
** Get next link **
- response.css('li.next a ::attr(href)').get()
```

- Add code to Spider parse method
```
class BookspiderSpider(scrapy.Spider):
    name = "bookspider"
    allowed_domains = ["books.toscrape.com"]
    start_urls = ["https://books.toscrape.com"]

    def parse(self, response):
        books = response.css('article.product_pod')
        
        for book in books:
            yield{
                'name' : book.css('h3 a::text').get(),
                'price' : book.css('.product_price .price_color').get(),
                'url' : book.css('h3 a').attrib['href'],
            }
        
        next_page = response.css('li.next a ::attr(href)').get()
        if next_page is not None:
            next_page_url = 'https://books.toscrape.com/' + next_page
            yield response.follow(next_page_url, callback = self.parse)

```


- Test functionality by executing
```
scrapy crawl bookspider
```

## Get each item details
- Navigate to each item details page, inspect html and test on scrapy shell
- Add code on spider:
```
class BookspiderSpider(scrapy.Spider):
    name = "bookspider"
    allowed_domains = ["books.toscrape.com"]
    start_urls = ["https://books.toscrape.com"]

    def parse(self, response):
        books = response.css('article.product_pod')
        
        for book in books:
            relative_url = book.css('h3 a ::attr(href)').get()
            if 'catalogue/' in relative_url:
                book_url = 'https://books.toscrape.com/' + relative_url
            else:
                book_url = 'https://books.toscrape.com/catalogue/' + relative_url
            yield response.follow(book_url, callback = self.parse_book_page)
        
        next_page = response.css('li.next a ::attr(href)').get()
        if next_page is not None:
            if 'catalogue/' in next_page:
                next_page_url = 'https://books.toscrape.com/' + next_page
            else:
                next_page_url = 'https://books.toscrape.com/catalogue/' + next_page
            yield response.follow(next_page_url, callback = self.parse)

    def parse_book_page(self, response):
        table_rows = response.css("table tr")
        print(response.css('p.price_color ::text').get())
        yield {
            'url': response.url,
            'title': response.css('.product_main h1::text').get(),
            'product_type': table_rows[1].css("td ::text").get(),
            'price_excl_tax': table_rows[2].css("td ::text").get(),
            'price_incl_tax': table_rows[3].css("td ::text").get(),
            'tax': table_rows[4].css("td ::text").get(),
            'availabilty': table_rows[5].css("td ::text").get(),
            'num_reviews': table_rows[6].css("td ::text").get(),
            'stars': response.css("p.star-rating").attrib['class'],
            'category': response.xpath("//ul[@class='breadcrumb']/li[@class='active']/preceding-sibling::li[1]/a/text()").get(),
            'description': response.xpath("//div[@id='product_description']/following-sibling::p/text()").get(),
            'price': response.css('p.price_color ::text').get(),
        }

```
- Export results to json and csv
```
scrapy crawler bookspider -O bookdata.csv
scrapy crawler bookspider -O bookdata.json
```