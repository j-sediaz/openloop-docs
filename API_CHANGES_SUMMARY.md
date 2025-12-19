# OpenLoop Partners API v1.1 - Update Summary

## 🚀 What's New

We've enhanced the Partners API with robust clinical workflow management, lab integration, and document verification capabilities. Here's everything you need to know about the updates.

---

## 📋 **NEW: Patient Document Management**

Identity verification and document management for patients.

### Endpoints

```bash
# Upload patient document
POST /patients/{id}/documents

# List all patient documents  
GET /patients/{id}/documents

# Get specific document
GET /patients/{id}/documents/{document_id}

# Delete document
DELETE /patients/{id}/documents/{document_id}

# Verify document
PUT /patients/{id}/documents/{document_id}/verify
```

### Document Types Supported
- `drivers_license` - Driver's license
- `passport` - Passport
- `state_id` - State ID card
- `insurance_card` - Insurance card
- `other` - Other identification

### Example: Upload Document

```json
POST /patients/pat_123/documents
{
  "type": "drivers_license",
  "document_number": "D1234567",
  "issuing_state": "CA",
  "issuing_country": "US",
  "expiration_date": "2028-05-15",
  "file_url": "https://secure.openloop.com/upload/doc.pdf",
  "file_type": "application/pdf"
}
```

### Document Statuses
- `pending` - Uploaded, awaiting verification
- `verified` - Verified by staff
- `rejected` - Rejected (invalid/unclear)
- `expired` - Document expired

---

## 🧪 **NEW: Lab Order Management**

Full lab ordering and results tracking integrated with patient orders.

### Endpoints

```bash
# Create standalone lab order
POST /lab-orders

# List all lab orders (filter by patient, order, status)
GET /lab-orders?patient_id=pat_123&status=completed

# Get specific lab order
GET /lab-orders/{id}

# Update lab order status
PUT /lab-orders/{id}

# Get labs for specific order (nested route)
GET /orders/{order_id}/labs

# Submit lab results for order (nested route)
POST /orders/{order_id}/labs/{lab_order_id}/results
```

### Lab Order Lifecycle

```
ordered → received → completed → reviewed
                           ↓
                      cancelled (optional)
```

### Example: Create Lab Order

```json
POST /lab-orders
{
  "patient_id": "pat_xyz789",
  "order_id": "ord_def456",
  "lab_id": "lab_total_testosterone",
  "lab_provider": "Quest Diagnostics",
  "notes": "Fasting required"
}
```

### Example: Submit Lab Results

```json
POST /orders/ord_123/labs/labord_456/results
{
  "value": "350",
  "unit": "ng/dL",
  "reference_range": {
    "min": 300,
    "max": 1000
  },
  "flag": "normal",
  "tested_at": "2025-01-14T10:00:00Z",
  "file_url": "https://secure.openloop.com/labs/result.pdf",
  "notes": "Results within normal range"
}
```

### Common Lab IDs
- `lab_total_testosterone` - Total Testosterone
- `lab_free_testosterone` - Free Testosterone
- `lab_hematocrit` - Hematocrit
- `lab_psa` - Prostate-Specific Antigen
- `lab_lipid_panel` - Lipid Panel
- `lab_a1c` - Hemoglobin A1C

### Result Flags
- `normal` - Within reference range
- `low` - Below reference range
- `high` - Above reference range
- `critical` - Requires immediate attention

---

## 🛍️ **ENHANCED: Order Management**

Orders now include clinical workflows, lab integration, provider review, and fulfillment tracking.

### New Order Fields

#### **Clinical Status**
Automated risk assessment and eligibility checking:
```json
"clinical_status": {
  "eligibility_check": "passed",     // passed, failed, pending
  "risk_level": "low",                // low, medium, high
  "requires_sync_consult": false,
  "approval_status": "pending_provider_review"
}
```

