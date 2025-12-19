# Questionnaires Workflow - Medical Weight Loss MVP

## 🎯 **Overview**

The questionnaire system separates **user-facing questions** from **internal business logic**. Partners retrieve questionnaire structures, collect patient responses, and receive automated eligibility assessments.

---

## 📋 **Complete Workflow**

### **Step 1: Retrieve Questionnaire Structure**

```bash
GET /questionnaires?program_id=MWL&type=intake
```

**Response:**
```json
{
  "message": "Questionnaires retrieved successfully",
  "data": {
    "questionnaires": [
      {
        "id": "q_mwl_intake_v1",
        "program_id": "MWL",
        "type": "intake",
        "title": "Medical Weight Loss - Initial Assessment",
        "items": [
          {
            "link_id": "weight_goal",
            "text": "What is your weight loss goal?",
            "type": "choice",
            "required": true,
            "read_only": false,  // USER ANSWERS THIS
            "answer_options": [
              {
                "value_coding": {
                  "code": "lose_21-50_lbs",
                  "display": "Lose 21-50 lbs for good"
                }
              }
            ]
          },
          {
            "link_id": "height_feet",
            "text": "What is your height (feet)?",
            "type": "integer",
            "required": true,
            "read_only": false  // USER ANSWERS THIS
          },
          {
            "link_id": "bmi",
            "text": "BMI",
            "type": "decimal",
            "required": true,
            "read_only": true  // CALCULATED INTERNALLY - DON'T SUBMIT
          },
          {
            "link_id": "allergy_medication_description",
            "text": "Please provide allergy details",
            "type": "string",
            "required": true,
            "read_only": false,
            "enabled_when": {  // CONDITIONAL DISPLAY
              "conditions": [
                {
                  "question_link_id": "medication_allergies",
                  "operator": "=",
                  "answer": { "value_string": "yes" }
                }
              ],
              "behavior": "all"
            }
          }
        ]
      }
    ]
  }
}
```

---

### **Step 2: Display Questions to Patient**

**Partner's Responsibility:**
1. Show questions where `read_only: false`
2. Apply conditional logic using `enabled_when`
3. Validate `required` fields
4. Collect answers from patient

**Example UI Logic:**
```javascript
questionnaire.items.forEach(item => {
  if (item.read_only) {
    return; // Skip - this is calculated internally
  }
  
  // Check if question should be shown
  if (item.enabled_when) {
    const shouldShow = evaluateConditions(item.enabled_when, userAnswers);
    if (!shouldShow) return;
  }
  
  // Display question to user
  renderQuestion(item);
});
```

---

### **Step 3: Submit Patient Responses**

```bash
POST /questionnaire-responses
```

**Request Body:**
```json
{
  "patient_id": "pat_123",
  "questionnaire_id": "q_mwl_intake_v1",
  "program_id": "MWL",
  "answers": {
    // Only submit fields with read_only: false
    "weight_goal": "lose_21-50_lbs",
    "height_feet": 5,
    "height_inches": 5,
    "weight": 160,
    "gender": "female",
    "date_of_birth": "1990-01-01",
    "state": "AL",
    "main_reason": "health_risks",
    "medical_exclusion_criteria": ["none"],
    "prior_weight_loss_meds_use": "no",
    "any_medications": "no",
    "medication_allergies": "no",
    "blood_pressure_range": "normal"
    
    // DON'T INCLUDE:
    // "bmi": 26.62,  ❌ Calculated internally
    // "age": 35,     ❌ Calculated internally
    // "height": 65   ❌ Calculated internally
  }
}
```

---

### **Step 4: Backend Processing (Automatic)**

**OpenLoop automatically calculates:**

