# Web Scraping Course - Comprehensive Notes

## 1. Introduction to Web Scraping

Web scraping is an automated process to extract data from websites, eliminating the need for manual copy-pasting. It's a crucial skill for Data Science and Data Analytics fields, enabling automatic extraction of textual and image data from complete websites.

## 2. What is Web Scraping?

**Definition**: The process of extracting any data from a website, including:
- Images
- Text
- Buttons and links
- Complete database tables

**Primary Use Cases**: 
- Machine Learning data collection
- Converting unstructured data to structured formats
- Automated data gathering for analysis

**Key Concept**: Instead of manually opening each page and copying data, web scraping allows creation of Python scripts to automatically extract all required data.

## 3. Types of Data and Formats

### Data Formats Scraped
- Excel format
- Text format
- Images
- Videos
- Database tables

### Data Extraction Method
All data is extracted via **HTML code**, which contains all text and information on websites. Web scraping uses HTML code to pull data, convert it into readable formats, and make it useful.

### Data Categories

**Structured Data**:
- Organized in table format (databases, Excel sheets)
- Can be directly used for analysis
- Ready for algorithms

**Unstructured Data**:
- Not in structured format
- Includes: video files, audio files, images, PDFs, links
- Requires conversion to structured format for Machine Learning and Data Science
- Algorithms require structured data for processing

## 4. Use Cases of Web Scraping

### Price Comparison
- Scrape product prices from e-commerce sites (Amazon, Flipkart)
- Compare prices and analyze market trends
- Example: Finding current market price of iPhone 15 Pro Max

### Sentiment Analysis
- Analyze customer reviews
- Understand public opinion about products or services

### Content Plagiarism Check
- Verify if content has been used by others
- Extract content for comparison

### News Aggregation
- Companies scrape news from other sources
- Modify and publish on their own platforms

## 5. Web Scraping Terminology

### Crawler (Spider)
- Opens all links on a website
- Collects databases from multiple links
- Operates across entire websites
- Example: Navigating from Python course → HTML course → CSS course

### Scraper
- Stays on a particular web page
- Collects all useful information from that page (headers, images, Q&A sections)
- Provides data in Excel, SQL, or XML files
- Operates on single pages

**Key Difference**: Crawler extracts data by moving across different websites; Scraper extracts data from a single web page.

## 6. Web Scraping Methods

Web scraping collects data from web pages and transfers it into desired formats (Excel, JSON, CSV, databases).

### Methods Available:
1. **Self-Built Web Scraper** (using Python - focus of this course)
2. Pre-Built Web Scrapers
3. Browser Extensions
4. Web Scraping Software
5. Cloud-Based Solutions

## 7. Python Tools for Web Scraping

### Key Libraries:
1. **Scrapy**
2. **BeautifulSoup** (primary focus of this course)
3. **Selenium**

These three are the most powerful and important libraries for web scraping.

## 8. Components of Web Scraping

### Loading the Website
First step: Load the target website for scraping.

### HTML Parsing
- HTML code initially appears unformatted and merged
- Parsing organizes HTML code into correct format
- Available parsers: HTML parsing, XML parsing

### HTML Tree Traversal
Websites have a tree-like structure (branches, sub-branches, leaves), known as the HTML tree.

**HTML Tree Structure Elements**:
```
<html>
  <head>
  <body>
    <div>
    <p> (paragraph)
    <a> (anchor tags)
    <img> (images)
    <h1>, <h2> (headings)
```

**Important Elements to Extract**:
- Images (`<img>`)
- Anchor tags (`<a>`) - crucial for moving between pages
- Heading tags (`<h1>`, `<h2>`)
- Paragraph content (`<p>`)

**Traversing**: "Traveling" through the HTML structure to extract data.

### Data Extraction and Transfer
After parsing and traversing the HTML tree:
1. Extract the desired data
2. Transfer to formats like CSV files or databases

## 9. Legality of Web Scraping

### General Rule
Web scraping is **generally illegal** without permission.

### Best Practices:

**1. Contact Website Owner**
- Ask for permission before scraping
- Essential ethical step

**2. Check robots.txt File**
If direct contact isn't possible (large companies like Amazon):
- Append `/robots.txt` to website URL
  - Example: `wscubetech.com/robots.txt`
  - Example: `amazon.in/robots.txt`
- File specifies:
  - **Disallowed**: Cannot be scraped
  - **Allowed**: Can be scraped
- Extracting disallowed data can lead to legal issues

**3. Search for APIs**
- If an API is available, use it instead
- APIs are designed for data access

**4. Scrape at Slow Rate**
- Do not overwhelm website with too many requests
- High request rates can overload servers
- Can cause legal problems

**5. Read Website Rules and Regulations**
- Adhere to company guidelines
- Follow terms of service

**Critical**: Scraping is illegal unless permission is explicitly given or allowed by robots.txt file.

## 10. Getting HTML Code with Python

### Required Modules

**1. requests module**
- Used to send HTTP requests (GET) to websites
- Fetches website content
- Installation: `pip install requests`

**2. BeautifulSoup module**
- Used to parse HTML content
- Extracts specific data
- Installation: `pip install beautifulsoup4` (referred to as bs4)

