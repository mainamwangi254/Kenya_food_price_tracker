# 📓 Naivas Online Scraper — Complete Code Guide

> A beginner-friendly explanation of every cell, every function, and every decision made in the Naivas scraper notebook.

---

## 📊 What This Scraper Collected

| Category | Products |
|---|---|
| 🥫 Food Cupboard | 300+ |
| 🥦 Fresh Food | 300+ |
| 📱 Electronics | 256 |
| 🍾 Naivas Liquor | 300+ |
| **Total** | **1,156+ products** |

---

## 🌐 What is Web Scraping?

Web scraping is the process of **automatically collecting data from websites**. Instead of manually copying product names and prices one by one, we write a Python program that:

- Visits the website just like your browser does
- Reads the HTML code of the page
- Finds and extracts the specific data we want
- Saves it neatly into a CSV or Excel file

> 💡 Think of it like a robot that reads a menu and writes down every item and price for you automatically.

---

## 🤖 robots.txt — Rules of the Road

Before scraping any website we always check its `robots.txt` file. This is a public file that tells bots what they are and are not allowed to scrape.

The Naivas robots.txt at `https://naivas.online/robots.txt` said:

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Sitemap: https://naivas.info/wp-sitemap.xml
```

| Directive | What it means |
|---|---|
| `User-agent: *` | Rules apply to ALL bots including our scraper |
| `Disallow: /wp-admin/` | Never touch the admin panel |
| `Allow: /wp-admin/admin-ajax.php` | One admin endpoint is open |
| `Sitemap:` | A map of all pages on the site |

> ✅ No product or category pages were blocked — we were fully allowed to scrape them.

> ⚠️ Always check robots.txt before scraping any new website. Ignoring it is unethical and potentially illegal.

---

## 📦 Libraries Used

| Library | What it does | How we used it |
|---|---|---|
| `requests` | Sends HTTP requests to websites | Used in the session to visit each category page |
| `cloudscraper` | Bypasses Cloudflare bot protection | Replaced requests after getting 403 blocked errors |
| `BeautifulSoup` | Reads and searches through HTML | Used to find product cards, names and prices |
| `pandas` | Organises data into tables | Used to create the final table and save to CSV/Excel |
| `time` | Controls timing and pauses | Used to wait between requests politely |
| `openpyxl` | Allows pandas to write Excel files | Used automatically when saving to `.xlsx` |

---

## 📓 Cell by Cell Explanation

---

### Cell 1 — Import Libraries

The first thing any Python script does is import the tools it needs.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time
```

- **`import requests`** — loads the requests library so we can visit websites
- **`from bs4 import BeautifulSoup`** — loads only BeautifulSoup from the bs4 library
- **`import pandas as pd`** — loads pandas with the short nickname "pd"
- **`import time`** — loads the time library for adding delays

> 💡 The `as pd` part is just a shortcut. Instead of typing `pandas.DataFrame()` we type `pd.DataFrame()`.

---

### Cell 2 — Settings and Configuration

All important settings are defined here in one place. If anything needs changing, you only edit this cell.

#### HEADERS

When your browser visits a website it sends information about itself called "headers". Without headers, websites detect our request as a bot and block it.

```python
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
    "Accept": "text/html,application/xhtml+xml...",
    "Accept-Language": "en-US,en;q=0.9",
    "Referer": "https://naivas.online/",
}
```

| Header | What it does |
|---|---|
| `User-Agent` | Identifies our scraper as Chrome browser on Windows |
| `Accept` | Tells the site what type of content we can receive |
| `Accept-Language` | Tells the site we prefer English |
| `Referer` | Makes it look like we came from their homepage |

#### DELAY_SECONDS

How many seconds we wait between each request. Set to 3 seconds.

> ⚠️ Never set below 2 seconds. Too fast = IP ban and server overload.

#### CATEGORIES

A dictionary mapping category names to their URLs.