#### **Labs Integration**
Track lab orders directly within orders:
```json
"labs": [
  {
    "id": "labord_001",
    "lab_id": "lab_total_testosterone",
    "lab_name": "Total Testosterone",
    "status": "ordered",
    "value": "350",
    "unit": "ng/dL",
    "in_range": true
  }
]
```

#### **Provider Review**
Track provider assignment and review process:
```json
"review": {
  "assigned_provider_id": "prov_1234",
  "assigned_at": "2025-01-11T14:20:00Z",
  "reviewed_at": "2025-01-11T16:00:00Z",
  "decision": "approved",
  "provider_notes": "Patient meets criteria for treatment",
  "patient_facing_notes": "Your order has been approved",
  "requires_sync_consult": false,
  "consult_scheduled_at": null
}
```

#### **Fulfillment Tracking**
Track pharmacy fulfillment and shipping:
```json
"fulfillment": {
  "status": "shipped",
  "pharmacy_id": "pharm_001",
  "tracking_number": "1Z999AA10123456784",
  "carrier": "UPS",
  "shipped_at": "2025-01-15T14:00:00Z",
  "estimated_delivery": "2025-01-18T17:00:00Z"
}
```

#### **Pricing Breakdown**
Detailed pricing information:
```json
"pricing": {
  "subtotal": 19900,
  "shipping": 0,
  "tax": 0,
  "discount": 0,
  "total": 19900,
  "currency": "USD"
}
```

#### **Subscription Management**
Track subscription details:
```json
"subscription": {
  "is_subscription": true,
  "subscription_id": "sub_stripe_xyz",
  "next_billing_date": "2025-02-11T14:15:00Z",
  "auto_renew": true
}
```

### New Order Statuses

**Complete order lifecycle:**
```
pending_payment → paid → pending_review → approved → fulfilling → fulfilled → active
                    ↓                         ↓
              failed/cancelled            rejected/on_hold
```

**Status Definitions:**
- `pending_payment` - Awaiting payment
- `paid` - Payment successful, awaiting clinical review
- `pending_review` - Awaiting provider review
- `approved` - Provider approved, ready for fulfillment
- `rejected` - Provider rejected order
- `fulfilling` - Being fulfilled by pharmacy
- `fulfilled` - Shipped/delivered
- `active` - Active subscription/treatment
- `cancelled` - Order cancelled
- `on_hold` - Temporarily paused

### Payment Statuses
Separate from order status for clarity:
- `pending` - Payment not yet processed
- `paid` - Successfully paid
- `failed` - Payment failed
- `refunded` - Fully refunded
- `partially_refunded` - Partially refunded

### New Order Endpoints

```bash
# Update fulfillment tracking
PUT /orders/{id}/fulfillment
{
  "status": "shipped",
  "tracking_number": "1Z999AA10123456784",
  "carrier": "UPS",
  "shipped_at": "2025-01-15T14:00:00Z"
}

# Get order labs
GET /orders/{id}/labs

# Submit lab results to order
POST /orders/{id}/labs/{lab_order_id}/results
```

### Enhanced Order Creation

```json
POST /orders
{
  "patient_id": "pat_xyz789",
  "program_id": "prog_trt_001",
  "product_id": "prod_trt_monthly_0100",
  "questionnaire_response_id": "qr_ghi789",
  "payment_method_id": "pm_stripe_abc123",
  "metadata": {
    "campaign": "new_year_promo_2025",
    "utm_source": "partner_referral",
    "partner_order_id": "external_ref_12345"
  }
}
```

---

## 🔄 **Complete Order Workflow**

### Standard Order Flow

1. **Patient Completes Questionnaire**
   - System evaluates risk level automatically
   - Determines if sync consultation required

2. **Create Order**
   ```bash
   POST /orders
   ```
   - Status: `pending_payment`

3. **Process Payment**
   - Payment succeeds → `payment_status: paid`
   - Order status → `paid`

