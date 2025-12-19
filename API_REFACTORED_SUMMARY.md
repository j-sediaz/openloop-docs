# API Refactored to Flat Structure ✅

## 🎯 **What Changed**

We simplified the API from **nested routes** to a **flat, RESTful structure** based on industry best practices (Stripe, Twilio).

---

## 📊 **Before vs After**

### **Documents**

#### ❌ Before (Nested - 5 endpoints)
```bash
POST   /patients/{id}/documents
GET    /patients/{id}/documents
GET    /patients/{id}/documents/{doc_id}
DELETE /patients/{id}/documents/{doc_id}
PUT    /patients/{id}/documents/{doc_id}/verify  # Action-based!
```

#### ✅ After (Flat - 4 endpoints + 1 convenience)
```bash
POST   /documents                     # Body: { patient_id, type, file_url, ... }
GET    /documents?patient_id=pat_123  # Filter by patient
GET    /documents/{id}
PATCH  /documents/{id}                # Update including { verified: true }
DELETE /documents/{id}

# Convenience endpoint (shortcut)
GET    /patients/{id}/documents       # Same as /documents?patient_id={id}
```

**Benefits:**
- ✅ Shorter URLs
- ✅ More flexible filtering
- ✅ No "action" endpoints (/verify)
- ✅ Standard PATCH for updates

---

### **Lab Orders**

#### ❌ Before (Nested - 6 endpoints)
```bash
POST /lab-orders
GET  /lab-orders
GET  /lab-orders/{id}
PUT  /lab-orders/{id}

POST /orders/{id}/labs/{lab_id}/results  # Action-based!
GET  /orders/{id}/labs
```

#### ✅ After (Flat - 4 endpoints + 1 convenience)
```bash
POST  /lab-orders                     # Body: { patient_id, order_id, lab_id }
GET   /lab-orders?order_id=ord_123   # Filter by order
GET   /lab-orders/{id}
PATCH /lab-orders/{id}                # Update including results data

# Convenience endpoint (shortcut)
GET   /orders/{id}/labs               # Same as /lab-orders?order_id={id}
```

**Benefits:**
- ✅ Submit results via PATCH (not separate POST)
- ✅ Can filter by any field
- ✅ One endpoint for all updates

---

### **Orders**

#### ❌ Before
```bash
POST /orders
GET  /orders
GET  /orders/{id}
PUT  /orders/{id}
DELETE /orders/{id}

PUT /orders/{id}/fulfillment  # Separate endpoint for fulfillment!
```

#### ✅ After
```bash
POST   /orders
GET    /orders
GET    /orders/{id}
PATCH  /orders/{id}           # Update everything (status, fulfillment, review)
DELETE /orders/{id}
```

**Benefits:**
- ✅ One update endpoint for everything
- ✅ PATCH supports partial updates
- ✅ Cleaner structure

---

### **HTTP Methods**

#### ❌ Before
- Used `PUT` for updates (requires full object)
- Had separate endpoints for sub-actions

#### ✅ After
- Use `PATCH` for updates (partial updates supported)
- Everything in one endpoint per resource

---

## 🎯 **New API Structure**

### **Complete Endpoint List (Simplified)**

```bash
# SYSTEM
GET    /health

# PATIENTS  
POST   /patients
GET    /patients
GET    /patients/{id}
PATCH  /patients/{id}
DELETE /patients/{id}

# DOCUMENTS (flat + 1 convenience)
POST   /documents
GET    /documents?patient_id=X&type=Y&status=Z
GET    /documents/{id}
PATCH  /documents/{id}
DELETE /documents/{id}
GET    /patients/{id}/documents        # Convenience

# UPLOADS
POST   /uploads

# QUESTIONNAIRES
POST   /questionnaire-responses
GET    /questionnaire-responses?patient_id=X&questionnaire_id=Y
GET    /questionnaire-responses/{id}

# PROGRAMS
GET    /programs
GET    /programs/{id}
GET    /programs/{id}/questionnaires
GET    /programs/{id}/products

# PRODUCTS
GET    /products

# ORDERS
POST   /orders
GET    /orders
GET    /orders/{id}
PATCH  /orders/{id}
DELETE /orders/{id}

# LAB ORDERS (flat + 1 convenience)
POST   /lab-orders
GET    /lab-orders?order_id=X&patient_id=Y&status=Z
GET    /lab-orders/{id}
PATCH  /lab-orders/{id}
GET    /orders/{id}/labs                # Convenience

# FUNNEL RESPONSES
POST   /funnel-responses
```