```python
CATEGORIES = {
    'Food Cupboard' : 'https://naivas.online/food-cupboard',
    'Fresh Food'    : 'https://naivas.online/fresh-food',
    'Electronics'   : 'https://naivas.online/electronics',
    'Naivas Liquor' : 'https://naivas.online/naivas-liqour',
}
```

---

### Cell 3 — Create a Cloudscraper Session

Originally this cell used `requests.Session()`. However Naivas uses **Cloudflare protection** which blocked our requests with a 403 error. We switched to `cloudscraper`.

```python
import cloudscraper

scraper = cloudscraper.create_scraper(
    browser={"browser": "chrome", "platform": "windows", "mobile": False}
)
scraper.headers.update(HEADERS)
session = scraper
```

- **`create_scraper()`** — creates a scraper that mimics Chrome on Windows
- **`scraper.headers.update(HEADERS)`** — adds our custom headers on top of cloudscraper's built-in ones
- **`session = scraper`** — assigns the scraper to the name "session" so all other cells work unchanged

> 💡 The last line `session = scraper` is clever — it means we did not have to rewrite Cells 4, 5 and 6 when we switched from requests to cloudscraper.

#### Why We Got 403 Originally

A 403 error means "Forbidden". Naivas uses Cloudflare which detects bots by checking for missing cookies, suspicious headers, and request patterns. Cloudscraper solves all of this automatically.

---

### Cell 4 — Function: fetch_page()

A **function** is a reusable block of code. This function visits a URL and returns the HTML content.

```python
def fetch_page(url):
    response = session.get(url, timeout=30)
    if response.status_code == 200:
        return response.text
```

HTTP status codes tell us what happened:

| Code | Meaning | What we do |
|---|---|---|
| 200 | Success | Return the HTML and continue |
| 403 | Forbidden — blocked | Print error and return None |
| 404 | Page not found | Print error and return None |
| Timeout | Site too slow | Print error and return None |

> 💡 We increased timeout from 15 to 30 seconds after Food Cupboard kept timing out.

---

### Cell 5 — Function: extract_products()

This function takes the raw HTML of a page and extracts product names, prices and links using BeautifulSoup.

#### How We Found the Right Selectors

Naivas uses a **custom Laravel/Livewire application** — not standard WooCommerce as we initially assumed. We ran several diagnostic cells to find the correct HTML structure.

| What we want | CSS selector used |
|---|---|
| Price container | `.product-price` |
| Product name | `.line-clamp-2` |
| Current price (green) | `.text-naivas-green` |
| Original price (red/crossed) | `.line-through` |
| Product URL | `a` (first anchor tag) |

#### The Navigation Logic

Since `product-price` is nested inside the product card, we navigate up the HTML tree:

```python
card = price_el.parent.parent
```

> `.parent` goes one level up in the HTML tree. Two `.parent` calls gets us to the full product card.

#### Extracting Text

```python
name = name_el.get_text(strip=True)
```

`.get_text(strip=True)` extracts visible text and removes extra spaces and newlines.

> ⚠️ The function must end with `return products` at 4 spaces indentation — outside the for loop but inside the function. Missing this was a bug we fixed during development.

---

### Cell 6 — Run the Scraper with Pagination

This cell loops through each category, goes through all pages, and collects all products.

#### What is Pagination?

Naivas shows 15 products per page. To get all products we visit every page:

| Page | URL |
|---|---|
| Page 1 | `https://naivas.online/food-cupboard` |
| Page 2 | `https://naivas.online/food-cupboard?page=2` |
| Page 3 | `https://naivas.online/food-cupboard?page=3` |
| ... | Until a page returns 0 products |

#### The While Loop

```python
while page_number <= MAX_PAGES:
```

A **while loop** keeps repeating as long as a condition is true. It stops when either `MAX_PAGES` is reached or a page returns 0 products.

#### The Stop Condition

```python
if len(products) == 0:
    break
```

`break` immediately exits the while loop when a page has 0 products.

#### Results