4. **Clinical Review Initiated**
   - Status → `pending_review`
   - System orders required labs (if applicable)
   - Provider assigned automatically or manually

5. **Labs Ordered (if required)**
   ```bash
   POST /lab-orders
   ```
   - Labs show in order `labs[]` array
   - Patient receives lab requisition

6. **Lab Results Submitted**
   ```bash
   POST /orders/{id}/labs/{lab_id}/results
   ```
   - Results reviewed by provider

7. **Provider Reviews Order**
   - Reviews questionnaire, labs, medical history
   - Makes decision: `approved` or `rejected`
   - Status → `approved` or `rejected`

8. **Fulfillment Begins**
   - Status → `fulfilling`
   - Sent to pharmacy
   - Prescription filled

9. **Order Shipped**
   ```bash
   PUT /orders/{id}/fulfillment
   ```
   - Status → `fulfilled`
   - Tracking number provided
   - Patient notified

10. **Treatment Active**
    - Status → `active`
    - Subscription billing begins (if applicable)
    - Renewal scheduled based on program rules

---

## 📊 **Key API Improvements**

### Nested Routes Pattern
All new endpoints follow RESTful nested route conventions:
- `/patients/{id}/documents` - Patient resources
- `/orders/{id}/labs` - Order resources
- `/orders/{id}/fulfillment` - Order resources

### Enhanced Data Models
- **Orders**: Added `clinical_status`, `labs[]`, `review`, `fulfillment`, `pricing`, `subscription`
- **Patients**: Can now have associated `documents[]`
- **Lab Orders**: Comprehensive lab tracking with results, reference ranges, and flags

### Better Status Tracking
- Separate `status` and `payment_status` for orders
- Detailed fulfillment status tracking
- Clinical approval workflow status

### Metadata Support
All resources now support flexible `metadata` fields for:
- Campaign tracking (UTM parameters)
- Partner IDs and external references
- Payment provider IDs (Stripe, Chargebee)
- Custom business logic

---

## 🎯 **Integration Examples**

### Complete Patient Onboarding Flow

```bash
# 1. Create patient
POST /patients
{ "email": "patient@example.com", ... }

# 2. Upload ID document
POST /patients/pat_123/documents
{ "type": "drivers_license", ... }

# 3. Verify document
PUT /patients/pat_123/documents/doc_456/verify
{ "verified": true }

# 4. Create order with questionnaire
POST /orders
{
  "patient_id": "pat_123",
  "product_id": "prod_trt_001",
  "questionnaire_response_id": "qr_789",
  "payment_method_id": "pm_stripe_abc"
}

# 5. System automatically creates lab orders
# Labs appear in GET /orders/ord_123

# 6. Submit lab results when available
POST /orders/ord_123/labs/labord_456/results
{ "value": "350", "unit": "ng/dL", ... }

# 7. Provider reviews and approves
# System updates order status to "approved"

# 8. Update fulfillment when shipped
PUT /orders/ord_123/fulfillment
{
  "status": "shipped",
  "tracking_number": "1Z999AA...",
  "carrier": "UPS"
}
```

---

## 🔐 **Authentication**

No changes to authentication. Continue using:

**API Key (Recommended):**
```bash
-H "x-api-key: YOUR_API_KEY"
```

**Bearer Token:**
```bash
-H "Authorization: Bearer YOUR_TOKEN"
```

---

## 🌐 **Environment**

**Staging:** `https://api-staging.openloophealth.com`

---

## 📝 **Breaking Changes**

### None! ✅

All changes are **additive**. Existing integrations will continue to work. New fields are optional and provide additional functionality.

### Deprecation Notices

The following fields are **deprecated but still functional**:
- `Order.total_amount` - Use `Order.pricing.total` instead
- `Order.currency` - Use `Order.pricing.currency` instead

These fields will be removed in v2.0 (no earlier than Q4 2025).

---

## 📚 **Resources**

