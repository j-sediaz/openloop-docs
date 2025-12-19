# OpenLoop Partners API - Implementation Checklist

## ✅ **Implemented (Complete)**

### Core Resources
- [x] **Patients** - Full CRUD
  - Create, Read, Update, Delete patients
  - Search and pagination
  - Demographics and contact info

- [x] **Programs** - Read operations
  - List all programs
  - Get program details
  - Filter by bundle/gender
  - Get program questionnaires
  - Get program products

- [x] **Products** - Read operations
  - List all products
  - Filter by program/provider/status
  - Pagination support

- [x] **Orders** - Full lifecycle
  - Create orders
  - List/search orders
  - Get order details
  - Update order status
  - Delete orders
  - Enhanced with clinical workflows

### New Features (v1.1)

- [x] **File Uploads**
  - `POST /uploads` - Upload files, get secure URLs
  - Supports documents, lab results, prescriptions
  - 10MB max, PDF/JPG/PNG formats

- [x] **Questionnaire Responses**
  - `POST /questionnaire-responses` - Submit responses
  - `GET /questionnaire-responses` - List with filters
  - `GET /questionnaire-responses/{id}` - Get specific response
  - Automatic risk assessment
  - Clinical flags and next steps

- [x] **Patient Documents**
  - `POST /patients/{id}/documents` - Upload document record
  - `GET /patients/{id}/documents` - List patient documents
  - `GET /patients/{id}/documents/{doc_id}` - Get specific document
  - `DELETE /patients/{id}/documents/{doc_id}` - Delete document
  - `PUT /patients/{id}/documents/{doc_id}/verify` - Verify document
  - Support for: drivers_license, passport, state_id, insurance_card

- [x] **Lab Orders**
  - `POST /lab-orders` - Create lab order
  - `GET /lab-orders` - List with filters
  - `GET /lab-orders/{id}` - Get lab order
  - `PUT /lab-orders/{id}` - Update lab order
  - `GET /orders/{id}/labs` - Get order's labs (nested)
  - `POST /orders/{id}/labs/{lab_id}/results` - Submit results (nested)
  - Full lifecycle: ordered → received → completed → reviewed

- [x] **Order Enhancements**
  - Clinical status tracking (risk_level, eligibility_check, approval_status)
  - Labs array integration
  - Provider review workflow
  - Fulfillment tracking
  - Pricing breakdown
  - Subscription management
  - `PUT /orders/{id}/fulfillment` - Update shipping info

### Documentation

- [x] **Complete OpenAPI 3.1 Spec** - All endpoints documented
- [x] **API Changes Summary** - Full detailed changelog
- [x] **DM Summary** - Quick reference for announcements
- [x] **Developer Workflows** - Step-by-step guides for:
  - Uploading documents
  - Submitting questionnaires
  - Complete patient onboarding
  - Lab order workflows
  - Common mistakes to avoid

---

## ❓ **Potentially Needed (To Discuss)**

### Subscriptions Management
Currently orders have `subscription` fields but no dedicated endpoints.

**Possible additions:**
- `GET /subscriptions` - List subscriptions
- `GET /subscriptions/{id}` - Get subscription details
- `PUT /subscriptions/{id}` - Update subscription (pause/resume)
- `DELETE /subscriptions/{id}` - Cancel subscription
- `GET /patients/{id}/subscriptions` - Get patient's subscriptions

**Use case:** Partners want to manage recurring billing directly

---

### Webhooks
No webhook system currently defined.

**Possible additions:**
- `POST /webhook-endpoints` - Register webhook URL
- `GET /webhook-endpoints` - List webhook endpoints
- `DELETE /webhook-endpoints/{id}` - Remove webhook
- `GET /webhook-events` - List webhook delivery history

**Events to consider:**
- `order.created`
- `order.status_changed`
- `order.approved`
- `order.rejected`
- `order.shipped`
- `payment.succeeded`
- `payment.failed`
- `lab.results_received`
- `document.verified`

**Use case:** Real-time notifications for order status changes

---

### Payments
Currently orders reference `payment_method_id` and `payment_status` but no payment endpoints.

**Possible additions:**
- `POST /payment-intents` - Create payment intent
- `POST /payment-methods` - Save payment method
- `GET /payments/{id}` - Get payment details
- `POST /refunds` - Create refund

**Alternative:** Partners handle payments directly with Stripe/Chargebee and just pass IDs

---

### Provider Management
Orders have `assigned_provider_id` but no provider endpoints.

**Possible additions:**
- `GET /providers` - List available providers
- `GET /providers/{id}` - Get provider details
- `GET /providers/{id}/availability` - Get provider schedule
- `POST /consultations` - Schedule consultation

**Alternative:** This might be internal-only (not exposed to partners)

---

### Patient Portal Endpoints
Patients accessing their own data.

**Possible additions:**
- `GET /me` - Get current patient (authenticated as patient)
- `GET /me/orders` - My orders
- `GET /me/documents` - My documents
- `GET /me/lab-results` - My lab results
- `GET /me/prescriptions` - My prescriptions