**Total: 29 endpoints** (vs 35 before)

---

## 📝 **Usage Examples**

### **Upload and Verify Document**

#### Before (2 endpoints)
```bash
# 1. Upload file
POST /uploads
→ file_url

# 2. Create document
POST /patients/pat_123/documents
{ "type": "drivers_license", "file_url": "...", ... }
→ doc_abc

# 3. Verify (separate endpoint)
PUT /patients/pat_123/documents/doc_abc/verify
{ "verified": true }
```

#### After (cleaner)
```bash
# 1. Upload file
POST /uploads
→ file_url

# 2. Create document
POST /documents
{ "patient_id": "pat_123", "type": "drivers_license", "file_url": "...", ... }
→ doc_abc

# 3. Verify (same update endpoint)
PATCH /documents/doc_abc
{ "verified": true, "verified_by": "admin_01", "status": "verified" }
```

---

### **Submit Lab Results**

#### Before
```bash
POST /orders/ord_123/labs/labord_456/results
{ "value": "350", "unit": "ng/dL", ... }
```

#### After (simpler)
```bash
PATCH /lab-orders/labord_456
{
  "status": "completed",
  "value": "350",
  "unit": "ng/dL",
  "flag": "normal",
  "results_received_at": "2025-01-15T10:00:00Z"
}
```

---

### **Update Order Fulfillment**

#### Before
```bash
PUT /orders/ord_123/fulfillment
{
  "status": "shipped",
  "tracking_number": "1Z999AA...",
  "carrier": "UPS"
}
```

#### After (same update endpoint)
```bash
PATCH /orders/ord_123
{
  "fulfillment": {
    "status": "shipped",
    "tracking_number": "1Z999AA...",
    "carrier": "UPS",
    "shipped_at": "2025-01-15T14:00:00Z"
  }
}
```

---

### **Get Patient's Documents**

#### Both work (flexibility)
```bash
# Option 1: Direct filter
GET /documents?patient_id=pat_123

# Option 2: Convenience shortcut
GET /patients/pat_123/documents

# Both return the same result
```

---

## ✅ **Benefits Summary**

### **Simpler**
- 29 endpoints (vs 35 before)
- Shorter URLs
- One update endpoint per resource

### **More RESTful**
- No action-based endpoints (/verify, /results)
- Use proper HTTP methods (PATCH for partial updates)
- Resources, not actions

### **More Flexible**
- Filter by any field with query params
- Partial updates with PATCH
- Can combine multiple updates in one call

### **Industry Standard**
- Follows Stripe/Twilio patterns
- Easier for developers to understand
- Better API design practices

---

## 🚀 **Migration Notes**

### **Breaking Changes**

1. **Document endpoints moved:**
   - `POST /patients/{id}/documents` → `POST /documents` (with `patient_id` in body)
   - `/documents/{id}/verify` removed → use `PATCH /documents/{id}`

2. **Lab results submission:**
   - `POST /orders/{id}/labs/{lab_id}/results` removed → use `PATCH /lab-orders/{id}`

3. **Order fulfillment:**
   - `PUT /orders/{id}/fulfillment` removed → use `PATCH /orders/{id}`

4. **HTTP method changes:**
   - `PUT` → `PATCH` for updates (patients, orders, lab-orders, documents)

### **Convenience Endpoints (Non-Breaking)**

These still work for backwards compatibility:
- `GET /patients/{id}/documents` → shortcut for `/documents?patient_id={id}`
- `GET /orders/{id}/labs` → shortcut for `/lab-orders?order_id={id}`

---

## 📚 **Updated Documentation**

All workflow guides updated:
- ✅ Upload documents flow
- ✅ Submit questionnaires flow
- ✅ Lab order lifecycle
- ✅ Order management

Check `DEVELOPER_WORKFLOWS.md` for complete examples.

---

**Result:** Cleaner, more RESTful, industry-standard API ✨

