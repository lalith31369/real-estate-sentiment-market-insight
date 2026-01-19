**Task B - Design Normalization Logic and ROI Features**

Project: Real Estate Sentiment & Market Insight

---

### 1. Objective

This task defines the normalization and standardization logic for raw real estate data. It includes rules for handling missing values, standardizing addresses, normalizing currency, and computing ROI-related features (e.g., rental yield, profit margin) to support analytics and scoring.

---

### 2. Inputs and Target Datasets

Normalization rules apply to:

- Property data (address, bedrooms, bathrooms, square feet, etc.)
- Market data (median price, average price per sqft, market location)
- Foreclosure and auction data (status, auction date, final price)
- Comparable sales (sale price, sale date, sqft)

---

### 3. Missing Value Handling Rules

**3.1 Required Identifiers (Must Not Be Missing)**

- property_id, market_id, foreclosure_id/auction_id (where applicable)Rule: If missing → reject record or send to quarantine for review.

**3.2 Property Attributes**

- bedrooms, bathrooms, square_feetRules:
- If square_feet is missing → attempt to infer from median sqft of comparable sales in the same market; if still missing → mark as null and flag.
- If bedrooms/bathrooms missing → impute using median by property type/market (if available); otherwise leave null and flag.

**3.3 Prices**

- auction final_price, sale_price, market median_priceRules:
- If price is missing → do not compute ROI; keep record but mark ROI fields as null and flag.

**3.4 Dates**

- auction_date, sale_date, filing_dateRules:
- If date missing → keep record but exclude from time-window analytics; flag for review.

**3.5 Duplicate Records**

Rules:

- Identify duplicates using stable keys (e.g., property_id + event_date + event_type).
- Keep the most recently updated record, or the record with the most complete fields.

---

### 4. Address Standardization Rules

Goal: Convert inconsistent address text into a uniform format for linking and grouping.

**4.1 Standard Format**

Store as:

- address_line_1, city, state, zip_codeExample: "123 W Main St, Cincinnati, OH 45220"

**4.2 Cleaning Rules**

- Trim spaces and remove double spaces
- Convert to Title Case (or consistent casing)
- Standardize abbreviations:
    - Street → St, Avenue → Ave, Road → Rd, Apartment → Apt
- Standardize directionals:
    - North → N, South → S, East → E, West → W
- Remove punctuation except where required (e.g., keep hyphen in ZIP+4)

**4.3 Validation Rules**

- state must be a 2-letter code
- zip_code must be 5 digits (or 9 digits for ZIP+4)
- If city/state/zip mismatch → flag

---

### 5. Currency Normalization

Assumption: Most U.S. data is in USD. If currency is provided and varies:

- Convert all monetary fields to USD using the exchange rate for the transaction date (or monthly average if daily rate unavailable).
- Store two fields:
    - original_currency, original_amount
    - amount_usd (standardized)

If currency is unknown:

- Assume USD but flag for validation.

---

### 6. ROI-Related Feature Engineering

These features are calculated after normalization to support scoring and market insight.

**6.1 Annual Rent Estimate**

- annual_rent = estimated_monthly_rent × 12If estimated_monthly_rent missing:
- Use market-level median rent estimate (if available); otherwise leave null and flag.

**6.2 Gross Rental Yield (Rental Yield %)**

- gross_rental_yield = (annual_rent / purchase_price) × 100Where purchase_price can be:
- auction final_price (preferred for distressed properties)If purchase_price missing → yield = null and flag

**6.3 Estimated Profit (Absolute)**

- estimated_profit = estimated_market_value − purchase_price − estimated_costs

**6.4 Profit Margin (%)**

- profit_margin = (estimated_profit / purchase_price) × 100If estimated_market_value missing:
- Use market median price or comparable sales median price (same market, recent window)

**6.5 Discount to Market (%)**

- discount_to_market = ((estimated_market_value − purchase_price) / estimated_market_value) × 100

**6.6 ROI Risk Flags (Rule-Based)**

- If square_feet is null OR purchase_price is null → roi_quality_flag = "insufficient_data"
- If purchase_price is extremely low/high compared to market (e.g., > 3σ deviation) → roi_quality_flag = "outlier_price"
- If address validation fails → roi_quality_flag = "address_mismatch"

---

### 7. Example Calculation

Given:

- purchase_price (auction final_price) = $120,000
- estimated_monthly_rent = $1,800
- estimated_market_value = $165,000
- estimated_costs = $10,000

Computed:

- annual_rent = 1,800 × 12 = $21,600
- gross_rental_yield = (21,600 / 120,000) × 100 = 18.0%
- estimated_profit = 165,000 − 120,000 − 10,000 = $35,000
- profit_margin = (35,000 / 120,000) × 100 = 29.17%
- discount_to_market = ((165,000 − 120,000) / 165,000) × 100 = 27.27%

---

### 8. Outputs Produced

The normalization process produces:

- Cleaned, standardized fields (address components, consistent numeric types, USD prices)
- Quality flags (missing key fields, outliers, invalid addresses)
- ROI features (rental yield, profit margin, discount to market)

---

### 9. Assumptions and Notes

- The schema and logic are designed using standard real estate domain concepts and representative datasets.
- ROI features are estimates that depend on the availability and quality of rent and market value data.
- Records with missing key identifiers are quarantined to protect data integrity.
