# Developer Workflows - OpenLoop Partners API

Complete step-by-step workflows for common integration patterns. Follow these guides to avoid confusion and implement correctly.

---

## 📤 **Workflow 1: Upload Patient Documents**

### How It Works
Documents require a **two-step process**:
1. Upload the file to get a secure URL
2. Create the document record with that URL

### Step-by-Step

#### Step 1: Upload the File

```bash
POST /uploads
Content-Type: multipart/form-data

# Fields:
# - file: [binary file data]
# - type: "document"
```

**Example with cURL:**
```bash
curl -X POST https://api-staging.openloophealth.com/uploads \
  -H "x-api-key: YOUR_API_KEY" \
  -F "file=@/path/to/drivers_license.pdf" \
  -F "type=document"
```

**Response:**
```json
{
  "message": "File uploaded successfully",
  "data": {
    "file_url": "https://secure.openloop.com/files/abc123def456.pdf",
    "file_id": "file_abc123def456",
    "file_type": "application/pdf",
    "file_size": 245678,
    "expires_at": null
  }
}
```

**IMPORTANT:** Save the `file_url` from the response. You'll use it in the next step.

#### Step 2: Create Document Record

```bash
POST /documents
Content-Type: application/json

{
  "patient_id": "pat_123",
  "type": "drivers_license",
  "file_url": "https://secure.openloop.com/files/abc123def456.pdf",
  "file_type": "application/pdf",
  "document_number": "D1234567",
  "issuing_state": "CA",
  "issuing_country": "US",
  "expiration_date": "2028-05-15"
}
```

**Example with cURL:**
```bash
curl -X POST https://api-staging.openloophealth.com/documents \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "patient_id": "pat_123",
    "type": "drivers_license",
    "file_url": "https://secure.openloop.com/files/abc123def456.pdf",
    "file_type": "application/pdf",
    "document_number": "D1234567",
    "issuing_state": "CA",
    "issuing_country": "US",
    "expiration_date": "2028-05-15"
  }'
```

**Response:**
```json
{
  "message": "Document uploaded successfully",
  "data": {
    "document": {
      "id": "doc_xyz789",
      "patient_id": "pat_123",
      "type": "drivers_license",
      "status": "pending",
      "file_url": "https://secure.openloop.com/files/abc123def456.pdf",
      "uploaded_at": "2025-01-15T14:30:00Z"
    }
  }
}
```

### File Requirements

- **Max file size:** 10MB
- **Supported formats:** PDF, JPG, JPEG, PNG
- **Document types:**
  - `drivers_license` - Driver's License
  - `passport` - Passport
  - `state_id` - State ID Card
  - `insurance_card` - Insurance Card
  - `other` - Other documents

### Common Errors

**413 File Too Large:**
```json
{
  "error": "ValidationError",
  "message": "File size exceeds maximum allowed (10MB)"
}
```
**Solution:** Compress the file or reduce quality.

**400 Invalid File Type:**
```json
{
  "error": "ValidationError",
  "message": "File type not supported. Use PDF, JPG, or PNG."
}
```
**Solution:** Convert file to supported format.

### Complete Example (JavaScript)

```javascript
async function uploadPatientDocument(patientId, file) {
  // Step 1: Upload file
  const formData = new FormData();
  formData.append('file', file);
  formData.append('type', 'document');
  
  const uploadResponse = await fetch('https://api-staging.openloophealth.com/uploads', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.OPENLOOP_API_KEY
    },
    body: formData
  });
  
  const uploadData = await uploadResponse.json();
  const fileUrl = uploadData.data.file_url;
  
  // Step 2: Create document record
  const documentResponse = await fetch(
    'https://api-staging.openloophealth.com/documents',
    {
      method: 'POST',
      headers: {
        'x-api-key': process.env.OPENLOOP_API_KEY,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        patient_id: patientId,
        type: 'drivers_license',
        file_url: fileUrl,
        file_type: uploadData.data.file_type,
        document_number: 'D1234567',
        issuing_state: 'CA',
        issuing_country: 'US',
        expiration_date: '2028-05-15'
      })
    }
  );
  
  return await documentResponse.json();
}
```

