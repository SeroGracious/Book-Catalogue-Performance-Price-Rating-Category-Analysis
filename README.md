# Book-Catalogue-Performance-Price-Rating-Category-Analysis
End-to-end data analytics project: a Python web scraper collects 1,000 books, pandas cleans the data, and a Power BI dashboard explores how price, rating and category relate.  Topics: web-scraping, python, beautifulsoup, pandas, power-bi, data-analysis, data-visualization, data-cleaning, capstone-project

<img width="1344" height="752" alt="Screenshot 2026-10-29 060424" src="https://github.com/user-attachments/assets/f54d79dc-ac42-4f6e-a987-616b5ccd26ca" />


Technical Report - Book Catalogue Price, Rating and Category Analysis
Company / Industry: Online Book Retail (practice dataset from books.toscrape.com)
Prepared for: Vephla University
Prepared by: Oriomah Ogheneserome Gracious
Date: October 2026
Introduction
This project collects, cleans and analyses catalogue data for 1,000 books from Books to Scrape 
(books.toscrape.com), a sandbox website built for practicing web scraping, to show how pricing, ratings 
and categories relate in an online bookstore.
The dataset captures each book's title, price, star rating, category, stock level and product description, 
gathered with a Python scraper. Because the site is a practice environment, the figures are illustrative 
and do not represent real retail results.
The goal of the analysis is to answer three questions: whether higher-priced books earn better ratings, 
how price and rating differ across categories, and how the catalogue is spread across price bands and 
stock levels. The findings are presented in a Power BI dashboard.
Methodology
The analysis followed a structured, step-by-step data analytics approach, from collecting the data with a 
Python web scraper to reporting the findings in Power BI.
1. Data Collection (Web Scraping): A Python scraper visited all 50 listing pages of 
books.toscrape.com to collect the title, price, star rating and link of 1,000 books. It then opened 
each book's own page to collect the category, UPC, stock availability, number of reviews and 
description. The raw data was saved to CSV.
2. Data Cleaning and Preparation: Prices and stock levels stored as text were converted to 
numbers, redundant tax and price columns were removed, the review count was dropped 
because every book had zero reviews, and the two missing descriptions were labelled.
3. Data Structuring: Category, star rating and price band were used as grouping fields. Price and 
stock were used as the measures to be summarized.
4. Exploratory Data Analysis (EDA): Descriptive statistics were calculated for price, rating and 
stock. Average price and rating were compared across categories, and the relationship between 
price and rating was tested.
5. Visualization and Reporting: A Power BI dashboard was built with summary cards, category 
comparisons, rating and price-band distributions, and slicers for category and star rating.
6. Insight Development: Observations from each visual were turned into key insights, together 
with notes on the limits of the data.
7. Tools Used: Python (requests, BeautifulSoup, pandas) in a Jupyter Notebook for scraping and 
cleaning, and Microsoft Power BI Desktop for analysis and visualization.
Story of Data
The cleaned dataset holds one row for each of the 1,000 books listed on Books to Scrape, described by 
eight fields. Some fields come from the listing pages and others from each book's own page.
Field What it describes Collected from Example
title Full book title Listing page A Light in the Attic
price Price in pounds (£10.00 to £59.99) Listing page 51.77
rating Star rating from 1 to 5 Listing page 3
url Link to the book's own page Listing page books.toscrape.com/ca
talogue/...
category Book genre, 50 categories in total Book page Poetry
upc Unique product code Book page a897fe39b1053632
stock Copies in stock (1 to 22) Book page 22
description Publisher's product description Book page It's hard to imagine a 
world without...
Title, category, upc, url and description are descriptive fields. Price, rating and stock are the measures 
used in the analysis.
Data Collection (Web Scraping with Python)
A two-stage Python scraper collected all 1,000 books in one run: the first stage gathers the book list, and 
the second opens every book's own page for the remaining details. Books to Scrape was chosen because 
it is built for scraping practice, so there are no legal or blocking concerns, and the scraper still pauses 
between requests and identifies itself.
Step What the code does Why it matters
1. Inspect the site Page structure was checked in the browser to 
find the HTML tags holding each field.
The tags decide what 
the scraper targets, and 
a layout change is fixed 
here.
2. Configuration and 
session
Constants set the base address, the page-URL 
pattern, a 0.3-second delay and a rating-word 
lookup (One to Five). One reusable requests
session sends a User-Agent header.
A single connection is 
faster, and identifying 
the scraper is basic 
good practice.
3. Request helper 
(get_soup)
Fetches a page with a 30-second timeout and 
returns parsed HTML. A 404 returns None, 
other errors are raised, and failures are retried 
up to 3 times with waits of 2, 4 and 6 seconds. 
One dropped 
connection does not 
crash a long run, and 
the 404 tells the loop it 
Step What the code does Why it matters
Text is read as UTF-8. has passed the last 
page. UTF-8 prevents Â£
appearing instead of £.
4. Listing-page parser 
(parse_listing_page)
For each book card it extracts the full title, the 
price text, the star rating (the CSS class word, 
such as Three, converted to 3) and the full link 
to the book's page.
Turns an unstructured 
web page into 
structured rows.
5. Detail-page parser 
(parse_detail_page)
Reads the category from the breadcrumb, 
then the product table (UPC, prices, tax, 
availability, number of reviews) and the 
description paragraph. Missing fields return 
None.
Category, stock and 
description exist only 
on each book's own 
page.
6. Pagination A loop requests pages 1 to 50 and stops when 
a page returns None, giving 20 books per page.
Without it only the first 
20 books would be 
collected.
7. Detail-page loop and 
saving
Each of the 1,000 book links is opened with a 
0.3-second pause, with progress printed every 
100 books. Listing data was saved first as a 
safety copy (books_listing.csv), then 
merged with the detail fields and saved as 
books_raw.csv.
The pause protects the 
server, and the safety 
copy means an 
interruption does not 
lose the first stage.
Observations:
1. Testing on a single page returned 20 books, and requesting page 999 correctly returned None.
2. The full run collected 1,000 books across 50 pages, giving a raw table of 1,000 rows and 12 
columns.
3. One request was dropped by the server ("Connection aborted"), and the retry logic recovered it 
on the first retry with no data lost.
Pre-Insights:
1. Collecting data in two stages and saving after the first one keeps a long scrape safe from 
interruptions.
2. The same pattern of list page, then detail page, can be reused on other sites such as job boards 
or property listings.
Data Cleaning and Preparation
Cleaning turned the raw scrape of 12 mostly text columns into an analysis-ready table of 1,000 books 
and 8 columns with no missing values. A copy of the raw data was cleaned, so the original file in 
data/raw stays untouched.
The first inspection showed 1,000 rows, every column except rating stored as text, 998 descriptions out 
of 1,000, and no duplicate books (checked by UPC).
Issue found Action taken Result
Price stored as text (£51.77) Removed the £ symbol and 
converted to a decimal number
Numeric price from 
£10.00 to £59.99, mean 
£35.07
Stock buried in text (In stock (22 
available))
Extracted the number with a 
regular expression
Integer stock from 1 to 
22, mean 8.59
Review count stored as text Converted to integer, then 
dropped because it was 0 for all 
1,000 books
One uninformative 
column removed
Columns that repeat the price or show 
£0.00 tax (price_excl_tax, 
price_incl_tax, tax) and the original 
availability text
Dropped Fewer redundant 
columns
2 books without a description Filled with "No description" 0 missing values
Possible duplicate books Checked UPC 0 duplicates
The cleaned table has 1,000 rows and 8 columns: title, price, rating, url, category, upc, description and 
stock. It was saved to data/clean/books_clean.csv with UTF-8 encoding so that it opens correctly in 
Power BI and Excel.
Data quality notes:
1. The site's "Default" (152 books) and "Add a comment" (67 books) categories are placeholder 
labels, not real genres.
2. 25 of the 50 categories have fewer than 10 books (89 books in total), so category averages for 
those groups are unreliable.
3. Every book has zero reviews, so review counts could not be analysed. This reflects the sandbox 
nature of the site.
Detailed Analysis
The dashboard covers 1,000 books across 50 categories, with an average price of £35.07 and an average 
rating of 2.92 out of 5.
• Total Books - 1,000
• Average Price - £35.07
• Average Rating - 2.92
• Categories - 50
Books by Star Rating
Observations:
1. One-star is the largest group with 226 books (22.6%), followed by 3 stars (203), 2 stars and 5 
stars (196 each) and 4 stars (179), the smallest group.
2. 422 books (42.2%) are rated 1 or 2 stars, compared with 375 books (37.5%) rated 4 or 5 stars.
3. The average rating of 2.92 sits in the middle of the 5-point scale.
Pre-Insights:
1. Ratings are spread almost evenly, with a slight lean towards low ratings.
2. The share of low-rated books is worth reviewing for quality or description problems.
Average Price by Star Rating
Observations:
1. Average price is £34.56 for 1-star books, £34.81 for 2 stars, £34.69 for 3 stars, £36.09 for 4 stars 
and £35.37 for 5 stars.
2. The gap between the lowest and highest bar is £1.53.
3. The correlation between price and rating is 0.03, which is essentially no relationship (calculated 
in Python on the cleaned data).
Pre-Insights:
1. Price does not predict rating, so a higher price is not a sign of a better-rated book.
2. Pricing decisions should not assume a link with customer ratings.
Books by Price Band
Observations:
1. £20-29 has the most books (206), followed by £40-49 (205), £50-60 (198), £10-19 (196) and £30-
39 (195).
2. The gap between the largest and smallest band is only 11 books.
Pre-Insights:
1. Prices are spread almost evenly from £10.00 to £59.99, so no price range dominates the 
catalogue.
2. Adding sales data would show which price bands actually sell.
Average Price by Category
Observations:
1. Novels has the highest average price at £54.81, followed by Health (£51.45) and Art (£38.52).
2. Erotica (£19.19) and Adult Fiction (£15.36) have the lowest averages shown.
3. These categories are small: Novels, Erotica and Adult Fiction have 1 book each, Health has 4, 
Christian Fiction 6 and Art 8.
Pre-Insights:
1. The extremes in this chart reflect single listings, not category trends.
2. Among categories with 10 or more books, Travel (£39.79) and Fantasy (£39.59) have the highest 
average prices, while Food and Drink (£31.41) and Thriller (£31.43) have the lowest.
Average Rating by Category
Observations:
1. Adult Fiction, Erotica and Novels each have a perfect 5.00 average, followed by Christian Fiction 
(4.17), Health (3.75) and Art (3.63).
2. The three perfect scores come from a single book each.
Pre-Insights:
1. Small categories produce extreme averages, so a perfect score here is not evidence of a strong 
category.
2. Among categories with 10 or more books, Poetry (3.53, 19 books) is the best rated and Science 
Fiction (2.25, 16 books) the lowest.
Post-Analysis Insights
1. Price does not predict rating: average price stays between £34.56 and £36.09 at every star level, 
with a correlation of 0.03.
2. Ratings skew low: 42.2% of books are rated 1 or 2 stars versus 37.5% rated 4 or 5 stars, and the 
average rating is 2.92 out of 5.
3. Prices are evenly spread from £10.00 to £59.99, with 195 to 206 books in each £10 price band.
4. Small categories drive the extremes: Novels (£54.81) and Health (£51.45) are the priciest, and 
three categories reach a perfect 5.00 rating, but each has only 1 to 4 books.
5. Among categories with 10 or more books, Travel (£39.79) and Fantasy (£39.59) are the priciest, 
Poetry is the best rated (3.53) and Science Fiction the lowest (2.25).
6. Total Books: 1,000
7. Average Price: £35.07
8. Average Rating: 2.92
Recommendations
1. Set prices independently of expected ratings, since the data shows no link between the two 
(correlation 0.03).
2. Review the 42.2% of books rated 1 or 2 stars for quality or description problems, starting with 
the lowest-rated categories such as Science Fiction (2.25) and Philosophy (2.36).
3. Compare categories only when each has enough books (10 or more), and treat extremes from 
single-book categories such as Novels, Erotica and Adult Fiction as noise.
4. Relabel the placeholder categories "Default" (152 books) and "Add a comment" (67 books) so 
that category analysis covers every book.
5. Add sales or demand data to find out which price bands actually sell, because the catalogue is 
spread almost evenly across all of them.
6. Re-run the scraper on a schedule to track changes in price, rating and stock over time, and 
extend it to collect review data where a site provides it.

