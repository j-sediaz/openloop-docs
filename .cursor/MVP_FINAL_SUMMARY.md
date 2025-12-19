# OpenLoop Partners API - MVP (Medical Weight Loss)

## 🎯 **MVP Scope: Medical Weight Loss Only**

This API powers the Medical Weight Loss (MWL) program exclusively in MVP.

---

## ✅ **Available Endpoints (18 endpoints)**

### **1. System (1 endpoint)**
```
GET  /health          # API health check
```

---

### **2. Patients (MVP) - 5 endpoints**
Patient management for the MWL program.

```
POST   /patients         # Create new patient
GET    /patients         # List all patients (with filters)
GET    /patients/{id}    # Get patient details
PATCH  /patients/{id}    # Update patient info
DELETE /patients/{id}    # Delete patient
```

**Key Features:**
- Full CRUD operations
- Location management (shipping addresses)
- Demographic data collection
- Metadata support for custom fields

---

### **3. Questionnaires (MVP) - 3 endpoints**
Patient intake questionnaires for MWL eligibility assessment.

```
POST   /questionnaire-responses        # Submit MWL intake form
GET    /questionnaire-responses        # List responses (with filters)
GET    /questionnaire-responses/{id}   # Get specific response
```

**Key Features:**
- Risk assessment calculation
- Sync consultation flagging
- Provider review workflow
- Answers stored as flexible JSON

---

### **4. Products (MVP) - 1 endpoint**
MWL product catalog - 16 GLP-1 products.

```
GET    /products         # List MWL products (filter by program_id=MWL)
```

**Available Products (16 total):**

| Medication | Form | Supply Durations | Codes |
|-----------|------|------------------|-------|
| **Semaglutide** | Oral | 4w, 12w, 24w, 48w | `osg_w04`, `osg_w12`, `osg_w24`, `osg_w48` |
| **Tirzepatide** | Oral | 4w, 12w, 24w, 48w | `otz_w04`, `otz_w12`, `otz_w24`, `otz_w48` |
| **Semaglutide** | Injection | 4w, 12w, 24w, 48w | `sem_w04`, `sem_w12`, `sem_w24`, `sem_w48` |
| **Tirzepatide** | Injection | 4w, 12w, 24w, 48w | `ter_w04`, `ter_w12`, `ter_w24`, `ter_w48` |

All products:
- ✅ Recurring subscriptions
- ✅ Stripe payment integration
- ✅ Personalized care included

---

### **5. Orders (MVP) - 5 endpoints**
Order and subscription management for MWL.

```
POST   /orders           # Create new order/subscription
GET    /orders           # List orders (with filters)
GET    /orders/{id}      # Get order details
PATCH  /orders/{id}      # Update order (status, fulfillment, etc.)
DELETE /orders/{id}      # Cancel order
```

**Order Lifecycle:**
```
pending → paid → pending_review → approved → fulfilling → fulfilled → active
```

**Key Features:**
- Subscription billing management
- Fulfillment tracking (carrier, tracking number)
- Payment method handling
- Status updates

---

### **6. Uploads - 1 endpoint**
File upload service for future document management.

```
POST   /uploads          # Upload file to CDN
```

**Response:**
```json
{
  "file_url": "https://cdn.openloop.com/files/abc123.pdf",
  "file_id": "file_abc123",
  "mime_type": "application/pdf",
  "size_bytes": 102400
}
```

---

### **7. Programs - 2 endpoints**
Program information (currently MWL only).

```
GET    /programs         # List all programs (returns: [MWL])
GET    /programs/{id}    # Get program details (MWL)
```

---

## 🔑 **Authentication**

Two methods supported:

### **API Key (Recommended)**
```bash
curl -H "x-api-key: your_api_key_here" \
  https://api-staging.openloophealth.com/patients
```

### **Bearer Token**
```bash
curl -H "Authorization: Bearer your_token_here" \
  https://api-staging.openloophealth.com/patients
```

---

## 📋 **Complete MWL Patient Flow**

### **Step 1: Create Patient**
```bash
POST /patients
{
  "email": "patient@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "phone": "+1-555-0123",
  "birth_date": "1985-06-15",
  "gender": "male",
  "sex": "male",
  "locations": [{
    "address_line_1": "123 Main St",
    "city": "Los Angeles",
    "state": "CA",
    "zip_code": "90001",
    "default": true,
    "type": "home"
  }]
}

# Response: { "data": { "patient": { "id": "pat_123" } } }
```

---

