# Web Scraper and Google Search Extractor

This project contains two Python scripts for extracting data from webpages. The first script scrapes URLs and their contents from the TradingView website, while the second script searches Google for a given keyword and extracts text from the results. Both scripts save the extracted data to an Excel file for easy analysis.

## Features
- **Web Scraping**: Scrapes webpages for links and extracts the text from those pages.
- **Google Search**: Uses Google search results to gather URLs and their corresponding content.
- **Data Extraction**: Retrieves and cleans up the webpage text for easier analysis.
- **Excel Output**: Saves the URLs and their extracted text into an Excel file.

## Requirements
- Python 3.x
- `requests` library
- `beautifulsoup4` library
- `openpyxl` library
- `googlesearch-python` library (for the second script)

To install the required libraries, run:
```bash
pip install requests beautifulsoup4 openpyxl googlesearch-python
```

## Scripts

### 1. **Web Scraper for TradingView**
This script scrapes the main TradingView website for links that lead to other pages on the TradingView site. It then extracts the text from those pages and saves it to an Excel file.

**How to use:**
1. Run the script.
2. It will scrape the TradingView homepage and follow internal links.
3. The extracted content will be saved to `webpage_data.xlsx`.

```python
import time
import requests
from bs4 import BeautifulSoup
import openpyxl

def main():
    headers = {
        "User-Agent": ("Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                       "AppleWebKit/537.36 (KHTML, like Gecko) "
                       "Chrome/91.0.4472.124 Safari/537.36")
    }
    url = "https://www.tradingview.com"
    response = requests.get(url, headers=headers)

    if response.status_code == 200:
        soup = BeautifulSoup(response.text, "html.parser")
        links = soup.find_all("a")

        urls = []
        texts = []

        for link in links:
            href = link.get("href")
            if href and href.startswith("https://www.tradingview.com"):
                time.sleep(5)
                page_response = requests.get(href, headers=headers)
                if page_response.status_code == 200:
                    page_soup = BeautifulSoup(page_response.text, "html.parser")
                    page_text = page_soup.get_text()

                    cleaned_text = ' '.join(page_text.split())
                    print(cleaned_text)

                    if cleaned_text:
                        urls.append(href)
                        texts.append(cleaned_text)

        workbook = openpyxl.Workbook()
        sheet = workbook.active
        sheet.append(["URL", "Text"])

        for url, text in zip(urls, texts):
            sheet.append([url, text])

        workbook.save("webpage_data.xlsx")
        print("Results saved to webpage_data.xlsx")
    else:
        print("Failed to retrieve the main page")

if __name__ == "__main__":
    main()
```

### 2. **Google Search Extractor**
This script takes a user-provided keyword, searches Google for results, and extracts the content from the top 10 links. The results are then saved to `google_search_results.xlsx`.

**How to use:**
1. Run the script.
2. Enter a keyword when prompted.
3. The script will search Google and extract text from the top 10 results.
4. The extracted content will be saved to `google_search_results.xlsx`.

```python
import time
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin
import openpyxl
from googlesearch import search

def main():
    headers = {
        "User-Agent": ("Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                       "AppleWebKit/537.36 (KHTML, like Gecko) "
                       "Chrome/91.0.4472.124 Safari/537.36")
    }

    keyword = input("Enter keyword to search on Google: ")
    num_results = 10

    urls = []
    texts = []

    print(f"Searching Google for: {keyword}")
    for url in search(keyword, num=num_results, stop=num_results, pause=2):
        print(f"\nProcessing: {url}")
        try:
            response = requests.get(url, headers=headers, timeout=10)
        except Exception as e:
            print(f"Error retrieving {url}: {e}")
            continue

        if response.status_code == 200:
            soup = BeautifulSoup(response.text, "html.parser")
            page_text = soup.get_text()

            cleaned_text = ' '.join(page_text.split())
            print("Extracted text (first 200 chars):", cleaned_text[:200], "...")

            urls.append(url)
            texts.append(cleaned_text)
        else:
            print("Failed to retrieve:", url)

        time.sleep(2)

    workbook = openpyxl.Workbook()
    sheet = workbook.active
    sheet.append(["URL", "Text"])
    for url, text in zip(urls, texts):
        sheet.append([url, text])

    workbook.save("google_search_results.xlsx")
    print("\nResults saved to google_search_results.xlsx")

if __name__ == "__main__":
    main()
```

## Output

Both scripts save the extracted URLs and their corresponding text into an Excel file:
- **webpage_data.xlsx** for TradingView scraping.
- **google_search_results.xlsx** for Google search results.

## Notes

- Be respectful of websites' terms of service and scraping policies.
- The scripts are designed for educational purposes. Make sure to handle requests responsibly by respecting the website's robots.txt file and rate limits.