```javascript
// 1. Derived fields
calculated_fields = {
  bmi: calculateBMI(height_feet, height_inches, weight),
  age: calculateAge(date_of_birth),
  height: (height_feet * 12) + height_inches,
  comorbidity_required: (bmi >= 25 && bmi < 27)
};

// 2. Eligibility evaluation
eligibility = {
  basic_mwl_eligibility: evaluateExclusions(answers),
  exclusion_reason: getExclusionReason(answers),
  sync_visit: requiresSyncConsult(answers, calculated_fields),
  sync_visit_reason: getSyncReason(answers, calculated_fields),
  clearance_required: needsClearance(answers)
};

// 3. Risk assessment
risk_assessment = {
  eligible: eligibility.basic_mwl_eligibility,
  risk_level: calculateRiskLevel(answers, calculated_fields),
  requires_sync_consult: eligibility.sync_visit,
  requires_clearance: eligibility.clearance_required,
  exclusion_reason: eligibility.exclusion_reason,
  sync_visit_reason: eligibility.sync_visit_reason,
  next_step: determineNextStep(eligibility)
};
```

---

### **Step 5: Receive Assessment Results**

**Response:**
```json
{
  "message": "Questionnaire response submitted successfully",
  "data": {
    "questionnaire_response": {
      "id": "qr_abc123",
      "patient_id": "pat_123",
      "questionnaire_id": "q_mwl_intake_v1",
      "program_id": "MWL",
      "status": "completed",
      
      "risk_assessment": {
        "eligible": true,
        "risk_level": "low",
        "requires_sync_consult": false,
        "requires_clearance": false,
        "exclusion_reason": null,
        "sync_visit_reason": null,
        "next_step": "allow_async_order"
      },
      
      "calculated_fields": {
        "bmi": 26.62,
        "age": 35,
        "height": 65,
        "comorbidity_required": true
      },
      
      "completed_at": "2025-01-15T14:30:00Z",
      "created_at": "2025-01-15T14:25:00Z"
    }
  }
}
```

---

### **Step 6: Proceed Based on `next_step`**

```javascript
switch (response.risk_assessment.next_step) {
  case "allow_async_order":
    // Patient is eligible - proceed to order creation
    createOrder({
      patient_id: patient.id,
      program_id: "MWL",
      product_id: selected_product_id,
      questionnaire_response_id: response.id
    });
    break;
    
  case "require_sync_consult":
    // Patient needs live consultation with provider
    showMessage("Your responses require a consultation with a provider");
    scheduleConsultation();
    break;
    
  case "require_clearance":
    // Patient needs clearance from PCP
    showMessage("Please obtain clearance from your primary care provider");
    requestClearanceDocumentation();
    break;
    
  case "exclude":
    // Patient is not eligible
    showMessage(`We cannot proceed: ${response.risk_assessment.exclusion_reason}`);
    provideResourcesAndSupport();
    break;
}
```

---

## 🔑 **Key Fields Explained**

### **`read_only` Field**

| Value | Meaning | Action |
|-------|---------|--------|
| `false` | User answers | Display question, collect answer, submit to API |
| `true` | Backend calculated | Don't display, don't submit - computed automatically |

**Examples:**
- `read_only: false` → `weight_goal`, `height_feet`, `medical_exclusion_criteria`
- `read_only: true` → `bmi`, `age`, `height`, `basic_mwl_eligibility`, `sync_visit`

---

### **`enabled_when` Conditional Logic**

Shows/hides questions based on previous answers.

**Example:**
```json
{
  "link_id": "allergy_medication_description",
  "text": "Please provide allergy details",
  "enabled_when": {
    "conditions": [
      {
        "question_link_id": "medication_allergies",
        "operator": "=",
        "answer": { "value_string": "yes" }
      }
    ],
    "behavior": "all"
  }
}
```

**Logic:** Only show `allergy_medication_description` if user answered "yes" to `medication_allergies`.

**Behaviors:**
- `all` - ALL conditions must be true (AND logic)
- `any` - ANY condition must be true (OR logic)

---

### **`risk_assessment.next_step` Values**

| Value | Meaning | Next Action |
|-------|---------|-------------|
| `allow_async_order` | Eligible, low risk | Create order immediately |
| `require_sync_consult` | Needs provider review | Schedule live consultation |
| `require_clearance` | Needs PCP clearance | Request clearance documentation |
| `exclude` | Not eligible | Show resources, cannot proceed |

---

## 📊 **Eligibility Rules (MWL)**

### **Absolute Exclusions (`exclude`):**