- **API Documentation:** https://docs-staging.openloophealth.com
- **OpenAPI Spec:** `/api-reference/openapi.json`
- **Support:** support@openloophealth.com

---

## 📤 **NEW: File Uploads**

A dedicated endpoint for uploading files before creating document records.

### Endpoint

```bash
POST /uploads
```

**Content-Type:** `multipart/form-data`

### How It Works

**Two-step process:**
1. Upload file to `/uploads` → Get secure `file_url`
2. Create document/record with that `file_url`

### Example

```bash
# Step 1: Upload file
curl -X POST https://api-staging.openloophealth.com/uploads \
  -H "x-api-key: YOUR_API_KEY" \
  -F "file=@drivers_license.pdf" \
  -F "type=document"

# Response:
{
  "data": {
    "file_url": "https://secure.openloop.com/files/abc123.pdf",
    "file_id": "file_abc123",
    "file_type": "application/pdf",
    "file_size": 245678
  }
}

# Step 2: Create document with file_url
curl -X POST https://api-staging.openloophealth.com/patients/pat_123/documents \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "drivers_license",
    "file_url": "https://secure.openloop.com/files/abc123.pdf",
    ...
  }'
```

**File Requirements:**
- Max size: 10MB
- Formats: PDF, JPG, JPEG, PNG
- Types: `document`, `lab_result`, `prescription`, `other`

---

## 📝 **NEW: Questionnaire Responses**

Submit and track patient questionnaire responses with automatic risk assessment.

### Endpoints

```bash
# Submit questionnaire response
POST /questionnaire-responses

# List questionnaire responses
GET /questionnaire-responses?patient_id=pat_123

# Get specific response
GET /questionnaire-responses/{id}
```

### How It Works

1. Get questionnaire schema from `/programs/{id}/questionnaires`
2. Collect patient answers
3. Submit via `POST /questionnaire-responses`
4. System automatically evaluates risk level
5. Use response ID when creating orders

### Example: Submit Response

```json
POST /questionnaire-responses
{
  "patient_id": "pat_xyz789",
  "questionnaire_id": "q_trt_intake",
  "program_id": "prog_trt_001",
  "answers": {
    "main_symptom": "low_energy",
    "duration_months": 12,
    "history_of_prostate_cancer": false
  }
}
```

### Response with Risk Assessment

```json
{
  "data": {
    "questionnaire_response": {
      "id": "qr_abc123",
      "answers": {...},
      "risk_assessment": {
        "risk_level": "low",
        "requires_sync_consult": false,
        "next_step": "allow_async_order",
        "flags": []
      }
    }
  }
}
```

**Key Fields:**
- `risk_level` - `low`, `medium`, or `high`
- `requires_sync_consult` - If `true`, patient needs video consultation before ordering
- `next_step` - Recommended action
- `flags` - Clinical concerns raised by responses

### Use with Orders

```json
POST /orders
{
  "patient_id": "pat_xyz789",
  "product_id": "prod_trt_001",
  "questionnaire_response_id": "qr_abc123"  // ← Required
}
```

---

## 🎉 **Summary**

### New Endpoints: 19
- 1 File upload endpoint
- 3 Questionnaire response endpoints
- 4 Document endpoints
- 6 Lab order endpoints  
- 5 Order enhancement endpoints

### New Capabilities
✅ Patient identity verification with documents
✅ Complete lab order lifecycle management
✅ Clinical risk assessment and provider review workflows
✅ Fulfillment and shipping tracking
✅ Subscription management
✅ Detailed pricing breakdowns

### What This Enables
- **Compliance**: Verify patient identity with documents
- **Clinical Quality**: Track labs and provider reviews
- **Transparency**: Real-time order and fulfillment status
- **Analytics**: Campaign tracking and metadata support
- **Automation**: Automated risk assessment and routing

---

**Questions?** Reach out to our partnership team or check the full API documentation.

