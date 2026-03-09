# Grocery Price Scraper
This project scrapes prices for grocery products from major South African retailers and normalizes them so you can compare baskets across stores.

Current stores:
Checkers (Playwright)
Pick n Pay (Playwright; best with aisle URLs)
Shoprite (HTML + hidden JSON)
Makro (Playwright; currently unreliable / WIP)

# 1. Setup
Requirements
Node.js 18+
npm
# Install dependencies
npm install
npm install

# Install Playwright browsers (for Checkers / PnP / Makro)
npx playwright install chromium
npx playwright install chromium

# 3. Project Structure (key files)
scrapers/checkersScraper.js – Checkers scraper function.
scrapers/pnpScraper.js – PnP scraper function (search or aisle URLs).
scrapers/makroScraper.js – Makro scraper function (needs refinement).
scrapers/shopriteScraper.js – Shoprite scraper using axios + cheerio.
scrapers/utils.js – parsePriceToNumber helper.
scrapers/masterScraper.js – runs all scrapers together.
services/browser.js – Playwright-extra + stealth browser launcher.
services/normalizeProducts.js – normalization helpers:
normalizeName
extractSizeToken
normalizeProducts
services/productMatcher.js – fuzzy grouping across stores:
assignProductIds
services/basketEngine.js – basket-level comparison:
compareBasket
testMatch.js – example script for testing cross-store product IDs.