**Use case:** Partner builds patient portal

---

### Prescriptions
Orders may result in prescriptions but no prescription endpoints.

**Possible additions:**
- `GET /orders/{id}/prescriptions` - Get order prescriptions
- `GET /prescriptions/{id}` - Get prescription details
- `POST /prescriptions/{id}/refill` - Request refill

**Use case:** Tracking prescription fills and refills

---

### Notes/Comments
Clinical notes and communication.

**Possible additions:**
- `POST /orders/{id}/notes` - Add note to order
- `GET /orders/{id}/notes` - Get order notes
- `POST /patients/{id}/notes` - Add clinical note

**Use case:** Partner providers adding notes

---

### Eligibility Checks
Pre-order eligibility verification.

**Possible additions:**
- `POST /eligibility-checks` - Check if patient eligible for program
  ```json
  {
    "patient_id": "pat_123",
    "program_id": "prog_trt_001"
  }
  ```
- Returns eligibility result before patient fills out questionnaire

**Use case:** Show which programs patient qualifies for upfront

---

### Appointment/Consultation Scheduling
For sync consultations.

**Possible additions:**
- `GET /availability` - Get available time slots
- `POST /consultations` - Schedule consultation
- `GET /consultations/{id}` - Get consultation details
- `PUT /consultations/{id}` - Reschedule
- `DELETE /consultations/{id}` - Cancel

**Use case:** When `requires_sync_consult: true`

---

### Analytics/Reporting
Partner performance metrics.

**Possible additions:**
- `GET /reports/orders` - Order statistics
- `GET /reports/revenue` - Revenue metrics
- `GET /reports/patients` - Patient counts

**Use case:** Partner dashboards

---

## 🎯 **Recommendations**

### Priority 1 (Strongly Recommended)
1. **Webhooks** - Critical for real-time order status updates
2. **Subscriptions Management** - Partners need to manage recurring billing
3. **Consultation Scheduling** - Required for high-risk patients

### Priority 2 (Nice to Have)
4. **Patient Portal Endpoints** - If partners want patient self-service
5. **Prescriptions** - For complete order visibility
6. **Eligibility Checks** - Better UX (show eligible programs upfront)

### Priority 3 (Can Wait)
7. **Provider Management** - Might be internal-only
8. **Payment Endpoints** - If payment is always via Stripe/Chargebee, not needed
9. **Notes/Comments** - Can be added later as needed
10. **Analytics** - Can build separately or use BI tools

---

## 📋 **Current API Coverage**

```
✅ Patient Management
✅ Document Verification
✅ Program Discovery
✅ Product Catalog
✅ Questionnaire Submission
✅ Order Creation
✅ Order Tracking
✅ Lab Ordering
✅ Lab Results
✅ Fulfillment Tracking
✅ File Uploads

⚠️  Subscription Management (partial - in orders but no dedicated endpoints)
⚠️  Payments (partial - referenced but not managed)
⚠️  Consultations (mentioned but no scheduling)

❌ Webhooks
❌ Provider Discovery
❌ Patient Portal
❌ Prescriptions
❌ Eligibility Pre-checks
❌ Analytics
```

---

## 🔍 **What's Missing from Current Implementation?**

### Identified Gaps

1. **No consultation scheduling** - Orders can require sync consults but no way to schedule them
2. **No webhook system** - Partners can't get real-time notifications
3. **Subscription endpoints** - Can't pause/resume/cancel subscriptions programmatically
4. **No prescription tracking** - Orders result in prescriptions but can't access them
5. **Payment is implicit** - Payments happen but aren't exposed via API

### Questions to Answer

1. **Who handles payments?**
   - If Stripe/Chargebee only → Current approach is fine
   - If partners need payment management → Add payment endpoints

2. **Who schedules consultations?**
   - If OpenLoop schedules internally → No endpoints needed
   - If partners schedule → Need consultation endpoints

3. **Do partners manage subscriptions?**
   - If yes → Add subscription management endpoints
   - If no → Current approach is fine

4. **Do partners need webhooks?**
   - If yes → Critical to add
   - If no → Can use polling

5. **Do patients access data directly?**
   - If yes → Need patient portal endpoints
   - If no → Partners fetch on patient's behalf (current approach)

---

## ✅ **What We Did Right**

1. ✅ Clear nested routes (`/patients/{id}/documents`, `/orders/{id}/labs`)
2. ✅ Two-step file upload process (clear and secure)
3. ✅ Automatic risk assessment on questionnaires
4. ✅ Comprehensive order lifecycle with clinical workflows
5. ✅ Detailed documentation with examples
6. ✅ Consistent error responses
7. ✅ Pagination on all list endpoints
8. ✅ Flexible metadata fields for extensibility

---

## 📞 **Next Steps**

1. **Review this checklist** with product team
2. **Prioritize missing features** based on partner needs
3. **Implement Priority 1** items (webhooks, subscriptions, consultations)
4. **Get partner feedback** on what they actually need
5. **Iterate** based on real-world usage

---

**Questions or feedback?** Update this checklist as decisions are made.

