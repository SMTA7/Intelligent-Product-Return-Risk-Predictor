# Intelligent-Product-Return-Risk-Predictor


## Overview
An automated n8n workflow that analyzes incoming orders and calculates risk scores based on multiple factors to identify potentially fraudulent or high-risk transactions.

## Node-by-Node Explanation

### 1. **Webhook** (Entry Point)
- **Purpose**: Receives POST requests to `/order-risk-check`
- **Function**: Triggers the workflow when order data is submitted
- **Output**: Raw order data from request body

### 2. **Edit Fields** (Data Extraction)
- **Purpose**: Extracts and structures order data from webhook payload
- **Function**: Maps JSON fields to workflow variables
- **Output**: Structured order data (order_id, customer_email, product_category, etc.)

### 3. **Edit Fields1** (Risk Score Initialization)
- **Purpose**: Initializes risk assessment variables
- **Function**: Sets risk_score = 0 and risk_factors = ""
- **Output**: Base risk assessment object

### 4. **If** (COD Payment Check)
- **Purpose**: Checks if payment method is Cash on Delivery
- **Function**: Evaluates if payment_mode == "COD"
- **Output**: Routes to risk increment or pass-through

### 5. **Edit Fields2** (COD Risk Increment)
- **Purpose**: Adds risk for COD payments
- **Function**: Increments risk_score by 1, adds "COD Payment" to factors
- **Output**: Updated risk data with COD penalty

### 6. **If1** (Low Order Value Check)
- **Purpose**: Checks if order value is below threshold
- **Function**: Evaluates if order_value < 5000
- **Output**: Routes to risk increment or pass-through

### 7. **Edit Fields3** (Low Value Risk Increment)
- **Purpose**: Adds risk for low-value orders
- **Function**: Increments risk_score by 1, adds "Low Order Value" to factors
- **Output**: Updated risk data with low-value penalty

### 8. **If2** (Product Category Check)
- **Purpose**: Checks for high-risk product categories
- **Function**: Evaluates if product_category == "Apparel" OR "Electronics"
- **Output**: Routes to risk increment or pass-through

### 9. **Edit Fields4** (Category Risk Increment)
- **Purpose**: Adds risk for high-risk product categories
- **Function**: Increments risk_score by 1, adds "High-risk product category" to factors
- **Output**: Updated risk data with category penalty

### 10. **If4** (First-Time Buyer Check)
- **Purpose**: Checks if customer is first-time buyer
- **Function**: Evaluates if customer_order_count == 1
- **Output**: Routes to risk increment or pass-through

### 11. **Edit Fields6** (First-Time Buyer Risk Increment)
- **Purpose**: Adds risk for first-time buyers
- **Function**: Increments risk_score by 1, adds "First-Time-buyer" to factors
- **Output**: Updated risk data with first-time buyer penalty

### 12. **Google Sheet** (Risky Regions Data)
- **Purpose**: Fetches list of risky regions from Google Sheets
- **Function**: Reads "RiskyRegions" sheet data
- **Output**: Array of risky region names

### 13. **Code** (Region Data Processing)
- **Purpose**: Processes risky regions data and merges with order data
- **Function**: Combines order data with risky_regions array
- **Output**: Merged data object with risky regions list

### 14. **If3** (Risky Region Check)
- **Purpose**: Checks if customer region is in risky regions list
- **Function**: Evaluates if order region exists in risky_regions array
- **Output**: Routes to risk increment or pass-through

### 15. **Edit Fields5** (Region Risk Increment)
- **Purpose**: Adds risk for risky regions
- **Function**: Increments risk_score by 1, adds "Risky Region" to factors
- **Output**: Updated risk data with region penalty

### 16. **Edit Fields8, 10, 11, 12, 13** (Pass-Through Nodes)
- **Purpose**: Maintain risk_score for conditions that don't trigger
- **Function**: Pass existing risk_score unchanged
- **Output**: Unchanged risk data for merge operation

### 17. **Merge** (Data Consolidation)
- **Purpose**: Combines all risk assessment branches
- **Function**: Merges up to 10 different risk evaluation paths
- **Output**: All risk assessment results combined

### 18. **Code1** (Final Risk Calculation)
- **Purpose**: Calculates final risk score and consolidates factors
- **Function**: Sums all risk_scores and combines risk_factors
- **Output**: Single object with total risk_score and combined risk_factors

### 19. **If5** (Risk Threshold Decision)
- **Purpose**: Determines if order is high-risk
- **Function**: Evaluates if risk_score > 3
- **Output**: Routes to high-risk actions or clean order logging

### 20. **Edit Fields7** (High-Risk Data Preparation)
- **Purpose**: Prepares complete order data for high-risk logging
- **Function**: Combines original order data with risk assessment results
- **Output**: Complete order record with risk information

### 21. **Append row in sheet** (High-Risk Logging)
- **Purpose**: Logs high-risk orders to Google Sheets
- **Function**: Appends order data to "High Risk Log" sheet
- **Output**: Confirmation of successful logging

### 22. **Send a message** (Email Alert)
- **Purpose**: Sends email notification for high-risk orders
- **Function**: Sends formatted email to risk management team
- **Output**: Email delivery confirmation

### 23. **Append row in sheet1** (Clean Order Logging)
- **Purpose**: Logs low-risk orders to Google Sheets
- **Function**: Appends order data to "Clean Orders" sheet
- **Output**: Confirmation of successful logging

## Postman Testing





### Request Setup
```
Method: POST
URL: https://risk-predictor.app.n8n.cloud/webhook-test/order-risk-check
Headers: Content-Type: application/json
```

### Sample Request Body
```json
{
  "order_id": "ORD-12345",
  "customer_email": "customer@example.com",
  "product_category": "Electronics",
  "order_value": 3500,
  "order_date": "2024-01-15",
  "payment_mode": "COD",
  "customer_order_count": 1,
  "region": "Mumbai"
}
```

### Expected Response
The workflow will process the order and return success/failure status based on the risk assessment and logging operations.

## Risk Score Calculation
- Each risk factor adds 1 point
- Maximum possible score: 5 points
- Threshold for high-risk classification: >3 points

## Setup Requirements

### Google Sheets
1. **RiskyRegions Sheet** - Contains list of high-risk regions -- https://docs.google.com/spreadsheets/d/1KxvATkTodI65Qm_IJEH_rFUfIWID1HMx5fOoXppWBNs/edit?usp=sharing
2. **High Risk Log** - Stores flagged orders  -- https://docs.google.com/spreadsheets/d/1CUKeXkBY3ZZriaNbTnHE42OCODc9MlgagRnA2QbVb9Y/edit?usp=sharing
3. **Clean Orders** - Stores approved orders  -- https://docs.google.com/spreadsheets/d/1NpQtVzShBtOzFfksjVrwkfmlnyZSt3BGdJNpHP1nRaI/edit?usp=sharing

### Credentials Needed
- Google Sheets OAuth2 API access
- Gmail OAuth2 for email notifications