---

## 📝 **Workflow 2: Submit Questionnaire Response**

### How It Works
Patient fills out a questionnaire → Submit answers → System evaluates risk → Use response ID for orders

### Step-by-Step

#### Step 1: Get Questionnaire Schema

```bash
GET /programs/{program_id}/questionnaires
```

**Example:**
```bash
curl -H "x-api-key: YOUR_API_KEY" \
  https://api-staging.openloophealth.com/programs/prog_trt_001/questionnaires
```

**Response:**
```json
{
  "message": "Questionnaire retrieved successfully",
  "data": {
    "questionnaire": {
      "id": "q_trt_intake",
      "program_id": "prog_trt_001",
      "title": "TRT Intake Questionnaire",
      "questions": [
        {
          "id": "main_symptom",
          "text": "What is your primary concern?",
          "type": "select",
          "required": true,
          "options": ["low_energy", "low_libido", "depressed_mood", "other"]
        },
        {
          "id": "duration_months",
          "text": "How long have you experienced these symptoms?",
          "type": "number",
          "required": true
        },
        {
          "id": "history_of_prostate_cancer",
          "text": "Have you been diagnosed with prostate cancer?",
          "type": "boolean",
          "required": true
        }
      ]
    }
  }
}
```

#### Step 2: Collect Patient Answers

Build a UI that collects answers for each question. Store answers as key-value pairs using the question `id` as the key.

#### Step 3: Submit Questionnaire Response

```bash
POST /questionnaire-responses
Content-Type: application/json

{
  "patient_id": "pat_xyz789",
  "questionnaire_id": "q_trt_intake",
  "program_id": "prog_trt_001",
  "answers": {
    "main_symptom": "low_energy",
    "duration_months": 12,
    "history_of_prostate_cancer": false,
    "current_medications": ["Lisinopril"]
  }
}
```

**Example with cURL:**
```bash
curl -X POST https://api-staging.openloophealth.com/questionnaire-responses \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "patient_id": "pat_xyz789",
    "questionnaire_id": "q_trt_intake",
    "program_id": "prog_trt_001",
    "answers": {
      "main_symptom": "low_energy",
      "duration_months": 12,
      "history_of_prostate_cancer": false
    }
  }'
```

**Response:**
```json
{
  "message": "Questionnaire response submitted successfully",
  "data": {
    "questionnaire_response": {
      "id": "qr_abc123",
      "patient_id": "pat_xyz789",
      "questionnaire_id": "q_trt_intake",
      "program_id": "prog_trt_001",
      "answers": {
        "main_symptom": "low_energy",
        "duration_months": 12,
        "history_of_prostate_cancer": false
      },
      "risk_assessment": {
        "risk_level": "low",
        "requires_sync_consult": false,
        "next_step": "allow_async_order",
        "flags": []
      },
      "completed_at": "2025-01-15T14:30:00Z",
      "created_at": "2025-01-15T14:30:00Z"
    }
  }
}
```

**IMPORTANT:** Save the `id` (questionnaire_response_id). You'll use it when creating an order.

#### Step 4: Use Response for Order

```bash
POST /orders
Content-Type: application/json

{
  "patient_id": "pat_xyz789",
  "program_id": "prog_trt_001",
  "product_id": "prod_trt_monthly_0100",
  "questionnaire_response_id": "qr_abc123",  // ← Use the ID from step 3
  "payment_method_id": "pm_stripe_abc123"
}
```

### Understanding Risk Assessment

The API automatically evaluates questionnaire responses and returns:

- **risk_level**: `low`, `medium`, or `high`
- **requires_sync_consult**: `true` if patient needs video consultation before ordering
- **next_step**: Recommended action (`allow_async_order`, `require_sync_consult`, etc.)
- **flags**: Array of clinical concerns raised

**If `requires_sync_consult: true`:**
- Do NOT allow immediate order creation
- Schedule a video consultation first
- Provider will review and potentially create order manually

