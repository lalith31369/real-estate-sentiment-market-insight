# **Task A - Define Data Schemas for Property/Market Linkage**
---

## **Project Name: Real Estate Sentiment & Market Insight**
---

## **1. Objective**

Define a detailed, normalized relational data schema for real estate data. The schema links property-level data with market-level data, foreclosure cases, auction events, and comparable sales. The design ensures clear connections between entities to support analytics and downstream processing.

---

## **2. Scope**

The schema design covers the following entities:

- Property data
- Market data
- Foreclosure case data
- Auction event data
- Comparable sales data

Each entity is modeled as an independent table with defined relationships using primary and foreign keys.

---

## **3. Schema Definitions**

### **3.1 Property Table**

| Column Name | Data Type | Description |
| --- | --- | --- |
| property_id | UUID (PK) | Unique identifier for each property |
| address | VARCHAR | Property street address |
| city | VARCHAR | City where the property is located |
| state | VARCHAR | State |
| zip_code | VARCHAR | ZIP code |
| bedrooms | INTEGER | Number of bedrooms |
| bathrooms | FLOAT | Number of bathrooms |
| square_feet | INTEGER | Total living area |
| market_id | UUID (FK) | Links the property to a market |

---

### **3.2 Market Table**

| Column Name | Data Type | Description |
| --- | --- | --- |
| market_id | UUID (PK) | Unique market identifier |
| city | VARCHAR | Market city |
| state | VARCHAR | Market state |
| median_price | FLOAT | Median home price |
| avg_price_per_sqft | FLOAT | Average price per square foot |

---

### **3.3 Foreclosure Table**

| Column Name | Data Type | Description |
| --- | --- | --- |
| foreclosure_id | UUID (PK) | Foreclosure case identifier |
| property_id | UUID (FK) | Associated property |
| filing_date | DATE | Date of foreclosure filing |
| status | VARCHAR | Foreclosure status |

---

### **3.4 Auction Table**

| Column Name | Data Type | Description |
| --- | --- | --- |
| auction_id | UUID (PK) | Auction identifier |
| foreclosure_id | UUID (FK) | Associated foreclosure case |
| auction_date | DATE | Date of auction |
| final_price | FLOAT | Final auction price |

---

### **3.5 Comparable Sales Table**

| Column Name | Data Type | Description |
| --- | --- | --- |
| sale_id | UUID (PK) | Sale record identifier |
| market_id | UUID (FK) | Associated market |
| sale_price | FLOAT | Sale price |
| sale_date | DATE | Date of sale |
| square_feet | INTEGER | Property size |

---

## **4. Relationships**

- One **Market** links to multiple **Properties**
- One **Property** can have multiple **Foreclosure Cases**
- Each **Foreclosure Case** may link to one **Auction**
- One **Market** can have multiple **Comparable Sales**

Foreign keys enforce all relationships to maintain referential integrity.

---

## **5. Constraints and Data Integrity**

- Primary keys uniquely identify each record
- Foreign keys enforce valid relationships between tables
- Critical linkage fields are non-null
- Data types ensure consistency and scalability

---

## **6. Conclusion**

This schema provides a structured, normalized foundation for linking property-level data with market-level insights. It supports downstream ingestion, normalization, valuation modeling, and end-to-end validation within the real estate analytics pipeline.

The schema is designed based on standard real estate domain concepts and representative datasets typically used in property and market analytics.

---