### **Step 2: Submit MWL Intake Questionnaire**
```bash
POST /questionnaire-responses
{
  "patient_id": "pat_123",
  "questionnaire_id": "q_mwl_intake",
  "program_id": "MWL",
  "answers": {
    "current_weight": "220",
    "target_weight": "180",
    "height": "5'10\"",
    "bmi": "31.5",
    "medical_history": "Type 2 diabetes",
    "current_medications": ["Metformin"],
    "allergies": "None",
    "previous_weight_loss_attempts": "Diet and exercise"
  }
}

# Response: 
# { 
#   "data": { 
#     "questionnaire_response": { 
#       "id": "qr_456",
#       "risk_assessment": {
#         "risk_level": "low",
#         "requires_sync_consult": false
#       }
#     } 
#   } 
# }
```

---

### **Step 3: Browse MWL Products**
```bash
GET /products?program_id=MWL

# Returns: 16 MWL products
# - 4 Semaglutide Oral (4w, 12w, 24w, 48w)
# - 4 Tirzepatide Oral (4w, 12w, 24w, 48w)
# - 4 Semaglutide Injection (4w, 12w, 24w, 48w)
# - 4 Tirzepatide Injection (4w, 12w, 24w, 48w)
```

---

### **Step 4: Create Order**
```bash
POST /orders
{
  "patient_id": "pat_123",
  "program_id": "MWL",
  "product_id": "sem_w12",  # 12-week Semaglutide Injection
  "questionnaire_response_id": "qr_456"
}

# Response: { "data": { "order": { "id": "ord_789", "status": "pending" } } }
```

---

### **Step 5: Track Order**
```bash
GET /orders/ord_789

# Returns full order details including:
# - Current status
# - Fulfillment info (when shipped)
# - Subscription details
# - Payment info
```

---

### **Step 6: Update Fulfillment (When Shipped)**
```bash
PATCH /orders/ord_789
{
  "status": "fulfilled",
  "fulfillment": {
    "status": "shipped",
    "tracking_number": "1Z999AA10123456784",
    "carrier": "UPS",
    "shipped_at": "2025-12-20T14:00:00Z",
    "estimated_delivery": "2025-12-23T17:00:00Z"
  }
}
```

---

## 🚀 **Quick Start**

### **1. Get API Key**
Contact OpenLoop to receive your API key.

### **2. Test Connection**
```bash
curl -H "x-api-key: your_key" \
  https://api-staging.openloophealth.com/health
```

### **3. Check Available Programs**
```bash
curl -H "x-api-key: your_key" \
  https://api-staging.openloophealth.com/programs
```

### **4. List MWL Products**
```bash
curl -H "x-api-key: your_key" \
  https://api-staging.openloophealth.com/products?program_id=MWL
```

### **5. Start Enrolling Patients**
Follow the patient flow above.

---

## 📊 **API Statistics**

- **Total Endpoints:** 18
- **MVP Programs:** 1 (Medical Weight Loss)
- **Products:** 16 (4 medications × 4 durations)
- **Authentication:** API Key + Bearer Token
- **Base URL (Staging):** `https://api-staging.openloophealth.com`
- **OpenAPI Version:** 3.1.0

---

## 🔮 **Coming After MVP**

The following features are planned for post-MVP releases:

1. **Documents** - Patient identity verification (drivers license, passport, etc.)
2. **Lab Orders** - Laboratory test ordering and results
3. **Funnel Responses** - All-in-one enrollment endpoint
4. **Additional Programs** - TRT, HRT, Microdosing, Longevity

See `_FUTURE_ENDPOINTS.md` for detailed specifications.

---

## 📞 **Support**

- **Staging API:** https://api-staging.openloophealth.com
- **Documentation:** https://docs-staging.openloophealth.com
- **Support:** support@openloophealth.com

---

## 📝 **Tags in OpenAPI**

Current API tags:
- ✅ **Patients (MVP)** - 5 endpoints
- ✅ **Questionnaires (MVP)** - 3 endpoints
- ✅ **Products (MVP)** - 1 endpoint
- ✅ **Orders (MVP)** - 5 endpoints
- ✅ **Uploads** - 1 endpoint
- ✅ **Programs** - 2 endpoints
- ✅ **System** - 1 endpoint

**Total: 18 endpoints ready for production**

---

**Version:** 1.1.0  
**Focus:** Medical Weight Loss (MWL)  
**Status:** ✅ Ready for Integration  
**Last Updated:** December 19, 2025