### Steps to Get HTML Content
```python
import requests

# Fetch data from URL
web = requests.get("https://tutorialfreak.com/")

# Check response status code
print(web.status_code)
# or
print(web)
```

### Response Status Codes

**200**: 
- Success
- Page is working
- Data can be accessed

**400-series**: 
- Client-side error
- Page not found
- Data cannot be read

**300-series**: 
- Redirection
- Page redirects to another URL

### Accessing Content
```python
# Get raw HTML content
print(web.content)

# Get URL
print(web.url)
```

**Note**: Raw HTML from `web.content` is usually messy and needs parsing.

## 11. HTML Parsing with BeautifulSoup

### HTML Parsing Definition
Extracting data from HTML code and making it "pretty" or formatted.

### Pretty-fying
- HTML code from `requests.get()` is not in readable, tree-like structure
- Pretty-fying converts messy code into well-structured, indented format
- Similar to browser developer tools view

### Required Modules
- `requests` (to get HTML code)
- `BeautifulSoup` (to parse and pretty-fy HTML)

### Steps for HTML Parsing
```python
import requests
from bs4 import BeautifulSoup

# Fetch HTML content
web = requests.get("https://tutorialfreak.com/")

# Always check robots.txt and status_code before proceeding!

# Access raw content
web.content  # Shows unformatted output

# Create BeautifulSoup object
soup = BeautifulSoup(web.content, 'html.parser')
# web.content: raw HTML data to parse
# 'html.parser': parser to use (other options: 'lxml', 'xml', 'html5lib')

# Pretty-fy the HTML
print(soup.prettify())  # Outputs clean, indented, readable format
```

The `soup` object now contains parsed HTML, allowing easier extraction of specific data elements.

## 12. Finding Elements by Tag Name

### Finding First Occurrence
```python
# Find first instance of a specific tag
first_p_tag = soup.find('p')
```

### Finding All Occurrences
```python
# Find all instances of a tag (returns a list)
all_a_tags = soup.find_all('a')

# Iterate through list
for tag in all_a_tags:
    print(tag)
```

### Extracting Text and Attributes

**Get Text Content**:
```python
print(first_p_tag.text)
# Only prints text within the tag
```

**Get Attributes** (treat tag as dictionary):
```python
# Get href attribute from anchor tag
print(first_a_tag['href'])

# Get src attribute from image tag
image_tag = soup.find('img')
print(image_tag['src'])
```

### Finding Specific Tags with Classes or IDs

**Finding by Class** (note the underscore - `class` is reserved in Python):
```python
element = soup.find('div', class_='my-class')

# Example: find div with copyright-info class
copyright_div = soup.find('div', class_='copyright-info')

# Example: find img with img-fluid class and extract src
img = soup.find('img', class_='img-fluid')
print(img['src'])
```

**Finding by ID**:
```python
element = soup.find('div', id='my-id')
```

**Combining Tag and Class/ID**:
```python
soup.find('a', class_='button')
```

**Finding with Multiple Classes**:
```python
soup.find('div', class_=['class1', 'class2'])
```

**Finding with Specific Attributes**:
```python
# Use attrs dictionary for complex attribute matching
soup.find('div', attrs={'data-id': '123'})
```

## 13. Finding Elements by ID

Finding elements by ID is straightforward using the `id` parameter:
```python
# Find element with specific ID
element = soup.find(id='some_id')

# Example: find img by id and extract src and alt
img_tag = soup.find('img', id='my-image')
print(img_tag['src'])
print(img_tag['alt'])
```

## 14. Finding Parent and Sibling Elements

### Accessing Parent Element
```python
# Get immediate parent tag
parent_tag = element.parent

# Example
div_tag = soup.find('div', class_='example')
print(div_tag.parent)
```

### Accessing Sibling Elements

**Next and Previous Siblings**:
```python
# Navigate to adjacent siblings
next_sib = element.next_sibling
prev_sib = element.previous_sibling

# Note: Often returns newline/whitespace
# May need to apply multiple times to find actual element
```

**Next and Previous Elements**:
```python
# Traverse through all elements in parsed tree (including text nodes)
next_elem = element.next_element
prev_elem = element.previous_element
```

### Accessing Children Elements
```python
# Get iterator for direct children
children = element.children

# Loop through children
div = soup.find('div', class_='parent')
for child in div.children:
    print(child)
```

## 15. HTML Traversing

**Definition**: Process of navigating through the parsed HTML tree.

**Capabilities**:
- Move **up** (to parents)
- Move **down** (to children)
- Move **sideways** (to siblings)

**Importance**: Data is often nested; need to pinpoint specific elements relative to others.

## 16. Extracting Attributes from Tags

Element objects behave like dictionaries for their attributes:
```python
# Extract attribute
value = tag['attribute_name']

# Example: extract src and alt from image
img_tag = soup.find('img')
src = img_tag['src']
alt = img_tag['alt']

# Handle missing attributes with get() method
# Returns None if attribute doesn't exist (prevents errors)
alt = img_tag.get('alt')
```

## 17. Modifying HTML Content

