# Amazon Wishlist Price Tracker

An intelligent n8n workflow that automatically monitors your Amazon wishlist for price drops and sends beautiful, AI-analyzed email alerts with personalized buying recommendations.

## Features

- **AI-Powered Analysis**: Google Gemini AI provides detailed buying recommendations for each price drop
- **Price History Tracking**: Maintains complete price history with trend visualization
- **Beautiful Email Reports**: Responsive HTML emails with charts, savings calculations, and AI insights
- **Smart Filtering**: Only notifies for significant price drops (>5% discount)
- **Automated Scheduling**: Runs checks every 6 hours automatically
- **Google Sheets Integration**: Stores all data in a structured spreadsheet
- **Web Scraping**: Uses ScrapingBee to reliably extract Amazon product data

## Workflow Architecture

```
Schedule Trigger (Every 6h)
        |
        v
HTTP Request (ScrapingBee)
        |
        v
HTML Parser (Extract Data)
        |
        v
Code: Clean Product Data
        |
        v
Google Sheets: Get Existing Data
        |
        v
    [IF: New Product?]
        |
        +----> YES: Code: Format New Products
        |              |
        |              v
        |      Google Sheets: Append New Products
        |
        +----> NO: Code: Compare Prices
                       |
                       v
                 [IF: Price Drop > 5%?]
                       |
                       +----> YES: Google Sheets: Update Price Drop
                       |              |
                       |              v
                       |      Code: Prepare AI Input
                       |              |
                       |              v
                       |      AI Agent (Gemini): Analyze Deals
                       |              |
                       |              v
                       |      Code: Generate Email Data
                       |              |
                       |              v
                       |      Code: Format HTML Email
                       |              |
                       |              v
                       |      Gmail: Send Alert
                       |
                       +----> NO: Google Sheets: Update Check Time
```

### Node Flow

1. **Schedule Trigger** → Runs every 6 hours
2. **HTTP Request** → Scrapes Amazon wishlist via ScrapingBee
3. **HTML Parser** → Extracts product titles, links, and prices
4. **Code (JavaScript)** → Cleans and structures product data
5. **Get Rows (Google Sheets)** → Fetches existing price history
6. **If (Check Existing)** → Determines if products are new or existing
7. **Code (New Products)** → Formats new products for initial tracking
8. **Append/Update Sheet** → Saves new products to Google Sheets
9. **Code (Price Comparison)** → Compares current vs previous prices
10. **If (Price Drop Filter)** → Filters for drops >5%
11. **Append/Update Sheet** → Updates sheet with price drops
12. **Prepare AI Input** → Formats data for AI analysis
13. **AI Agent (Gemini)** → Analyzes deals and generates recommendations
14. **Generate Chat Data** → Structures AI response with product data
15. **Format Email** → Creates beautiful HTML email
16. **Send Message (Gmail)** → Delivers price drop alert

## Prerequisites

### Required Services

- **n8n** (self-hosted or cloud)
- **ScrapingBee Account** (for web scraping)
- **Google Account** (for Sheets and Gmail)
- **Google Gemini API** (for AI analysis)

### Required Credentials

1. **ScrapingBee API Key**
2. **Google Sheets OAuth2**
3. **Gmail OAuth2**
4. **Google Gemini (PaLM) API**

## Setup Instructions

### 1. Clone and Import

Import `workflow.json` into your n8n instance.

### 2. Configure Amazon Wishlist

Update the URL in the **HTTP Request** node with your Amazon wishlist URL and ScrapingBee API key.

### 3. Setup Google Sheets

1. Create a new Google Sheet with these column headers:
   ```
   Product_Name | Product_Link | ASIN | Original_Price | Current_Price | 
   Previous_Price | Price_Drop_Percent | Check_Date | Check_Time | 
   Price_History | Notes
   ```

2. Update the `documentId` in all Google Sheets nodes with your Google Sheet ID.

### 4. Configure Email Recipient

Update the email address in the **format email** node with your desired recipient address.

### 5. Add Credentials

Configure the following credentials in n8n:

- **ScrapingBee API**: Add your API key
- **Google Sheets OAuth2**: Authorize your Google account
- **Gmail OAuth2**: Authorize your Gmail account
- **Google Gemini API**: Add your API key

### 6. Test the Workflow

1. Click "Execute workflow" to run a manual test
2. Check the Google Sheet for populated data
3. Verify you receive the email alert (may need to trigger price drops first)

## Google Sheets Structure

