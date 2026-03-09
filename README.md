🛒 Grocery Price Scraper

This project scrapes grocery prices from major South African retailers and normalizes the data so that products can be compared across stores.

The long-term goal is to allow users to build a shopping basket and determine the cheapest store (or combination of stores).

Supported Stores

Currently supported retailers:

Checkers – Playwright scraper

Pick n Pay – Playwright scraper (works best with aisle URLs)

Shoprite – HTML + hidden JSON extraction

Makro – Playwright scraper (Work in Progress)

1. Setup
Requirements

Node.js 18+

npm

Install Dependencies
npm install
Install Playwright Browsers

Required for the Checkers, Pick n Pay, and Makro scrapers.

npx playwright install chromium
2. Project Structure
scrapers/
  checkersScraper.js      # Checkers scraper
  pnpScraper.js           # Pick n Pay scraper
  makroScraper.js         # Makro scraper (WIP)
  shopriteScraper.js      # Shoprite scraper (axios + cheerio)
  utils.js                # Price parsing utilities
  masterScraper.js        # Runs all scrapers together

services/
  browser.js              # Playwright browser launcher (stealth enabled)
  normalizeProducts.js    # Product normalization helpers
  productMatcher.js       # Fuzzy product grouping
  basketEngine.js         # Basket-level comparison logic

testMatch.js              # Script to test cross-store product matching
3. Unified Data Format

All scrapers return a standard product object:

{
  "store": "Checkers",
  "name": "Albany Brown Bread 700g",
  "price": 18.99
}

This unified format allows all downstream systems to operate consistently.

4. Product Normalization

Products are normalized to allow reliable cross-store comparison.

normalizeProducts(products) adds additional fields:

{
  "store": "Checkers",
  "name": "Albany Brown Bread 700g",
  "price": 18.99,
  "normalizedName": "albany brown bread",
  "sizeToken": "700g",
  "canonicalKey": "albany brown bread 700g"
}

Normalization includes:

Removing punctuation

Converting to lowercase

Extracting size tokens:

g

kg

ml

L

multipacks

5. Cross-Store Product Matching

Products from different stores are grouped using fuzzy matching.

assignProductIds(products) produces:

{
  "store": "Checkers",
  "name": "Albany Superior Brown Bread 700g",
  "price": 19.99,
  "product_id": "prod_1"
}

All stores selling the same item share the same product_id.

Matching Logic

Matching is based on:

Normalized product names

Size tokens

String similarity threshold (~0.9)

6. Running Individual Scrapers

Run from the project root.

Checkers
node scrapers/checkersScraper.js "Albany Brown Bread 700g"
Shoprite
node scrapers/shopriteScraper.js "milk"
Pick n Pay (Search Term)
node scrapers/pnpScraper.js "bread"
Pick n Pay (Aisle URL)
node scrapers/pnpScraper.js "https://www.pnp.co.za/c/fresh-milk-and-cream44805192"
Makro (Work in Progress)
node scrapers/makroScraper.js "milk"

Each command prints structured JSON output.

7. Running All Scrapers Together

Use the master controller:

node scrapers/masterScraper.js "Albany Brown Bread 700g"

Example output:

[
  {
    "store": "Checkers",
    "name": "Albany Superior Brown Bread 700g",
    "price": 19.99
  },
  {
    "store": "Shoprite",
    "name": "Albany Superior Brown Bread 700g",
    "price": 17.99
  }
]

You can also use it programmatically:

const runAllScrapers = require('./scrapers/masterScraper');

const results = await runAllScrapers('Albany Brown Bread 700g');
8. Basket Comparison Engine

The basket engine compares prices across stores for multiple items.

CLI Usage
node services/basketEngine.js "milk" "bread" "eggs"

or

node services/basketEngine.js "milk,bread,eggs"
Example Output
[
  {
    "item": "milk",
    "matches": [
      { "store": "Checkers", "price": 35.99 },
      { "store": "Shoprite", "price": 16.99 }
    ],
    "cheapest": {
      "store": "Shoprite",
      "price": 16.99
    }
  }
]
9. Current Limitations
Pick n Pay

Free-text search often returns no results.

Works more reliably with aisle URLs.

TODO

Improve selectors

Reverse engineer product tile structure

Makro

Makro pages use heavy dynamic loading and bot protection, making scraping unstable.

TODO

Inspect network requests

Attempt JSON/API extraction instead of DOM scraping

Future Goals

Add more retailers (Woolworths, Spar, Boxer)

Improve fuzzy matching accuracy

Store historical price data

Build a basket optimization engine

Develop a web UI for price comparison
