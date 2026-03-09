# 🛒 Kenya Food Price Tracker

> A collaborative data collection project scraping product prices from Kenyan supermarkets. Currently in the **data collection phase** — building the dataset that will power a food price comparison tool for Kenyan consumers.

---

## 📍 Where We Are Right Now

This project is in its **early stage**. We are focused on one thing:

> Collecting clean, structured product and price data from Kenyan supermarkets.

Once we have reliable data from multiple stores, we will build the comparison and tracking features on top of it.

---

## 🏪 Supermarkets Being Scraped

| Supermarket | Status | Maintainer |
|---|---|---|
| Naivas Online | ✅ Complete — 1,156 products collected | Andrew Maina |
| Quickmart | 🔄 In Progress | *(Collaborator)* |

---

## 📦 Product Categories (Naivas)

| Category | Products Collected |
|---|---|
| 🥫 Food Cupboard | 300+ |
| 🥦 Fresh Food | 300+ |
| 📱 Electronics | 256 |
| 🍾 Naivas Liquor | 300+ |
| **Total** | **1,156+ products** |

---

## 🗂️ Project Structure

```
kenya-food-price-tracker/
│
├── README.md                        ← You are here
├── requirements.txt                 ← Python libraries needed
├── .gitignore                       ← Files excluded from Git
│
├── scrapers/                        ← All scraping notebooks
│   ├── naivas_scraper.ipynb         ← ✅ Complete
│   └── quickmart_scraper.ipynb      ← 🔄 In progress
│
├── data/
│   ├── raw/                         ← Data exactly as scraped
│   │   ├── naivas_products.csv      ← ✅ Available
│   │   └── quickmart_products.csv   ← 🔄 Coming soon
│   └── processed/                   ← Combined data (coming soon)
│
├── analysis/                        ← Price analysis (coming soon)
│
└── docs/
    └── naivas_scraper_guide.md      ← Full Naivas scraper documentation
```

---

## ⚙️ Installation and Setup

### Step 1 — Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/kenya-food-price-tracker.git
cd kenya-food-price-tracker
```

### Step 2 — Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 3 — Run the Naivas Scraper
```bash
jupyter notebook scrapers/naivas_scraper.ipynb
```

Run each cell from top to bottom using `Shift + Enter`.

---

## 📊 Standardised Data Format

All scrapers must output CSV files with **exactly these columns** so data from all stores can be combined later:

| Column | Description | Example |
|---|---|---|
| `store` | Supermarket name | Naivas |
| `category` | Product category | Food Cupboard |
| `product_name` | Full product name | Sunrice Basmati Rice 5Kg |
| `current_price` | Current price as a number | 999 |
| `original_price` | Original price as a number | 1825 |
| `discount_amount` | Amount saved as a number | 826 |
| `product_url` | Direct link to product | https://naivas.online/... |
| `date_scraped` | Date scraped in YYYY-MM-DD | 2024-03-08 |

> ⚠️ All scrapers must follow this format exactly so data from all stores can be combined.

---

## ⚖️ Ethical Statement

- ✅ `robots.txt` reviewed before scraping every website
- ✅ Minimum 3 second delay between all requests
- ✅ Only publicly visible data collected
- ✅ No user data or personal information collected
- ✅ For consumer benefit only — not for resale

---

## 👥 Team

| Name | Role |
|---|---|
| Andrew Maina | Lead Developer — Naivas Scraper |
| *(Collaborator)* | Developer — Quickmart Scraper |

---

*Built with ❤️ in Kenya 🇰🇪*
