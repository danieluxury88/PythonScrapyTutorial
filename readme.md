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
