# Medical Weight Loss (MWL) MVP - API Overview

## 🎯 **MVP Scope**

The initial Partners API MVP is focused **exclusively on Medical Weight Loss (MWL)** program delivery.

---

## ✅ **MVP Endpoints (Available Now)**

### **Core Patient Workflow**

```bash
# 1. Patient Management
POST   /patients              # Create patient
GET    /patients              # List patients
GET    /patients/{id}         # Get patient details
PATCH  /patients/{id}         # Update patient
DELETE /patients/{id}         # Delete patient

# 2. Questionnaire Management
POST   /questionnaire-responses     # Submit MWL intake questionnaire
GET    /questionnaire-responses     # List responses
GET    /questionnaire-responses/{id} # Get specific response

# 3. Product Catalog
GET    /products?program_id=MWL     # List MWL products

# 4. Order Management
POST   /orders                      # Create MWL subscription order
GET    /orders                      # List orders
GET    /orders/{id}                 # Get order details
PATCH  /orders/{id}                 # Update order (status, fulfillment)
DELETE /orders/{id}                 # Cancel order

# 5. Utilities
POST   /uploads                     # Upload files (for future use)
GET    /programs                    # List programs (returns MWL)
GET    /programs/MWL                # Get MWL program details
```

**Total: 16 endpoints ready for MWL**

---

## 🔮 **Coming Soon (Not in MVP)**

These endpoints are documented but marked as **deprecated** until launch:

- ❌ **Documents** - Patient identity verification
- ❌ **Lab Orders** - Lab test ordering and results
- ❌ **Funnel Responses** - All-in-one enrollment
- ❌ **Program Questionnaires** - Nested questionnaire retrieval
- ❌ **Program Products** - Nested product retrieval

**Use the MVP endpoints instead for now.**

---

## 💊 **MWL Product Catalog**

The MVP supports **16 MWL products** across 4 medication types and 4 supply durations:

### **Medication Types**

1. **Semaglutide Oral** - Oral GLP-1 medication
2. **Tirzepatide Oral** - Oral dual GLP-1/GIP medication  
3. **Semaglutide Injection** - Injectable GLP-1 medication
4. **Tirzepatide Injection** - Injectable dual GLP-1/GIP medication

### **Supply Durations**

- 4 weeks (28 days)
- 12 weeks (84 days)
- 24 weeks (168 days)
- 48 weeks (336 days)

### **Complete Product List (16 products)**

#### **Semaglutide Oral (4 products)**
```
osg_w04 - Oral Semaglutide 4 week supply
osg_w12 - Oral Semaglutide 12 week supply
osg_w24 - Oral Semaglutide 24 week supply
osg_w48 - Oral Semaglutide 48 week supply
```

#### **Tirzepatide Oral (4 products)**
```
otz_w04 - Oral Tirzepatide 4 week supply
otz_w12 - Oral Tirzepatide 12 week supply
otz_w24 - Oral Tirzepatide 24 week supply
otz_w48 - Oral Tirzepatide 48 week supply
```

#### **Semaglutide Injection (4 products)**
```
sem_w04 - Injectable Semaglutide 4 week supply
sem_w12 - Injectable Semaglutide 12 week supply
sem_w24 - Injectable Semaglutide 24 week supply
sem_w48 - Injectable Semaglutide 48 week supply
```

#### **Tirzepatide Injection (4 products)**
```
ter_w04 - Injectable Tirzepatide 4 week supply
ter_w12 - Injectable Tirzepatide 12 week supply
ter_w24 - Injectable Tirzepatide 24 week supply
ter_w48 - Injectable Tirzepatide 48 week supply
```

**All products:**
- ✅ Recurring subscriptions
- ✅ Require payment link
- ✅ Personalized care program

---

## 📝 **Complete MWL Patient Flow**

### **Step-by-Step Integration**

```bash
# 1. Create patient
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
    "country": "US",
    "zip_code": "90001",
    "default": true,
    "type": "home"
  }]
}
# Response: patient_id = "pat_abc123"

# 2. Submit MWL intake questionnaire
POST /questionnaire-responses
{
  "patient_id": "pat_abc123",
  "questionnaire_id": "q_mwl_intake",
  "program_id": "MWL",
  "answers": {
    "current_weight": "220",
    "target_weight": "180",
    "height": "5'10\"",
    "medical_history": "...",
    "current_medications": ["Metformin"]
  }
}
# Response: questionnaire_response_id = "qr_xyz789"
# Check: risk_assessment.requires_sync_consult

# 3. Browse MWL products
GET /products?program_id=MWL
# Returns: 16 MWL products (Semaglutide & Tirzepatide)

# 4. Create order with selected product
POST /orders
{
  "patient_id": "pat_abc123",
  "program_id": "MWL",
  "product_id": "sem_w12",  // 12-week Semaglutide Injection
  "questionnaire_response_id": "qr_xyz789"
}
# Response: order_id = "ord_mno789"
# Order status: pending_payment → paid → pending_review → approved → fulfilling → fulfilled → active

# 5. Track order status
GET /orders/ord_mno789
# Returns: Full order details with status, fulfillment, subscription info

# 6. Update fulfillment (when shipped)
PATCH /orders/ord_mno789
{
  "status": "fulfilled",
  "fulfillment": {
    "status": "shipped",
    "tracking_number": "1Z999AA...",
    "carrier": "UPS",
    "shipped_at": "2025-01-20T14:00:00Z"
  }
}
```

---

## 🔑 **Key Features**

### **Product Configuration**
- All MWL products are **recurring subscriptions**
- Billing intervals match supply duration (28, 84, 168, or 336 days)
- Stripe integration via payment links
- Personalized care program included

### **Order Management**
- Track order lifecycle from payment to delivery
- Update fulfillment status and tracking
- Manage subscription billing
- Cancel/pause subscriptions

### **Questionnaire & Risk Assessment**
- Submit MWL intake questionnaire
- Automatic risk level calculation
- Sync consultation routing if high-risk
- Store responses for provider review

---

## 🚀 **Getting Started with MWL MVP**

### **Prerequisites**
1. API key from OpenLoop
2. Stripe account for payment processing
3. MWL program questionnaire ID

### **Quick Start**
```bash
# 1. Test API connection
GET /health

# 2. Get MWL program details
GET /programs/MWL

# 3. List MWL products
GET /products?program_id=MWL

# 4. Create your first patient and order
POST /patients → POST /questionnaire-responses → POST /orders
```

---

## 📊 **API Version**

- **Version:** 1.1.0
- **Environment:** `https://api-staging.openloophealth.com`
- **Focus:** Medical Weight Loss (MWL)
- **Endpoints:** 16 active MVP endpoints
- **Products:** 16 MWL products (4 medications × 4 durations)

---

## 🎯 **What's Next**

After MWL MVP stabilizes, we'll add:
1. Patient document verification
2. Lab order integration
3. Additional programs (TRT, HRT, Microdosing, Longevity)
4. Enhanced provider workflows
5. Consultation scheduling

---

## 📞 **Support**

- **Staging API:** https://api-staging.openloophealth.com
- **Documentation:** https://docs-staging.openloophealth.com
- **Contact:** support@openloophealth.com

---

**Focus: Medical Weight Loss MVP** 🎯  
**Status: Ready for Integration** ✅  
**16 Products | 16 Endpoints | 1 Program** 💊