| Category | Pages | Products |
|---|---|---|
| Food Cupboard | 20 (limit hit — more exist) | 300 |
| Fresh Food | 20 (limit hit — more exist) | 300 |
| Electronics | 18 (last page found) | 256 |
| Naivas Liquor | 20 (limit hit — more exist) | 300 |

> 💡 Change `MAX_PAGES = 50` to collect even more products.

---

### Cell 7 — Preview the Data

```python
df = pd.DataFrame(all_products)
df.head(10)
```

- **`pd.DataFrame()`** — converts our list of products into a table with rows and columns
- **`.head(10)`** — shows the first 10 rows to verify data looks correct

---

### Cell 8 — Save the Data

```python
df.to_csv('naivas_products.csv', index=False, encoding='utf-8-sig')
df.to_excel('naivas_products.xlsx', index=False)
```

| Format | Best for |
|---|---|
| CSV | Universal — opens in Excel, Google Sheets, Python |
| Excel (.xlsx) | Sharing, charts, filtering |

- **`index=False`** — do not add an extra row-number column
- **`encoding='utf-8-sig'`** — ensures KES currency symbol displays correctly in Excel

---

## 🔬 Diagnostic Cells Explained

These were temporary cells used during development to understand the website HTML structure.

---

### Diagnostic 1 — Print All Class Names

When initial selectors returned 0 products, we printed every CSS class on the page:

```python
for tag in soup.find_all(True):
    for cls in tag.get('class', []):
        all_classes.add(cls)
```

- **`soup.find_all(True)`** — finds every HTML tag on the page
- **`tag.get('class', [])`** — gets class attribute, returns `[]` if none exists

This revealed `product-price`, `product-card-img` and other useful class names.

---

### Diagnostic 2 — Print Raw Page HTML

```python
print(soup.prettify()[:3000])
```

- **`.prettify()`** — formats HTML with proper indentation
- **`[:3000]`** — takes only the first 3000 characters

> 💡 This confirmed the site uses Laravel/Livewire — NOT WordPress WooCommerce. This explained why standard selectors like `li.product` returned nothing.

---

### Diagnostic 3 — Test Selectors

```python
cards = soup.select('.item')
print(cards[0].prettify())
```

- **`soup.select('.item')`** — finds all elements with class "item"
- **`[0]`** — gets the first item (Python counts from 0 not 1)

This revealed `.item` was returning navigation menu items, not product cards.

---

### Diagnostic 4 — Find Product via Price Element

```python
prices = soup.select('.product-price')
print(prices[0].parent.parent.prettify())
```

- **`.parent.parent`** — navigates two levels up the HTML tree

This showed the complete product card structure containing name, price and link.

---

### Diagnostic 5 — Test Pagination

```python
test_urls = [
    'https://naivas.online/food-cupboard?page=2',
    'https://naivas.online/food-cupboard?p=2',
]
```

Both returned 15 products confirming `?page=2` is the correct pagination pattern.

---

## 🐛 Errors Encountered and Fixed

| Error | Cause | Fix |
|---|---|---|
| 403 Forbidden | Cloudflare detected bot traffic | Switched to cloudscraper |
| 0 products found | Wrong CSS selectors | Used diagnostic cells to find correct ones |
| NoneType not iterable | Missing `return products` statement | Added return at correct indentation |
| IncompleteInputError | Missing closing bracket `)` | Added missing bracket |
| Timeout errors | Website response > 15 seconds | Increased timeout to 30 seconds |
| ModuleNotFoundError openpyxl | Library not installed | Ran `pip install openpyxl` |

---

## 🚀 What to Build Next

- [ ] Increase `MAX_PAGES = 50` to collect more products
- [ ] Add more Naivas categories
- [ ] Add `date_scraped` column for price history tracking
- [ ] Standardise output format to match other scrapers
- [ ] Combine with Quickmart and Carrefour data

---

*Part of the Kenya Food Price Tracker project 🇰🇪*
