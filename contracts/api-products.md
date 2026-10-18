# API Contract: Products, Cost Versions, Price History & Pricing Calculator

**Version:** 1.0.0  
**Status:** Approved  
**Author:** backend-ai / leader  
**Applies to:** Milestone M2  

---

## 1. Global Conventions

- **Base URL**: `/api/v1/businesses/{business_id}`
- **Authentication**: `Authorization: Bearer <access_token>`
- **Authorization**: Caller must be an active member of `{business_id}` (`RequireTenant` middleware).
- **Format**: JSON `snake_case`, RFC 3339 timestamps.
- **Decimal Precision**:
  - Currency values are represented as string decimals with 2 decimal places (`"15000.00"`).
  - Stock values are represented as string decimals with 3 decimal places (`"10.000"`).
- **Standard Error Envelope**:
  ```json
  {
    "error": {
      "code": "ERROR_CODE",
      "message": "Pesan ramah pengguna dalam Bahasa Indonesia",
      "details": null
    }
  }
  ```

---

## 2. Endpoints Specification

### 2.1 List Products
`GET /api/v1/businesses/{business_id}/products`

**Query Parameters:**
- `search` (optional string): Case-insensitive search on `name` or `sku`.
- `category` (optional string): Filter by exact category.
- `status` (optional string): Filter by status (`active` | `archived`). Default: `active`.
- `page` (optional integer): Default `1`.
- `page_size` (optional integer): Default `20`, maximum `100`.

**Response `200 OK`:**
```json
{
  "items": [
    {
      "id": "018f23a0-1234-7000-8000-000000000001",
      "business_id": "018f23a0-0000-7000-8000-000000000000",
      "sku": "KOP-001",
      "name": "Kopi Susu Gula Aren",
      "category": "Minuman",
      "unit": "Cup",
      "current_selling_price": "18000.00",
      "current_cost_price": "8500.00",
      "min_stock": "10.000",
      "current_stock": "45.000",
      "status": "active",
      "created_at": "2026-10-18T08:00:00Z",
      "updated_at": "2026-10-18T08:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_items": 1,
    "total_pages": 1
  }
}
```

---

### 2.2 Create Product
`POST /api/v1/businesses/{business_id}/products`

**Request Body:**
```json
{
  "sku": "KOP-001",
  "name": "Kopi Susu Gula Aren",
  "category": "Minuman",
  "unit": "Cup",
  "current_selling_price": "18000.00",
  "current_cost_price": "8500.00",
  "min_stock": "10.000",
  "current_stock": "50.000"
}
```

**Behavior**:
- Atomically inserts product record.
- Inserts initial record in `cost_versions` (with `reason: "initial_setup"` and `effective_from: now()`).
- Inserts initial record in `price_history` (with `reason: "initial_setup"` and `effective_from: now()`).
- Writes audit log `product.created`.

**Response `201 Created`:**
Returns the created product object.

**Errors:**
- `400 BAD_REQUEST`: Validation failure.
- `409 CONFLICT` (`SKU_ALREADY_EXISTS`): SKU is already registered for this business.

---

### 2.3 Get Product Detail
`GET /api/v1/businesses/{business_id}/products/{product_id}`

**Response `200 OK`:**
Returns the product object with current prices and stock.

**Errors:**
- `404 PRODUCT_NOT_FOUND`: Product does not exist or belongs to another business.

---

### 2.4 Update Product
`PATCH /api/v1/businesses/{business_id}/products/{product_id}`

**Request Body (all fields optional):**
```json
{
  "name": "Kopi Susu Gula Aren Spesial",
  "category": "Minuman",
  "unit": "Cup",
  "min_stock": "15.000",
  "current_stock": "40.000",
  "status": "active",
  "new_selling_price": "20000.00",
  "new_cost_price": "9000.00",
  "change_reason": "Kenaikan harga susu sapi murni"
}
```

**Behavior**:
- If `new_cost_price` differs from `current_cost_price`: appends new row in `cost_versions`.
- If `new_selling_price` differs from `current_selling_price`: appends new row in `price_history`.
- Updates `products` row.
- Writes audit log `product.updated`.

**Response `200 OK`:**
Returns the updated product object.

---

### 2.5 Get Cost Versions History
`GET /api/v1/businesses/{business_id}/products/{product_id}/cost-versions`

**Response `200 OK`:**
```json
{
  "items": [
    {
      "id": "018f23a0-2222-7000-8000-000000000002",
      "product_id": "018f23a0-1234-7000-8000-000000000001",
      "cost_price": "9000.00",
      "reason": "Kenaikan harga susu sapi murni",
      "effective_from": "2026-10-18T10:00:00Z",
      "created_at": "2026-10-18T10:00:00Z"
    },
    {
      "id": "018f23a0-1111-7000-8000-000000000001",
      "product_id": "018f23a0-1234-7000-8000-000000000001",
      "cost_price": "8500.00",
      "reason": "initial_setup",
      "effective_from": "2026-10-18T08:00:00Z",
      "created_at": "2026-10-18T08:00:00Z"
    }
  ]
}
```

---

### 2.6 Get Price History
`GET /api/v1/businesses/{business_id}/products/{product_id}/price-history`

**Response `200 OK`:**
```json
{
  "items": [
    {
      "id": "018f23a0-3333-7000-8000-000000000002",
      "product_id": "018f23a0-1234-7000-8000-000000000001",
      "selling_price": "20000.00",
      "reason": "Penyesuaian margin",
      "effective_from": "2026-10-18T10:00:00Z",
      "created_at": "2026-10-18T10:00:00Z"
    }
  ]
}
```

---

### 2.7 Pricing Calculator Simulation
`POST /api/v1/businesses/{business_id}/pricing-calculator`

**Request Body:**
```json
{
  "cost_price": "10000.00",
  "method": "margin",
  "target_percentage": 25.0
}
```
*Note: `method` can be `"margin"` or `"markup"`.*

**Calculation Formulas (TRD TD-12):**
- When `method = "margin"`:
  $$\text{Recommended Price} = \frac{\text{Cost}}{1 - (\text{Target} / 100)}$$
- When `method = "markup"`:
  $$\text{Recommended Price} = \text{Cost} \times (1 + (\text{Target} / 100))$$
- $\text{Profit} = \text{Price} - \text{Cost}$
- $\text{Margin \%} = \frac{\text{Profit}}{\text{Price}} \times 100$
- $\text{Markup \%} = \frac{\text{Profit}}{\text{Cost}} \times 100$

**Response `200 OK`:**
```json
{
  "recommended_selling_price": "13333.33",
  "profit_per_unit": "3333.33",
  "margin_percentage": 25.0,
  "markup_percentage": 33.33,
  "warnings": []
}
```

**Warnings Generation:**
- If `margin_percentage < 15.0`: adds warning `"MARGIN_TOO_LOW"` (*"Margin di bawah 15% berisiko tergerus biaya operasional tak terduga."*).
- If `recommended_selling_price <= cost_price`: adds warning `"SELLING_BELOW_COST"` (*"Harga jual tidak boleh lebih rendah atau sama dengan HPP."*).
