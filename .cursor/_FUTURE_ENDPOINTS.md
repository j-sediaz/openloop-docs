# Future Endpoints - Post-MVP

This document contains endpoints that were removed from the MVP API but will be implemented in future releases. Keep this as a reference for the roadmap.

---

## 📋 **Documents Endpoints** (6 endpoints)

Patient identity verification and document management.

### **POST /documents**
Create a new document record for a patient after file upload.

**Request Body:**
```json
{
  "patient_id": "pat_123",
  "type": "drivers_license",
  "file_url": "https://cdn.openloop.com/files/doc_123.pdf",
  "issued_date": "2020-01-15",
  "expiration_date": "2025-01-15",
  "issuing_authority": "CA DMV"
}
```

**Response:**
```json
{
  "message": "Document created successfully",
  "data": {
    "document": {
      "id": "doc_abc123",
      "patient_id": "pat_123",
      "type": "drivers_license",
      "status": "pending",
      "file_url": "https://cdn.openloop.com/files/doc_123.pdf",
      "created_at": "2025-12-19T10:00:00Z"
    }
  }
}
```

---

### **GET /documents**
List all documents with optional filtering.

**Query Parameters:**
- `patient_id` (string) - Filter by patient ID
- `type` (enum) - Filter by document type: `drivers_license`, `passport`, `state_id`, `insurance_card`, `other`
- `status` (enum) - Filter by status: `pending`, `verified`, `rejected`, `expired`
- `limit` (integer, default: 50) - Max results
- `offset` (integer, default: 0) - Skip results

---

### **GET /documents/{id}**
Retrieve a specific document by ID.

---

### **PATCH /documents/{id}**
Update document information including verification status.

**Request Body:**
```json
{
  "status": "verified",
  "verified_at": "2025-12-19T11:00:00Z",
  "verified_by": "admin_user_123",
  "notes": "Document verified successfully"
}
```

---

### **DELETE /documents/{id}**
Remove a document from the system.

---

### **GET /patients/{id}/documents** (Convenience)
Shortcut to get all documents for a specific patient.  
Equivalent to: `GET /documents?patient_id={id}`

---

## 🧬 **Lab Orders Endpoints** (5 endpoints)

Laboratory test ordering and results management.

### **POST /lab-orders**
Create a new lab order for a patient.

**Request Body:**
```json
{
  "patient_id": "pat_123",
  "order_id": "ord_456",
  "lab_provider": "Quest Diagnostics",
  "tests": [
    {
      "name": "Lipid Panel",
      "code": "80061",
      "notes": "Fasting required"
    }
  ],
  "priority": "routine"
}
```

**Response:**
```json
{
  "message": "Lab order created successfully",
  "data": {
    "lab_order": {
      "id": "lab_xyz789",
      "patient_id": "pat_123",
      "order_id": "ord_456",
      "status": "ordered",
      "lab_provider": "Quest Diagnostics",
      "tests": [...],
      "created_at": "2025-12-19T10:00:00Z"
    }
  }
}
```

---

### **GET /lab-orders**
Retrieve all lab orders with optional filtering.

**Query Parameters:**
- `patient_id` (string) - Filter by patient
- `order_id` (string) - Filter by order
- `status` (enum) - Filter by status: `ordered`, `received`, `completed`, `reviewed`, `cancelled`
- `limit` (integer, default: 50)
- `offset` (integer, default: 0)

---

### **GET /lab-orders/{id}**
Retrieve a specific lab order by ID.

---

### **PATCH /lab-orders/{id}**
Update lab order including status and results.

**Request Body:**
```json
{
  "status": "completed",
  "results": {
    "test_name": "Lipid Panel",
    "values": {
      "total_cholesterol": 180,
      "hdl": 55,
      "ldl": 100,
      "triglycerides": 125
    },
    "unit": "mg/dL",
    "reference_range": "Normal",
    "interpretation": "Within normal limits"
  },
  "completed_at": "2025-12-20T14:00:00Z",
  "provider_reviewed": true,
  "provider_notes": "Results look good"
}
```