# 4. Unified Data Shape
Every scraper returns an array of:
{
  store: "Checkers" | "Pick n Pay" | "Shoprite" | "Makro",
  name: "Albany Brown Bread 700g",
  price: 18.99 // number
}
{  store: "Checkers" | "Pick n Pay" | "Shoprite" | "Makro",  name: "Albany Brown Bread 700g",  price: 18.99 // number}
normalizeProducts(products) enriches each with:
{
  store,
  name,
  price,
  normalizedName, // cleaned string
  sizeToken,      // e.g. "700g", "2kg", "6x45g"
  canonicalKey    // normalizedName + sizeToken
}
{  store,  name,  price,  normalizedName, // cleaned string  sizeToken,      // e.g. "700g", "2kg", "6x45g"  canonicalKey    // normalizedName + sizeToken}
assignProductIds(products) adds:
{
  store,
  name,
  price,
  product_id: "prod_1" // same ID for same product across stores
}
{  store,  name,  price,  product_id: "prod_1" // same ID for same product across stores}
5. Running Individual Scrapers
From project root:
# Checkers
node scrapers/checkersScraper.js "Albany Brown Bread 700g"

# Shoprite
node scrapers/shopriteScraper.js "milk"

# PnP – search term
node scrapers/pnpScraper.js "bread"

# PnP – aisle URL (example: fresh milk & cream)
node scrapers/pnpScraper.js "https://www.pnp.co.za/c/fresh-milk-and-cream44805192"

# Makro – WIP
node scrapers/makroScraper.js "milk"
# Checkersnode scrapers/checkersScraper.js "Albany Brown Bread 700g"# Shopritenode scrapers/shopriteScraper.js "milk"# PnP – search termnode scrapers/pnpScraper.js "bread"# PnP – aisle URL (example: fresh milk & cream)node scrapers/pnpScraper.js "https://www.pnp.co.za/c/fresh-milk-and-cream44805192"# Makro – WIPnode scrapers/makroScraper.js "milk"
Each prints structured JSON with the unified shape.
5. Running All Scrapers Together
Use the master controller:
node scrapers/masterScraper.js "Albany Brown Bread 700g"
node scrapers/masterScraper.js "Albany Brown Bread 700g"
Output: combined array of all stores for that query:
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
[  {    "store": "Checkers",    "name": "Albany Superior Brown Bread 700g",    "price": 19.99  },  {    "store": "Shoprite",    "name": "Albany Superior Brown Bread 700g",    "price": 17.99  }]
You can also import it in code:
const runAllScrapers = require('./scrapers/masterScraper');

const results = await runAllScrapers('Albany Brown Bread 700g');
const runAllScrapers = require('./scrapers/masterScraper');const results = await runAllScrapers('Albany Brown Bread 700g');
6. Product Normalization & Matching
Normalize products
const { normalizeProducts } = require('./services/normalizeProducts');

const normalized = normalizeProducts(results);
const { normalizeProducts } = require('./services/normalizeProducts');const normalized = normalizeProducts(results);
Assign cross-store product IDs
const { assignProductIds } = require('./services/productMatcher');
const runAllScrapers = require('./scrapers/masterScraper');

(async () => {
  const all = await runAllScrapers('Albany Brown Bread 700g');
  const withIds = assignProductIds(all); // adds product_id
  console.log(withIds);
})();
const { assignProductIds } = require('./services/productMatcher');const runAllScrapers = require('./scrapers/masterScraper');(async () => {  const all = await runAllScrapers('Albany Brown Bread 700g');  const withIds = assignProductIds(all); // adds product_id  console.log(withIds);})();
Example object:
{
  store: 'Checkers',
  name: 'Albany Superior Brown Bread 700g',
  price: 19.99,
  product_id: 'prod_1'
}
{  store: 'Checkers',  name: 'Albany Superior Brown Bread 700g',  price: 19.99,  product_id: 'prod_1'}
Same product from PnP/Shoprite/Makro will share product_id: 'prod_1' (based on normalized name + size + fuzzy similarity).
7. Basket Engine (Cart-Level Comparison)
Use compareBasket to see cheapest store per item in a user cart:
# space-separated
node services/basketEngine.js "milk" "bread" "eggs"

# or comma-separated
node services/basketEngine.js "milk,bread,eggs"
# space-separatednode services/basketEngine.js "milk" "bread" "eggs"# or comma-separatednode services/basketEngine.js "milk,bread,eggs"
Example result shape:
[
  {
    "item": "milk",
    "normalizedItem": "milk",
    "matches": [
      { "store": "Checkers", "name": "Clover Full Cream Milk Fresh 2L", "price": 35.99 },
      { "store": "Shoprite", "name": "Ritebrand Full Cream Milk 1L", "price": 16.99 }
    ],
    "cheapest": {
      "store": "Shoprite",
      "name": "Ritebrand Full Cream Milk 1L",
      "price": 16.99
    }
  }
]
[  {    "item": "milk",    "normalizedItem": "milk",    "matches": [      { "store": "Checkers", "name": "Clover Full Cream Milk Fresh 2L", "price": 35.99 },      { "store": "Shoprite", "name": "Ritebrand Full Cream Milk 1L", "price": 16.99 }    ],    "cheapest": {      "store": "Shoprite",      "name": "Ritebrand Full Cream Milk 1L",      "price": 16.99    }  }]
You can also call it from code:
const { compareBasket } = require('./services/basketEngine');

const basket = ['milk', 'bread', 'eggs'];
const comparison = await compareBasket(basket);
const { compareBasket } = require('./services/basketEngine');const basket = ['milk', 'bread', 'eggs'];const comparison = await compareBasket(basket);
8. Current Limitations / TODO
PnP
Free-text search ("Albany Brown Bread 700g") often returns no results due to complex front-end structure.
Works better with aisle URLs (e.g. milk).
TODO: inspect/lock down robust selectors for search result product tiles.
Makro
Strong anti-bot and dynamic loading; current Playwright heuristic is fragile.
TODO: switch to an HTML/JSON-based approach similar to Shoprite if possible.
Matching
assignProductIds uses string-similarity with a default threshold of 0.9.
May need tuning for edge cases and multi-pack / promo naming.
9. Git & GitHub Workflow
Basic steps to track changes and push:
# 1. Init repo (first time only)
git init
git add .
git commit -m "Initial scrapers and basket engine"

# 2. Add remote (replace with your repo URL)
git remote add origin git@github.com:<your-username>/grocery-price-scraper.git

# 3. Push
git push -u origin main   # or master, depending on your default branch
# 1. Init repo (first time only)git initgit add .git commit -m "Initial scrapers and basket engine"# 2. Add remote (replace with your repo URL)git remote add origin git@github.com:<your-username>/grocery-price-scraper.git# 3. Pushgit push -u origin main   # or master, depending on your default branch
When you make changes:
git status
git add <files>
git commit -m "Describe your change"
git push
git status