| Column | Description |
|--------|-------------|
| Product_Name | Full product title from Amazon |
| Product_Link | Direct link to Amazon product page |
| ASIN | Amazon Standard Identification Number |
| Original_Price | First recorded price |
| Current_Price | Latest price from current check |
| Previous_Price | Price from previous check |
| Price_Drop_Percent | Percentage decrease from previous price |
| Check_Date | Date of price check (YYYY-MM-DD) |
| Check_Time | Time of price check (HH:MM:SS) |
| Price_History | Comma-separated list of all prices |
| Notes | Status messages (e.g., "Initial load", "Price dropped") |

## AI Analysis Features

The Google Gemini AI provides:

- **Deal Quality Assessment**: BUY NOW, GOOD DEAL, or WAIT verdicts
- **Value Analysis**: Compares discount against historical data and market trends
- **Product Evaluation**: Analyzes features and specifications for the price point
- **Timing Recommendations**: Advises on urgency and optimal purchase timing
- **Actionable Insights**: Clear buy/wait/skip recommendations

## Email Report Features

### Summary Section
- Total savings across all price drops
- Best discount percentage
- Highlighted best deal

### Per-Product Cards Include
- Deal quality badge (AMAZING / GREAT / GOOD)
- Price comparison with visual indicators
- Discount percentage and savings amount
- Price drop visualization (bar chart)
- Price history sparkline with trend
- AI analysis with 4-point recommendation
- Direct "View on Amazon" button
- Product metadata (ASIN, check time)

### Email Example

```
Price Drop Alert!
AI-Powered Amazon Wishlist Tracker

Total Savings: ₹1,100
Best Discount: 10.0%

Best Deal: Redmi 13 5G Prime Edition... (10.0% OFF)

Product 1: GREAT DEAL
₹10,999 → ₹9,899 (10.0% OFF)
You Save: ₹1,100

AI Analysis: GOOD DEAL
- Decent 10% discount makes this new smartphone launch very competitive
- Strong features including 108MP camera, 5G, and large 6.7in display
- Solid debut price for a new phone, but deeper festive sales might follow
- Recommendation: Consider if you need a budget 5G phone with good camera now
```

## Customization Options

### Adjust Price Drop Threshold

Modify the **If1** node condition:

```javascript
// Current: >5% drops
{ leftValue: "={{ $json.Price_Drop_Percent }}", rightValue: 5, operator: "gt" }

// Example: >10% drops only
{ leftValue: "={{ $json.Price_Drop_Percent }}", rightValue: 10, operator: "gt" }
```

### Change Schedule Frequency

Update the **Schedule Trigger** node interval:
- Current: Every 6 hours
- Options: Hourly, daily, custom cron expression

### Modify AI Prompt

Edit the **prepare AI input** node to customize AI analysis style:

```javascript
// Current: Detailed 4-point analysis
// Customize: Change verdict thresholds, point structure, or analysis focus
```

## Troubleshooting

### No Products Found

- Verify Amazon wishlist URL is correct and public
- Check ScrapingBee API quota and validity
- Ensure wishlist has products with valid prices

### Price Drops Not Detected

- Run workflow manually twice (first run populates initial prices)
- Check if products actually had price changes
- Verify Google Sheets has data in Current_Price column

### Email Not Received

- Check Gmail OAuth2 authorization
- Verify email address in "format email" node
- Check spam/promotions folder
- Ensure price drops exceed 5% threshold

### AI Analysis Failed

- Verify Google Gemini API key is valid
- Check API quota limits
- Review AI Agent node error logs

## Performance Metrics

- **Average Execution Time**: 30-60 seconds
- **Products Tracked**: Unlimited (based on your wishlist)
- **API Calls per Run**: 
  - 1x ScrapingBee request
  - 2-3x Google Sheets operations
  - 1x Gemini AI request (only when price drops detected)
  - 1x Gmail send (only when price drops detected)

## Privacy & Security

- Your Amazon wishlist must be set to **public** for scraping to work
- All credentials are stored securely in n8n
- Price data is stored in your private Google Sheet
- Emails are sent only to your configured address
- No third-party data sharing

## Potential Enhancements

- Support for multiple wishlists
- Telegram/Discord notifications
- Price prediction using historical trends
- Multi-region Amazon support
- SMS alerts for critical deals
- Dashboard visualization with charts
- Competitor price comparison

## Acknowledgments

- **n8n** - Workflow automation platform
- **ScrapingBee** - Web scraping service
- **Google Gemini** - AI analysis
- **Amazon** - Product data source
