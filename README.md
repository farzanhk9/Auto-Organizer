import requests
from bs4 import BeautifulSoup
import csv
import json
import os

class WebScraper:
    def __init__(self, base_url):
        self.base_url = base_url
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
        }

    def fetch_page(self):
        """Fetches the HTML content of the page."""
        try:
            response = requests.get(self.base_url, headers=self.headers)
            response.raise_for_status()
            return response.text
        except requests.exceptions.RequestException as e:
            print(f"Error fetching {self.base_url}: {e}")
            return None

    def parse_data(self, html):
        """Parses specific elements from the HTML (Example: Articles)."""
        soup = BeautifulSoup(html, 'html.parser')
        results = []

        # Example: Scraping article titles and links (Modify selectors as needed)
        for item in soup.find_all('div', class_='article-card'):
            data = {
                'title': item.find('h2').get_text(strip=True),
                'link': item.find('a')['href'],
                'category': item.find('span', class_='tag').get_text(strip=True) if item.find('span', class_='tag') else 'N/A'
            }
            results.append(data)
        
        return results

    def save_to_csv(self, data, filename="output.csv"):
        """Saves the scraped data to a CSV file."""
        if not data:
            return
        keys = data[0].keys()
        with open(filename, 'w', newline='', encoding='utf-8') as f:
            dict_writer = csv.DictWriter(f, fieldnames=keys)
            dict_writer.writeheader()
            dict_writer.writerows(data)
        print(f"Successfully saved to {filename}")

    def save_to_json(self, data, filename="output.json"):
        """Saves the scraped data to a JSON file."""
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=4)
        print(f"Successfully saved to {filename}")

if __name__ == "__main__":
    # Example URL (Replace with a real target)
    TARGET_URL = "https://example-blog.com/articles"
    
    scraper = WebScraper(TARGET_URL)
    print("Starting Scraper...")
    
    html_content = scraper.fetch_page()
    if html_content:
        scraped_data = scraper.parse_data(html_content)
        
        if scraped_data:
            scraper.save_to_csv(scraped_data)
            scraper.save_to_json(scraped_data)
        else:
            print("No data found matching the selectors.")
