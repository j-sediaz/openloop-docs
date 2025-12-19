# OpenLoop Partners API v1.1 - Quick Update

## 🚀 What's New

We've shipped **19 new endpoints** for clinical workflow management. Here's the TL;DR:

---

## **1️⃣ File Uploads** 📤

Upload files and get secure URLs (for documents, lab results, etc.)

```bash
POST /uploads
```

**How it works:**
1. Upload file via `POST /uploads` (multipart/form-data)
2. Get back `file_url`
3. Use that URL when creating documents/records

**Example:**
```bash
# Upload file
curl -F "file=@license.pdf" -F "type=document" /uploads
# Returns: { "file_url": "https://secure.../abc123.pdf" }

# Use URL in document
POST /patients/pat_123/documents
{ "file_url": "https://secure.../abc123.pdf", ... }
```

**Limits:** 10MB max, PDF/JPG/PNG only

---

## **2️⃣ Questionnaire Responses** 📝

Submit patient questionnaires with automatic risk assessment

```bash
POST /questionnaire-responses        # Submit
GET  /questionnaire-responses        # List (filter by patient/questionnaire/program)
GET  /questionnaire-responses/{id}   # Get one
```

**How it works:**
1. Get questionnaire from `/programs/{id}/questionnaires`
2. Submit answers via `POST /questionnaire-responses`
3. System returns risk assessment automatically
4. Use response ID when creating orders

**Example:**
```json
POST /questionnaire-responses
{
  "patient_id": "pat_123",
  "questionnaire_id": "q_trt_intake",
  "answers": {
    "main_symptom": "low_energy",
    "history_of_prostate_cancer": false
  }
}

// Response includes risk assessment:
{
  "id": "qr_abc123",
  "risk_assessment": {
    "risk_level": "low",
    "requires_sync_consult": false,  // ← Check this!
    "next_step": "allow_async_order"
  }
}
```

**IMPORTANT:** If `requires_sync_consult: true`, patient needs video consultation before ordering!

---

## **3️⃣ Patient Documents** 📄

Upload and verify patient IDs (driver's license, passport, etc.)

```bash
POST   /patients/{id}/documents              # Upload
GET    /patients/{id}/documents              # List all
GET    /patients/{id}/documents/{doc_id}     # Get one
DELETE /patients/{id}/documents/{doc_id}     # Delete
PUT    /patients/{id}/documents/{doc_id}/verify  # Verify
```

**Types:** `drivers_license`, `passport`, `state_id`, `insurance_card`
**Statuses:** `pending`, `verified`, `rejected`, `expired`

---

## **4️⃣ Lab Orders** 🧪

Complete lab ordering and results tracking

```bash
POST /lab-orders                # Create lab order
GET  /lab-orders               # List (filter by patient/order/status)
GET  /lab-orders/{id}          # Get details
PUT  /lab-orders/{id}          # Update status

# Nested within orders:
GET  /orders/{id}/labs                              # Get order's labs
POST /orders/{id}/labs/{lab_id}/results            # Submit results
```

**Lab Lifecycle:** `ordered → received → completed → reviewed`

**Common Labs:** `lab_total_testosterone`, `lab_hematocrit`, `lab_psa`, `lab_a1c`

---

## **5️⃣ Enhanced Orders** 🛍️

Orders now include clinical workflows and tracking

### New Fields in Orders:

**Clinical Status:**
```json
{
  "eligibility_check": "passed",
  "risk_level": "low",
  "requires_sync_consult": false,
  "approval_status": "pending_provider_review"
}
```

**Labs Array:**
```json
"labs": [
  {
    "id": "labord_001",
    "lab_id": "lab_total_testosterone",
    "status": "completed",
    "value": "350",
    "unit": "ng/dL"
  }
]
```

**Provider Review:**
```json
{
  "assigned_provider_id": "prov_1234",
  "decision": "approved",
  "provider_notes": "Patient meets criteria"
}
```

**Fulfillment Tracking:**
```json
{
  "status": "shipped",
  "tracking_number": "1Z999AA...",
  "carrier": "UPS",
  "shipped_at": "2025-01-15T14:00:00Z"
}
```

### New Order Statuses:
```
pending_payment → paid → pending_review → approved → 
fulfilling → fulfilled → active
```

### New Order Endpoints:
```bash
PUT /orders/{id}/fulfillment    # Update shipping info
```

---

## 📊 **Complete Workflow Example**

```bash
# 1. Create patient
POST /patients

# 2. Upload document file
POST /uploads
# Returns: file_url

# 3. Create document record
POST /patients/pat_123/documents
{ "file_url": "...", ... }

# 4. Get questionnaire
GET /programs/prog_trt_001/questionnaires

# 5. Submit questionnaire
POST /questionnaire-responses
{ "patient_id": "pat_123", "answers": {...} }
# Returns: questionnaire_response_id & risk_assessment

# 6. Check risk assessment, then create order
POST /orders
{ "questionnaire_response_id": "qr_789", ... }
# System auto-creates lab orders if needed

# 7. Submit lab results when available
POST /orders/ord_123/labs/labord_456/results

# 8. Provider reviews & approves
# (system handles this)

# 9. Update fulfillment
PUT /orders/ord_123/fulfillment
```

---

## ✅ **No Breaking Changes**

All changes are **additive**. Your existing integration works as-is.

---

## 📌 **Key Benefits**

✅ **Compliance** - Verify patient identity
✅ **Clinical Quality** - Track labs and reviews  
✅ **Visibility** - Real-time order status
✅ **Automation** - Risk assessment built-in
✅ **Analytics** - Campaign tracking via metadata

---

## 🌐 **Resources**

- **Staging:** `https://api-staging.openloophealth.com`
- **Docs:** https://docs-staging.openloophealth.com
- **Version:** `1.1.0`

---

## 🎯 **What to Do**

1. ✅ Review the new endpoints above
2. ✅ Test in staging environment
3. ✅ Implement document verification (if needed)
4. ✅ Integrate lab workflows (if needed)
5. ✅ Update your order status handling

**Questions?** Reply to this message or contact support@openloophealth.com