### Modifying Tag Attributes
```python
# Change existing attribute or add new one
tag['attribute_name'] = 'new_value'

# Example: change src attribute
img_tag['src'] = 'new_image.jpg'

# Delete attribute
del tag['attribute_name']
```

### Modifying Tag Text Content
```python
# Change text content
tag.string = 'New Text'
# or
tag.text = 'New Text'

# Example: modify paragraph text
p_tag = soup.find('p')
p_tag.string = 'Updated paragraph content'
```

### Adding New Tags
```python
# Create new Tag objects
# Add to tree using append() or insert()

# Example: append new <p> tag to div
div = soup.find('div')
new_p = soup.new_tag('p')
new_p.string = 'This is a new paragraph'
div.append(new_p)
```

## 18. Practical Example: Extracting Course Titles

**Goal**: Extract titles of various courses from website.

**Steps**:

1. **Inspect HTML**: Use browser developer tools
2. **Identify Patterns**: Course titles are in `<h3>` tags with specific class
3. **Use find_all() with class**:
```python
course_titles = soup.find_all('h3', class_='course-title')
```
4. **Iterate and Extract Text**:
```python
titles_list = []
for title in course_titles:
    # Extract text and clean up
    clean_title = title.text.strip()
    titles_list.append(clean_title)
```
5. **Print Results**:
```python
for title in titles_list:
    print(title)
```

## 19. Practical Example: Extracting Image URLs

**Goal**: Get src attribute (URL) of various images on page.

**Steps**:

1. **Inspect Image HTML**: Find `<img>` tag and attributes
2. **Use find_all() for images**:
```python
all_images = soup.find_all('img')
```
3. **Iterate and Extract src**:
```python
image_urls = []
for img in all_images:
    src = img.get('src')  # Use get() to handle missing src
    if src:
        image_urls.append(src)
```
4. **Print URLs**:
```python
for url in image_urls:
    print(url)
```

## 20. Introduction to Selenium

### Why Selenium?

**Limitations of requests + BeautifulSoup**:
- Good for static HTML only
- Cannot handle JavaScript-generated content

**Selenium Advantages**:
- Interacts with web pages like a real browser
- Can click buttons
- Can fill forms
- Can wait for content to load
- Essential for dynamic websites with client-side JavaScript

## 21. Setting Up Selenium and WebDriver

### Installation
```bash
pip install selenium
```

### WebDriver Setup
1. Download appropriate WebDriver (e.g., ChromeDriver for Chrome)
2. **Version must match browser version**
3. Place executable in accessible location (project directory or system PATH)

### Basic Usage
```python
from selenium import webdriver

# Initialize browser instance
driver = webdriver.Chrome(executable_path='path/to/chromedriver')

# Open URL
driver.get('http://example.com')
# Browser automatically opens and navigates
```

### Finding Elements

**Older methods**:
- `find_element_by_id()`
- `find_element_by_name()`
- `find_element_by_class_name()`
- `find_element_by_tag_name()`
- `find_element_by_xpath()`
- `find_element_by_css_selector()`

**Modern approach**:
```python
from selenium.webdriver.common.by import By

# Find single element
element = driver.find_element(By.ID, 'element_id')

# Find multiple elements
elements = driver.find_elements(By.CLASS_NAME, 'class_name')

# Extract text
text = element.text
```

### Clicking Elements
```python
# Find button and click
button = driver.find_element(By.ID, 'submit-button')
button.click()
```

## 22. Selenium for Filling Forms

**Steps**:

1. **Identify Input Fields**: Find by id, name, or other locators
2. **Send Keys**: Type text into input field
```python
input_field = driver.find_element(By.NAME, 'search')
input_field.send_keys('python')
```
3. **Submit Form**: Click submit button or use submit()
```python
submit_button = driver.find_element(By.ID, 'search-button')
submit_button.click()
# or
form_element.submit()
```

**Example**: Navigate to search bar, type "python," click search button.

## 23. Handling Dynamic Content and Waits

### Why Waits?
Dynamic websites load content asynchronously; elements may not be immediately available.

### Implicit Waits
Set global wait time for elements to appear:
```python
driver.implicitly_wait(10)  # Wait up to 10 seconds
```
Driver waits up to this duration before throwing error if element not found.

### Explicit Waits
Wait for specific condition before proceeding:
```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

# Wait for element to be present
element = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, 'some_id'))
)
```

**Use Case**: Script might fail without waits if elements take time to load.

## Summary

### Topics Covered:
1. **Web Scraping Basics**: Definition, use cases, legality
2. **requests Module**: Fetching HTML content, status codes
3. **BeautifulSoup**: Parsing, prettifying, finding elements, traversing tree
4. **Selenium**: Setup, interacting with dynamic content, waits

### Key Takeaways:
- Always check `robots.txt` and get permission
- Use appropriate tools: BeautifulSoup for static content, Selenium for dynamic
- Practice scraping ethically and legally
- Start simple and build to more complex scrapers

### Next Steps:
- Practice on real websites
- Explore advanced techniques
- Combine tools for comprehensive scraping solutions
- Study more complex scenarios and edge cases
