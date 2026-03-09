# Grocery Price Scraper

This project scrapes grocery prices from major South African retailers and normalizes the data so that products can be compared across stores. The long-term goal is to allow users to build a shopping basket and see the cheapest store or store combination.

Currently supported stores:

Checkers – Playwright scraper

Pick n Pay – Playwright scraper (works best with aisle URLs)

Shoprite – HTML + hidden JSON extraction

Makro – Playwright scraper (work in progress)

1. Setup
Requirements

Node.js 18+

npm

Install dependencies
npm install
Install Playwright browsers

Required for the Checkers, Pick n Pay, and Makro scrapers.

npx playwright install chromium
2. Project Structure

Key files in the project:

scrapers/
  checkersScraper.js     -> Checkers scraper
  pnpScraper.js          -> Pick n Pay scraper
  makroScraper.js        -> Makro scraper (WIP)
  shopriteScraper.js     -> Shoprite scraper (axios + cheerio)
  utils.js               -> price parsing utilities
  masterScraper.js       -> runs all scrapers together

services/
  browser.js             -> Playwright browser launcher (stealth enabled)
  normalizeProducts.js   -> product normalization helpers
  productMatcher.js      -> fuzzy product grouping
  basketEngine.js        -> basket-level comparison logic

testMatch.js             -> script to test cross-store product matching
3. Unified Data Format

All scrapers return a standard product object:

{
  "store": "Checkers",
  "name": "Albany Brown Bread 700g",
  "price": 18.99
}

This unified format allows all downstream systems to operate consistently.

4. Product Normalization

Products are normalized to make cross-store comparisons possible.

normalizeProducts(products) adds:

{
  "store": "Checkers",
  "name": "Albany Brown Bread 700g",
  "price": 18.99,
  "normalizedName": "albany brown bread",
  "sizeToken": "700g",
  "canonicalKey": "albany brown bread 700g"
}

Normalization includes:

removing punctuation

lowercasing

extracting size tokens (g, kg, ml, L, multipacks)

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

Matching uses:

normalized names

size tokens

string similarity threshold (~0.9)

6. Running Individual Scrapers

From the project root:

Checkers
node scrapers/checkersScraper.js "Albany Brown Bread 700g"
Shoprite
node scrapers/shopriteScraper.js "milk"
Pick n Pay (search term)
node scrapers/pnpScraper.js "bread"
Pick n Pay (aisle URL)
node scrapers/pnpScraper.js "https://www.pnp.co.za/c/fresh-milk-and-cream44805192"
Makro (work in progress)
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

Run from CLI:

node services/basketEngine.js "milk" "bread" "eggs"

or

node services/basketEngine.js "milk,bread,eggs"

Example output:

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

TODO:

improve selectors

reverse engineer product tile structure

Makro

Heavy dynamic loading and bot protection.

Current Playwright approach is fragile.

TODO:

inspect network requests

attempt JSON/API extraction instead of DOM scraping