---

### **GET /orders/{id}/labs** (Convenience)
Shortcut to get all lab orders for a specific order.  
Equivalent to: `GET /lab-orders?order_id={id}`

---

## 🚀 **Funnel Responses Endpoint** (1 endpoint)

All-in-one patient enrollment endpoint.

### **POST /funnel-responses**
Create patient, questionnaire response, and invoices in a single operation.

**Use Case:** Streamlined enrollment flow for partners who want to submit everything at once.

**Request Body:**
```json
{
  "patient": {
    "email": "john.doe@example.com",
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
  },
  "questionnaire": {
    "questionnaire_id": "q_mwl_intake",
    "program_id": "MWL",
    "answers": {
      "current_weight": "220",
      "target_weight": "180",
      "height": "5'10\"",
      "medical_history": "..."
    }
  },
  "product_id": "sem_w12",
  "metadata": {
    "referral_source": "partner_xyz"
  }
}
```

**Response:**
```json
{
  "message": "Funnel response created successfully",
  "data": {
    "patient_id": "pat_1a2b3c4d",
    "questionnaire_response_id": "qr_xyz789",
    "order_id": "ord_mno789",
    "invoice_ids": ["inv_001", "inv_002"]
  }
}
```

**Benefits:**
- Single API call for complete enrollment
- Atomic operation (all-or-nothing)
- Reduces integration complexity
- Better for high-volume partners

---

## 🔗 **Nested Program Routes** (2 endpoints)

Convenience endpoints for retrieving program-specific data.

### **GET /programs/{programId}/questionnaires**
Retrieve the questionnaire required for a specific program.

**Example:** `GET /programs/MWL/questionnaires`

**Alternative (MVP):** Use `GET /questionnaire-responses` and filter by program

---

### **GET /programs/{programId}/products**
Get all products associated with a specific program.

**Example:** `GET /programs/MWL/products`

**Alternative (MVP):** Use `GET /products?program_id=MWL`

---

## 📊 **Implementation Priority**

### **Phase 2 (Post-MVP):**
1. ✅ **Documents** - Patient identity verification (required for compliance)
2. ✅ **Funnel Responses** - Streamlined enrollment for partners

### **Phase 3 (Advanced Features):**
3. ✅ **Lab Orders** - Laboratory test integration (TRT, HRT programs)
4. ✅ **Nested Routes** - Convenience endpoints (optional)

---

## 🧩 **Related Schemas**

These schemas were also kept in the OpenAPI spec for future use:

### **Document Schemas:**
- `DocumentInput`
- `DocumentResponse`
- `DocumentUpdateInput`
- `DocumentListResponse`
- `DocumentType` (enum)
- `DocumentStatus` (enum)

### **Lab Order Schemas:**
- `LabOrderInput`
- `LabOrderResponse`
- `LabOrderUpdateInput`
- `LabOrderListResponse`
- `LabTest`
- `LabResult`

### **Funnel Response Schemas:**
- `FunnelResponseInput`
- `FunnelResponseOutput`

---

## 📝 **Notes for Implementation**

### **Documents:**
- Requires CDN/storage integration (S3, Cloudflare R2)
- Needs verification workflow (manual or automated)
- Consider OCR for automatic data extraction
- HIPAA compliance for storage

### **Lab Orders:**
- Integration with Quest Diagnostics, LabCorp, or similar
- HL7/FHIR standards for results
- Provider review workflow required
- Results need to be securely stored and accessible

### **Funnel Responses:**
- Should be transactional (all-or-nothing)
- Consider idempotency keys
- Proper error handling for partial failures
- Webhook notifications for completion

---

## 🔄 **Migration Path**

When implementing these endpoints:

1. **Add endpoints back to openapi.json**
2. **Update tags to remove "(MVP)" suffix**
3. **Implement backend routes**
4. **Add to integration tests**
5. **Update documentation examples**
6. **Notify partners of new features**

---

**Last Updated:** December 19, 2025  
**Total Removed Endpoints:** 14  
**Status:** Kept as reference for post-MVP development

