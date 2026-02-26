Practice 01 — Message Routing and Data Transformation

Course: Enterprise Application Integration

Goal

Develop an integration pipeline that:

Receives orders from three distinct source systems

Routes them using a content-driven routing mechanism

Converts them into a unified canonical order structure

Enhances order items via a live Pricing API

Sends the enriched canonical order to downstream systems

EAI Patterns Applied:

Message Translator

Content-Based Router

Content Enricher

Pipes & Filters

Canonical Data Model

Architecture Overview

Integration Flow:

[HTTP Input]
      ↓
[Content-Based Router]
      ↓
[Source-Specific Translator]
      ↓
[Product ID Standardization]
      ↓
[Pricing API Request]
      ↓
[Attach Price Details]
      ↓
[Content Filter]
      ↓
[HTTP Response]
Source Systems

1. Web Store

Format: Nested JSON

Example:

{
  "orderId": "WEB-123",
  "customerEmail": "janis@ex.com",
  "items": [
    { "productCode": "PROD-001", "quantity": 1, "priceEur": 899 }
  ],
  "orderDate": "2026-02-17T10:30Z",
  "shippingMethod": "express"
}

2. Mobile App

Format: Flat JSON

3. B2B Partner

Format: XML

Requires: XML → JSON conversion, product ID normalization, grouping multiple rows into a single order

Canonical Order Structure

All orders are transformed into this standard JSON model:

{
  "schemaVersion": "1.0",
  "canonicalOrderId": "string",
  "sourceSystem": "WEB | MOBILE | B2B",
  "receivedAt": "ISO-8601",
  "customer": {
    "name": "string",
    "email": "string",
    "phone": "string",
    "company": "string",
    "externalReferences": []
  },
  "items": [
    { "productId": "PROD-XXX", "quantity": 0, "unitPrice": 0.0, "currency": "ISO-4217", "taxRate": 0.0 }
  ],
  "shipping": { "method": "express | standard | bulk" },
  "timestamps": { "orderCreated": "ISO-8601" }
}

Design Principles:

Single superset model for all sources

ISO-8601 timestamps

ISO-4217 currency codes

Optional fields allowed

Self-describing (includes sourceSystem + schemaVersion)

Field Mapping

Web → Canonical

Web Field	Canonical Field
orderId	canonicalOrderId
customerEmail	customer.email
productCode	items.productId
quantity	items.quantity
priceEur	items.unitPrice
orderDate	timestamps.orderCreated
shippingMethod	shipping.method

Mobile → Canonical

Mobile Field	Canonical Field
oid	canonicalOrderId
email	customer.email
sku	items.productId
ts	timestamps.orderCreated

B2B XML → Canonical

XML Field	Canonical Field
ref	canonicalOrderId
cust_name	customer.name
cust_phone	customer.phone
sku	items.productId
qty	items.quantity
unit_price	items.unitPrice
currency	items.currency
order_ts	timestamps.orderCreated
ship	shipping.method

All dates converted to ISO-8601

Product IDs standardized (see below)

Product ID Normalization

Required before enriching with Pricing API:

Source	Raw ID	Standardized ID
Web	PROD-001	PROD-001
Mobile	001	PROD-001
B2B	SKU-PROD-001	PROD-001

Normalization Rules:

Add PROD- prefix if missing

Remove SKU- prefix if present

Content Enrichment — Pricing API

Each order item is enriched via:

GET http://pricing-api:3000/pricing/:id

Appended fields:

unitPrice

currency

taxRate

Prices are dynamically fetched, never hardcoded.

Content Filtering

Before sending downstream, remove sensitive fields:

customer.phone

customer.email (for external systems)

Allowlist-based filtering preferred for external integrations.

Schema Evolution

Example: Web adds giftWrap field

Solution: add as optional in canonical model

Do not make mandatory

Other systems ignore unknown fields

Ensures backward compatibility.

Deployment Instructions

Start Docker in the project directory:

docker compose up --build

Open Node-RED:

http://localhost:1880

Test the endpoint:

POST http://localhost:1880/order
Example body:

{
  "sourceSystem": "WEB",
  "orderId": "WEB-123",
  "customerEmail": "test@test.com",
  "items": [{ "productCode": "001", "quantity": 1, "priceEur": 100 }],
  "orderDate": "2026-02-17T10:30Z",
  "shippingMethod": "express"
}
Benefits of Using a Canonical Model

Without it: N × (N−1) translators needed

With it: only 2 × N translators

Supports scaling as more systems are added

Summary

This design demonstrates:

Proper application of EAI transformation patterns

Modular, maintainable flow using Pipes & Filters

Canonical Data Model implementation

Real-time content enrichment

Awareness of schema evolution and backward compatibility

The integration process is fully operational and meets lab requirements.
