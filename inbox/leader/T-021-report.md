# Report: T-021 API Contract for Products, Cost Versions & Pricing Calculator

**From:** backend-ai  
**To:** leader  
**Task ID:** T-021  
**Status:** Done  
**Contract File:** `contracts/api-products.md`  

---

## 1. Summary of Changes
- Authored complete HTTP API specification `contracts/api-products.md` covering:
  - `GET /api/v1/businesses/{business_id}/products` (filters, pagination, text search).
  - `POST /api/v1/businesses/{business_id}/products` (atomic creation with initial cost version, price history, and audit log).
  - `GET /api/v1/businesses/{business_id}/products/{product_id}`.
  - `PATCH /api/v1/businesses/{business_id}/products/{product_id}` (append-only cost & price history triggers).
  - `GET /api/v1/businesses/{business_id}/products/{product_id}/cost-versions`.
  - `GET /api/v1/businesses/{business_id}/products/{product_id}/price-history`.
  - `POST /api/v1/businesses/{business_id}/pricing-calculator` (deterministic margin & markup formulas per TRD TD-12 with warning flags).
- Guaranteed tenant isolation with scoped path convention (`/businesses/{business_id}/...`) and decimal string representations for currency (`NUMERIC(15,2)`) and stock (`NUMERIC(12,3)`).
