# Japan Used Car Export Scraper

A web scraper for collecting used car listings from SBT Japan, one of Japan's largest used car exporters. Built and debugged interactively, aided by Claude AI (Anthropic).

## About

This project scrapes used car listing data from SBT Japan and saves it to a CSV file for analysis, research, or inventory purposes. The scraper handles pagination across 2,870+ pages and supports overnight batch runs with auto-resume on failure.

## Data Collected

Each car record includes:
| Field | Description |
| --- | --- |
| ``` stock_id ``` | Unique SBT identifier |
| ``` title ``` | Full car name (year, make, model, grade) |
| ``` year ``` | Registration |
| ``` price_usd ```|Vehicle price in USD
| ``` total_price ```|Total price including shipping estimate|
|``` mileage```|Odometer reading (km)|
|```engine_cc```|Engine displacement (cc)|
|```transmission```|AT / MT / CVT etc|
|```fuel_type```|Petrol / Diesel / Hybrid|
|```drive_type```|2WD / 4WD|
|```steering```|RHD / LHD|
|```color```|Exterior color|
|```location```|Warehouse location in Japan|
|```url```|Direct link to listing|

## Requirements

Python 3.13+
Playwright
Pandas
nest_asyncio

### Install dependencies:

pip install playwrite pandas nest_asincio
playwrite intall chromium

## Usage
### In Jupyter Notebook

Run the notebook cells sequentially. The scraper uses ```subprocess``` to run Playwright in a completely separate proess, bypassing Jupiter's asyncio event loop conflict on Windows.

## Small Test Scrape (~500 cars)

```
all_cars = scrape(
    "https://www.sbtjapan.com/used-cars/search?show_numbers=20&registration_year_from=2018",
    max_pages=25,
    timeout=600
)
```

## Overnight / Full Scrape (~287,000 cars)

The batch scraper runs all 2870 pages in batches of 25:

- Auto-saves every 25 pages to used_car_export_data_full.csv
- Saves progress to scraper_progress.json
- Skips failed batches and continues automatically
- Resumes from last saved page if interrupted — just re-run the cell

```
BATCH_SIZE  = 25      # pages per batch before saving
TOTAL_PAGES = 2870    # total pages on site (~287,000 cars)
```
# Output Files
| File | Description |
| --- | --- |
|```used_car_export_data.csv```| Sample / test scrape|
|```used_car_export_data_full.csv|Full overnight scrape|
|```scraper_progress.json|Progress tracker for resuming interrupted runs|
|```README.md|This file|


### To find your files, run:
```
import os
print(os.getcwd())
```

# Known Limitations
- SBT Japan stocks mostly Japanese domestic brands (Toyota, Honda, Nissan, Suzuki etc.)
- European brands (Mercedes, BMW, Audi) are available on the site but require filtering by make_id
- Some listings display Ask instead of a numeric price
- Cars registered before 2000 may have null year value (older date formats not parsed)
- The site uses Cloudflare bot protection — the scraper uses polite delays (~2.5s per page) to avoid blocks

# Technical Notes

## Windows + Jupyter asyncio Fix

Playwright requires ProactorEventLoop on Windows but Jupyter on Windows defaults to SelectorEventLoop. The running loop cannot be closed or replaced while Jupyter is active. The solution was to run Playwright in a completely separate subprocess

```
result = subprocess.run([sys.executable, "-c", script], capture_output=True, text=True, timeout=700)
```
This bypasses the asyncio conflict entirely and works reliably on:

- Windows 10/11
- Anaconda Python 3.13
- Jupyter Notebook / JupyterLab

# CSS Selectors Used 
Confirmed by inspecting live HTML on SBT Japan

```
Card container : li.search-result__item
Title          : .card-product__product
Vehicle price  : .card-product__vehicle-price .card-product__price
Total price    : .card-product__total-price .card-product__price
Stock ID       : .card-product__stock-value
Mileage        : .card-product__status.-mileage
Engine         : .card-product__status.-engine-capacity
Transmission   : .card-product__status.-transmission
Fuel type      : .card-product__status.-fuel-type
Drive type     : .card-product__status.-drive-type
Steering       : .card-product__status.-steering-type
Color          : .card-product__status.-body-color
Location       : .card-product__location-value
Next page      : a.pagination__link.-next:not(.-is-disabled)
```

# AI Assistance
This project was built interactively with assistance from Claude by Anthropic.

Claude assisted with:

- Diagnosing and resolving the Windows asyncio / Playwright NotImplementedError
- Identifying that SelectorEventLoop cannot be replaced while Jupyter is running
- Devising the subprocess workaround as the correct fix
- Inspecting live HTML across 20+ debug steps to find real CSS selectors
- Discovering the correct SBT Japan listing URL (/used-cars not /used-cars-for-sale)
- Building pagination, batch scraping, and auto-resume logic

# Disclaimer

This scraper is intended for personal research and data analysis only.
Please respect SBT Japan's Terms of Service and do not hammer their servers with rapid requests.

# License
```
MIT License — free to use, modify, and distribute.
```