**If `requires_sync_consult: false`:**
- Patient can proceed to order immediately
- Provider will review asynchronously

### Complete Example (JavaScript)

```javascript
async function submitQuestionnaireAndCreateOrder(patientId, programId, answers, productId, paymentMethodId) {
  // Step 1: Get questionnaire
  const questionnaireResponse = await fetch(
    `https://api-staging.openloophealth.com/programs/${programId}/questionnaires`,
    {
      headers: { 'x-api-key': process.env.OPENLOOP_API_KEY }
    }
  );
  const questionnaireData = await questionnaireResponse.json();
  const questionnaireId = questionnaireData.data.questionnaire.id;
  
  // Step 2: Submit answers
  const responseSubmit = await fetch(
    'https://api-staging.openloophealth.com/questionnaire-responses',
    {
      method: 'POST',
      headers: {
        'x-api-key': process.env.OPENLOOP_API_KEY,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        patient_id: patientId,
        questionnaire_id: questionnaireId,
        program_id: programId,
        answers: answers
      })
    }
  );
  
  const responseData = await responseSubmit.json();
  const questionnaireResponseId = responseData.data.questionnaire_response.id;
  const riskAssessment = responseData.data.questionnaire_response.risk_assessment;
  
  // Step 3: Check if sync consult required
  if (riskAssessment.requires_sync_consult) {
    return {
      success: false,
      reason: 'sync_consult_required',
      message: 'A video consultation is required before proceeding.'
    };
  }
  
  // Step 4: Create order
  const orderResponse = await fetch(
    'https://api-staging.openloophealth.com/orders',
    {
      method: 'POST',
      headers: {
        'x-api-key': process.env.OPENLOOP_API_KEY,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        patient_id: patientId,
        program_id: programId,
        product_id: productId,
        questionnaire_response_id: questionnaireResponseId,
        payment_method_id: paymentMethodId
      })
    }
  );
  
  return await orderResponse.json();
}
```

---

## 🛍️ **Workflow 3: Complete Patient Onboarding → Order**

The full end-to-end flow combining everything.

### Step-by-Step

```bash
# 1. Create patient
POST /patients
{
  "email": "john.doe@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "phone": "+1-555-0123",
  "birth_date": "1990-05-15",
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
# Response: patient_id = "pat_123"

# 2. Upload document file
POST /uploads
[multipart/form-data with file]
# Response: file_url = "https://secure.openloop.com/files/abc123.pdf"

# 3. Create document record
POST /documents
{
  "patient_id": "pat_123",
  "type": "drivers_license",
  "file_url": "https://secure.openloop.com/files/abc123.pdf",
  "file_type": "application/pdf",
  "document_number": "D1234567",
  "issuing_state": "CA",
  "issuing_country": "US",
  "expiration_date": "2028-05-15"
}
# Response: document_id = "doc_456"

# 4. Get questionnaire for program
GET /programs/prog_trt_001/questionnaires
# Response: questionnaire_id = "q_trt_intake"

# 5. Submit questionnaire response
POST /questionnaire-responses
{
  "patient_id": "pat_123",
  "questionnaire_id": "q_trt_intake",
  "program_id": "prog_trt_001",
  "answers": {
    "main_symptom": "low_energy",
    "duration_months": 12,
    "history_of_prostate_cancer": false
  }
}
# Response: questionnaire_response_id = "qr_789"
# Check: risk_assessment.requires_sync_consult = false (proceed)

# 6. Create order
POST /orders
{
  "patient_id": "pat_123",
  "program_id": "prog_trt_001",
  "product_id": "prod_trt_monthly_0100",
  "questionnaire_response_id": "qr_789",
  "payment_method_id": "pm_stripe_abc123"
}
# Response: order_id = "ord_001", status = "pending_payment"

# 7. Payment processed (by payment provider webhook)
# Order status changes: pending_payment → paid → pending_review

# 8. System automatically creates lab orders (if required by program)
GET /orders/ord_001/labs
# Response: labs = [{ "id": "labord_111", "lab_id": "lab_total_testosterone", "status": "ordered" }]

# 9. Patient completes labs, results are submitted
PATCH /lab-orders/labord_111
{
  "status": "completed",
  "value": "350",
  "unit": "ng/dL",
  "reference_range": { "min": 300, "max": 1000 },
  "flag": "normal",
  "results_received_at": "2025-01-14T16:30:00Z"
}

# 10. Provider reviews and approves
# Order status changes: pending_review → approved → fulfilling

# 11. Pharmacy fulfills and ships
PATCH /orders/ord_001
{
  "fulfillment": {
    "status": "shipped",
    "tracking_number": "1Z999AA10123456784",
    "carrier": "UPS",
    "shipped_at": "2025-01-15T14:00:00Z"
  }
}
# Order status changes: fulfilling → fulfilled → active
```

---

## 🧪 **Workflow 4: Standalone Lab Order**

When you need to order labs independently (not part of an order).

### Step-by-Step

```bash
# 1. Create lab order
POST /lab-orders
{
  "patient_id": "pat_xyz789",
  "lab_id": "lab_total_testosterone",
  "lab_provider": "Quest Diagnostics",
  "notes": "Fasting required"
}
# Response: lab_order_id = "labord_456"

# 2. Update status when patient completes
PATCH /lab-orders/labord_456
{
  "status": "received",
  "collected_at": "2025-01-16T09:00:00Z",
  "notes": "Sample received at lab"
}

# 3. Submit results when available
PATCH /lab-orders/labord_456
{
  "status": "completed",
  "value": "350",
  "unit": "ng/dL",
  "reference_range": { "min": 300, "max": 1000 },
  "flag": "normal",
  "in_range": true,
  "file_url": "https://secure.openloop.com/labs/result.pdf",
  "results_received_at": "2025-01-16T14:00:00Z"
}
```

---

## ⚠️ **Common Mistakes to Avoid**

### ❌ DON'T: Try to upload documents directly to `/documents`

```bash
# WRONG - This won't work
POST /documents
[file binary data]
```

### ✅ DO: Use two-step process

```bash
# CORRECT - Step 1: Upload file
POST /uploads
[file binary data]

# CORRECT - Step 2: Create document with URL
POST /documents
{ "patient_id": "pat_123", "file_url": "https://...", ... }
```

---

### ❌ DON'T: Create order before submitting questionnaire

```bash
# WRONG - No questionnaire_response_id
POST /orders
{
  "patient_id": "pat_123",
  "product_id": "prod_trt_001"
}
```

### ✅ DO: Submit questionnaire first, then create order

```bash
# CORRECT - Step 1: Submit questionnaire
POST /questionnaire-responses
{ "patient_id": "pat_123", "answers": {...} }

# CORRECT - Step 2: Create order with response ID
POST /orders
{
  "patient_id": "pat_123",
  "product_id": "prod_trt_001",
  "questionnaire_response_id": "qr_789"
}
```

---

### ❌ DON'T: Ignore risk assessment

```javascript
// WRONG - Always allowing orders
const response = await submitQuestionnaire(answers);
createOrder(response.id); // Might require sync consult!
```

### ✅ DO: Check risk assessment before creating order

```javascript
// CORRECT - Check assessment
const response = await submitQuestionnaire(answers);
if (response.risk_assessment.requires_sync_consult) {
  showMessage('Video consultation required');
  scheduleConsultation();
} else {
  createOrder(response.id);
}
```

---

## 📚 **Quick Reference**

### Upload Files
```
POST /uploads → file_url → Use in documents/labs/etc
```

### Submit Questionnaire
```
GET /programs/{id}/questionnaires → Submit answers → POST /questionnaire-responses → qr_id
```

### Create Order
```
POST /orders with questionnaire_response_id
```

### Upload Document
```
POST /uploads → POST /documents with { patient_id, file_url, ... }
```

### Lab Workflow
```
POST /lab-orders → Update status → Submit results
```

---

**Need Help?** Check the full API documentation or contact support@openloophealth.com