If patient answers "yes" to any of these → `eligible: false`:
- Severe kidney/liver disease
- Eating disorder
- Suicidal thoughts
- Active cancer
- Organ transplant on anti-rejection meds
- Severe GI condition
- Substance abuse disorder
- MTC/MEN2 history
- Type 1 diabetes or Type 2 on insulin
- Pancreatitis
- Pregnancy/breastfeeding
- Blood pressure ≥160/100
- Heart rate >110 bpm

---

### **Requires Clearance (`require_clearance`):**

Patient eligible but needs PCP/specialist approval:
- Birth <6 months ago
- IBD
- Bariatric surgery <1 year
- SIADH or low sodium
- Chronic opiate use
- Personal history thyroid cancer (non-MTC)

---

### **BMI Requirements:**

```javascript
if (prior_glp1_use === true) {
  minimum_bmi = 22;
} else if (has_weight_related_comorbidity) {
  minimum_bmi = 25;
} else {
  minimum_bmi = 27;
}

// Check comorbidity requirement
if (bmi >= 25 && bmi < 27 && !has_comorbidity) {
  eligible = false;
  exclusion_reason = "bmi_without_comorbidity";
  next_step = "exclude";
}
```

**Weight-related comorbidities:**
- Hypertension
- Sleep apnea
- Prediabetes
- Type 2 diabetes (not on insulin)
- High cholesterol
- PCOS
- Etc.

---

## 🔄 **Questionnaire Types**

### **1. Intake (`type: "intake"`)**
- First assessment for new patients
- Comprehensive medical history
- Establishes baseline eligibility

### **2. Refill (`type: "refill"`)**
- Follow-up questionnaire for existing patients
- Updates on medication tolerance
- Current weight and vitals
- Changes in medical history

---

## 🎯 **Partner Implementation Checklist**

- [ ] Retrieve questionnaire using `GET /questionnaires?program_id=MWL&type=intake`
- [ ] Display only questions with `read_only: false`
- [ ] Implement `enabled_when` conditional logic for showing/hiding questions
- [ ] Validate `required` fields before submission
- [ ] Submit only user-provided answers (not calculated fields)
- [ ] Handle `risk_assessment.next_step` appropriately
- [ ] Show calculated BMI, age to user from `calculated_fields`
- [ ] Route patients based on eligibility results

---

## 📝 **Example: Complete Integration**

```javascript
// 1. Get questionnaire
const { questionnaires } = await fetch(
  '/questionnaires?program_id=MWL&type=intake'
).then(r => r.json());

const questionnaire = questionnaires[0];

// 2. Display questions (pseudo-code)
const userAnswers = {};
questionnaire.items.forEach(item => {
  if (item.read_only) return; // Skip calculated fields
  
  if (item.enabled_when && !shouldShowQuestion(item, userAnswers)) {
    return; // Skip conditional questions
  }
  
  // Show question to user and collect answer
  userAnswers[item.link_id] = getUserAnswer(item);
});

// 3. Submit responses
const response = await fetch('/questionnaire-responses', {
  method: 'POST',
  body: JSON.stringify({
    patient_id: patient.id,
    questionnaire_id: questionnaire.id,
    program_id: 'MWL',
    answers: userAnswers
  })
});

const { questionnaire_response } = response.data;

// 4. Show results to user
console.log(`BMI: ${questionnaire_response.calculated_fields.bmi}`);
console.log(`Age: ${questionnaire_response.calculated_fields.age}`);

// 5. Proceed based on eligibility
if (questionnaire_response.risk_assessment.next_step === 'allow_async_order') {
  // Show products and create order
  proceedToOrder(questionnaire_response.id);
} else {
  // Handle other scenarios
  handleIneligibility(questionnaire_response.risk_assessment);
}
```

---

## 🚀 **Benefits of This Approach**

1. **Separation of Concerns**
   - Partners handle UI/UX
   - OpenLoop handles clinical logic

2. **Security**
   - Medical exclusion rules are private
   - Partners can't manipulate eligibility

3. **Flexibility**
   - Questionnaires can evolve without partner changes
   - Conditional logic managed centrally

4. **Scalability**
   - Same pattern works for intake, refill, and future questionnaire types
   - Easy to add new programs

---

**Last Updated:** December 19, 2025  
**API Version:** 1.1.0  
**Program:** Medical Weight Loss (MWL)  
**Questionnaires:** Intake ✅ | Refill 🔜

