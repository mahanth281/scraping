import requests
from bs4 import BeautifulSoup
import pandas as pd
from datetime import datetime

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 "
                  "(KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36"
}

BASE_URL = "https://www.amazon.in/s?k=laptop"

def is_ad(soup_item):
    return bool(soup_item.find('span', class_='s-label-popover-default'))

def extract_data(soup):
    product_list = []
    results = soup.find_all('div', {'data-component-type': 's-search-result'})
    
    for item in results:
        title = item.h2.text.strip() if item.h2 else "N/A"
        img_tag = item.find('img')
        image_url = img_tag['src'] if img_tag else "N/A"
        
        rating = item.find('span', {'class': 'a-icon-alt'})
        rating_text = rating.text.strip() if rating else "No rating"
        
        price_whole = item.find('span', {'class': 'a-price-whole'})
        price_fraction = item.find('span', {'class': 'a-price-fraction'})
        price = f"₹{price_whole.text.strip()}.{price_fraction.text.strip()}" if price_whole and price_fraction else "N/A"
        
        ad_status = "Ad" if is_ad(item) else "Organic"
        
        product_list.append({
            "Title": title,
            "Image URL": image_url,
            "Rating": rating_text,
            "Price": price,
            "Result Type": ad_status
        })
    
    return product_list

def main():
    print("[INFO] Sending request to Amazon...")
    response = requests.get(BASE_URL, headers=HEADERS)
    if response.status_code != 200:
        print("[ERROR] Failed to retrieve the webpage.")
        return
    
    soup = BeautifulSoup(response.content, "html.parser")
    print("[INFO] Parsing product data...")
    data = extract_data(soup)
    
    df = pd.DataFrame(data)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    output_filename = f"amazon_laptops_{timestamp}.csv"
    df.to_csv(output_filename, index=False)
    print(f"[SUCCESS] Data saved to {output_filename}")

if __name__ == "__main__":
    main()
