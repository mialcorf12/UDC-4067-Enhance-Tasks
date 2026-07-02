# UDC-4067 — Update and Enhance Tasks in Salesforce

---

## User Story

**As a** Clinical Training Manager,
**I want** the Salesforce Task object to support new task types (VIP Preference, VIP Training/Onboarding, VIP Case Review, New Account Training, Existing Account Training, Case Review, Check In, and Request Training) with their corresponding Record Types, Page Layouts, and fields,
**so that** I can accurately track and manage customer interactions without relying on workarounds or free-text notes.

---

## Description

The current Task configuration in Salesforce does not reflect the operational workflows used by CSL, IS, RSM, and Clinical teams. Stakeholders need purpose-built task types — each with a dedicated Record Type and Page Layout — to capture interaction details (training methods, software versions, case review dates, engagement types, etc.) in a structured, reportable way.

This story covers the full configuration change: creating 7 new Record Types + Page Layouts, editing the existing "Case Review" Record Type and Page Layout to the new structure, deactivating 3 legacy Record Types, and assigning layouts to the relevant Profiles.

---

## Implementation Decisions

| Decision | Resolution |
|----------|------------|
| Multi-assignee on Request Training | Use standard single Assigned To field — no custom solution needed |
| Record Types vs. Dynamic Forms | Record Types + Page Layouts (no Dynamic Forms) |
| "Additional Features Reviewed" field type | Multi-select picklist |
| "Date Uploaded to SFDC" on VIP Preference | Manually entered by CSL user — no Flow or formula required |
| Pricing Tier on Request Training | Currency field manually populated by RSM user — no auto-populate |
| Profiles for layout assignment | CSL → uLab Clinical Training & Development; IS → uLab Clinical Training & Development; RSM → uLab Area Sales Directors, uLab Sales Manager; Clinical Team → uLab uAssist Lead & Sr. Trainer |
| Case Review RT existing | Edit existing RT (dev name: `Step_5_Case_Review`) and Page Layout to new structure (IS scope); create new RT for VIP Case Review (CSL scope) |
| Reminder Set field | Use standard Activity/Task field `IsReminderSet` — no custom field needed |
| Patient Name field | Use existing custom field `Patient_s_Name__c` (label: Patient's Name) — no new field needed |
| Custom fields deployment target | Fields deployed to Activity (not Task directly) — visible on Task layouts via Activity inheritance |
| Training Method picklist | Conserve legacy value `In-Person` (with hyphen); `In Person` (with space) removed to avoid duplicates |
| Communication Preference picklist | Filtered per RT: Check In and Request Training show only spec values (Office phone, Cell, Text, Email, In Person) |
| Subject on Request Training | Use `Training_Type__c` custom picklist field for structured data — standard Subject remains free text |

---

## Record Type Inventory

| Record Type Name | Developer Name | Description | Action | Completed By |
|------------------|----------------|-------------|--------|--------------|
| VIP Preference | `VIP_Preference` | Completed by CSL to capture VIP preferences | Create new | CSL |
| VIP Training/Onboarding | `VIP_Training_Onboarding` | Completed by CSL to capture training/onboarding | Create new | CSL |
| VIP Case Review | `VIP_Case_Review` | Completed by CSL to capture VIP Case Reviews | Create new | CSL |
| New Account Training | `New_Account_Training` | Completed by IS to capture initial training | Create new | IS |
| Existing Account Training | `Existing_Account_Training` | Completed by IS to capture training with an existing customer | Create new | IS |
| Case Review | `Step_5_Case_Review` | Completed by IS to capture Case Reviews | Edit existing RT + Page Layout | IS |
| Check in | `Check_In` | Completed by Clinical Team or RSM to capture check in calls | Create new | Clinical Team / RSM |
| Request Training | `Request_Training` | Completed by RSM to request training | Create new | RSM |
| Advanced Case Review | `Advanced_Case_Review` | — | Deactivate (already inactive) | — |
| Advanced Training | `Step_6_Advanced_Training` | — | Deactivate (already inactive) | — |
| Compass Tool Activity | `Compass_Tool_Activity` | — | Deactivate (already inactive) | — |

---

## Page Layout Assignment Rule

- Each Record Type has its own Page Layout with the **same name as the Record Type**.
- Page Layout assignments by role:

| Role | Profile(s) | Record Types |
|------|------------|-------------|
| CSL | uLab Clinical Training & Development | VIP Preference, VIP Training/Onboarding, VIP Case Review, Case Review |
| IS | uLab Clinical Training & Development | New Account Training, Existing Account Training, Case Review |
| RSM | uLab Area Sales Directors, uLab Sales Manager | Check In, Request Training |
| Clinical Team | uLab uAssist Lead & Sr. Trainer | Check In |

---

## Custom Field Inventory

Fields deployed to **Activity** (visible on Task and Event via inheritance). Fields already existing in the org are noted.

| API Name | Label | Type | Layout(s) | Notes |
|----------|-------|------|-----------|-------|
| `Date_Sent__c` | Date Sent | Date | VIP Preference, Check In, Request Training | Deployed to Task |
| `Date_Received__c` | Date Received | Date | VIP Preference | Deployed to Activity |
| `Date_Uploaded_to_SFDC__c` | Date Uploaded to SFDC | Date | VIP Preference | Deployed to Activity; manually entered by CSL |
| `Software_Version__c` | Software Version | Picklist | Training layouts | Pre-existing in org |
| `Software_Version_Number__c` | Software Version Number | Text(50) | Training layouts | Deployed to Activity |
| `Additional_Features_Reviewed__c` | Additional Features Reviewed | Multi-Select Picklist | Training + Case Review layouts | Pre-existing in org |
| `Training_Method__c` | Training Method | Picklist | Training layouts | Pre-existing; values: **In-Person**, Online, Combo |
| `Training_Date__c` | Training Date | Date | Training layouts | Deployed to Activity |
| `Patient_s_Name__c` | Patient's Name | Text | Training + Case Review layouts | Pre-existing in org — reused (not a new field) |
| `Case_Review_Method__c` | Case Review Method | Picklist | Case Review layouts | Pre-existing in org |
| `Case_Review_Date__c` | Case Review Date | Date | Case Review layouts | Pre-existing in org |
| `IsReminderSet` | Reminder Set | Checkbox | Check In | **Standard Activity/Task field** — not custom |
| `Engagement_Type__c` | Engagement Type | Picklist | Check In | Pre-existing in org; new values added: 14 day, 30 day, 60 day, 90 day |
| `Scheduling_Options__c` | Scheduling Options | Multi-Select Picklist | Check In | Deployed to Activity |
| `Communication_Preference__c` | Communication Preference | Picklist | Check In, Request Training | Pre-existing; RT-filtered: Office phone, Cell, Text, Email, In Person |
| `Printer__c` | Printer | Text(100) | Request Training | Deployed to Activity |
| `Scanner__c` | Scanner | Picklist | Request Training | Deployed to Activity |
| `EasyRx__c` | EasyRx | Picklist (Yes, No) | Request Training | Deployed to Activity |
| `Pricing_Tier__c` | Pricing Tier | Currency | Request Training | Deployed to Activity; manually entered by RSM |
| `Training_Goals__c` | Training Goals & Expectations | Long Text Area(32768) | Request Training | Deployed to Activity |
| `Additional_Account_Notes__c` | Additional Account Notes | Long Text Area(32768) | Request Training | Pre-existing in org |
| `Training_Type__c` | Training Type | Picklist | Training layouts, Request Training | Deployed to Activity; values: New Account Training, Existing Account Training, VIP Training |

---

## Acceptance Criteria

### Scenario 1: New Record Types are available when creating a Task
```
Given a Salesforce user navigates to create a new Activity/Task on an Account or Contact record
When they select a Record Type
Then the following Record Types are available:
  - VIP Preference
  - VIP Training/Onboarding
  - VIP Case Review
  - New Account Training
  - Existing Account Training
  - Case Review
  - Check In
  - Request Training
And the following Record Types are inactive and no longer selectable:
  - Advanced Case Review
  - Advanced Training
  - Compass Tool Activity
```

### Scenario 2: Each Record Type renders its own Page Layout
```
Given a user selects a specific Record Type (e.g., "Check In")
When the Task record opens
Then only the Page Layout fields defined for that Record Type are visible
And no fields from other Record Types or deactivated layouts appear
```

### Scenario 3: Training Page Layout renders correctly (VIP Training/Onboarding, New Account Training, Existing Account Training)
```
Given a user creates a task using any training Record Type
When the task form loads
Then the following fields are displayed:
  Assigned To, Status, Subject, Priority, Due Date, Related To (Company name),
  Name (office contact), Software Version (picklist: Cloud / Desktop / uDesign 9.1 / Earlier version),
  Software Version Number (text), Additional Features Reviewed (multi-select picklist:
    Account Settings / SmartRx Forms / Integrations / Refinement / IDB),
  Training Method (picklist: In-Person / Online / Combo),
  Training Type (picklist: New Account Training / Existing Account Training / VIP Training),
  Training Date, Notes (unlimited text), Patient's Name
And the label "Course Check" does not appear — it is replaced by "Refinement"
```

### Scenario 4: Case Review Page Layout renders correctly (Case Review and VIP Case Review)
```
Given a user creates a task using either "Case Review" or "VIP Case Review" Record Type
When the task form loads
Then the following fields are displayed:
  Assigned To, Priority, Subject, Status, Due Date, Related To (Company name),
  Name (office contact), Additional Features Reviewed (multi-select picklist:
    Account Settings / SmartRx Forms / Integrations / Refinement / IDB),
  Case Review Method (picklist: In Person / Online / Combo),
  Case Review Date, Notes (unlimited text), Patient's Name
```

### Scenario 5: Check In Page Layout renders correctly
```
Given a user creates a task using the "Check In" Record Type
When the task form loads
Then the following fields are displayed:
  Assigned To, Status, Subject, Priority, Due Date, Related To (Company name),
  Name (office contact), Reminder Set (standard Activity checkbox — IsReminderSet),
  Date Sent, Engagement Type (picklist includes: 14 day / 30 day / 60 day / 90 day),
  Scheduling Options (multi-select picklist: Schedule VIP Preference Review /
    Schedule VIP Onboarding/Training / Schedule VIP Case Review / Schedule Training /
    Schedule New Feature Training / Schedule Case Review / Schedule RSM Visit /
    Team Educational Meeting / Schedule Check In / Other),
  Communication Preference (picklist: Office phone / Cell / Text / Email / In Person),
  Notes (unlimited text)
```

### Scenario 6: VIP Preference Page Layout renders correctly
```
Given a user creates a task using the "VIP Preference" Record Type
When the task form loads
Then the following fields are displayed:
  Assigned To, Priority, Subject, Status, Date Sent, Date Received,
  Date Uploaded to SFDC (manually entered by CSL), Related To (Company name),
  Name (office contact), Notes (unlimited text)
```

### Scenario 7: Request Training Page Layout renders correctly
```
Given a user creates a task using the "Request Training" Record Type
When the task form loads
Then the following fields are displayed:
  Assigned To (standard single-user field), Status, Subject, Priority, Date Sent,
  Related To (Company name), Name (office contact),
  Communication Preference (picklist: Office phone / Cell / Text / Email / In Person),
  Printer (text), Scanner (picklist: Alliedstar / Medit / 3Shape / iTero / Shining / Other),
  EasyRx (picklist: Yes / No),
  Pricing Tier (currency field — manually entered by RSM),
  Training Type (picklist: New Account Training / Existing Account Training / VIP Training),
  Training Goals & Expectations (unlimited text),
  Additional Account Notes (unlimited text),
  Notes (unlimited text)
```

### Scenario 8: Deactivated Record Types preserve existing Task records
```
Given the Record Types Advanced Case Review, Advanced Training, and Compass Tool Activity are deactivated
When an admin queries existing Task records with those Record Types
Then the records remain intact and queryable via SOQL and Reports
And no data migration or deletion occurs
And no active Flows or validation rules reference the deactivated Record Types
```

### Scenario 9: Page Layout assignment covers correct Profiles per Record Type
```
Given a Page Layout is assigned for a given Record Type
When a user from the corresponding Profile creates a Task with that Record Type
Then they see the correct Page Layout for that Record Type
And no Profile receives the wrong layout or a blank layout
```

### Salesforce Non-Functional AC
- [x] Record Types configured with `Active = true`; deactivated RTs set to `Active = false` — existing records are not deleted
- [x] Page Layouts named identically to their Record Type for traceability
- [x] Custom fields deployed to Activity — visible on Task layouts via platform inheritance
- [x] `IsReminderSet` (standard) used for Reminder Set — no custom field created
- [x] `Patient_s_Name__c` (existing) reused for Patient Name — no duplicate field created
- [x] `Training_Method__c` picklist deduplicated — `In Person` (space) removed, `In-Person` (hyphen) retained
- [x] `Communication_Preference__c` filtered per RT (Check In, Request Training) to show only spec values
- [x] Multi-select picklist for "Additional Features Reviewed" — standard SF type (reportable)
- [x] No automation (Flow/Apex) added in this story — declarative metadata only
- [x] Deployment via SF CLI (`sf project deploy start`) — no change sets

---

## Out of Scope

- Automation (Flow / Apex triggers) driven by Record Type — separate story if needed
- Email or notification logic tied to task creation
- Reporting or dashboard changes
- Mobile layout configuration
- Any changes to Opportunity, Case, or Account objects
- Dynamic Forms — not applicable (Record Types + Page Layouts confirmed)

---

## Story Points

**13 points** — 7 new Record Types + 1 edited Record Type + 8 Page Layouts + 20 net-new custom fields (2 existing fields reused: `Patient_s_Name__c`, `IsReminderSet`) + multi-select picklist configuration + deactivation of 3 legacy RTs + layout assignments to 4 profiles. Purely declarative, but high layout and field volume.

---

## Subtasks

### ST-1 — Deploy custom fields to Activity
Deploy all net-new custom fields to the Activity object (propagates to Task via platform inheritance):
- `Date_Sent__c` — Date → deployed to Task directly
- `Date_Received__c` — Date → Activity
- `Date_Uploaded_to_SFDC__c` — Date → Activity (manually entered by CSL)
- `Software_Version_Number__c` — Text(50) → Activity
- `Training_Date__c` — Date → Activity
- `Training_Method__c` — Picklist (In-Person, Online, Combo) → pre-existing; `In Person` value removed
- `Scheduling_Options__c` — Multi-Select Picklist → Activity
- `Printer__c` — Text(100) → Activity
- `Scanner__c` — Picklist (Alliedstar, Medit, 3Shape, iTero, Shining, Other) → Activity
- `EasyRx__c` — Picklist (Yes, No) → Activity
- `Pricing_Tier__c` — Currency → Activity (manually entered by RSM)
- `Training_Goals__c` — Long Text Area(32768) → Activity
- `Training_Type__c` — Picklist (New Account Training, Existing Account Training, VIP Training) → Activity
- **Reused (no new field)**: `Patient_s_Name__c` (existing, label: Patient's Name), `IsReminderSet` (standard Activity/Task)

### ST-2 — Edit existing Record Type and Page Layout: Case Review (IS)
- Developer name confirmed: `Step_5_Case_Review`
- Updated RT description: "Completed by IS to capture Case Reviews" ✅
- Rebuilt Page Layout "Case Review": Assigned To, Priority, Subject, Status, Due Date, Related To, Name, Additional Features Reviewed, Case Review Method, Case Review Date, Patient's Name, Notes
- Assigned to: uLab Clinical Training & Development ✅

### ST-3 — Create Record Type and Page Layout: VIP Preference (CSL)
- RT `VIP_Preference` — Active: true ✅
- Page Layout "VIP Preference": Assigned To, Priority, Subject, Status, Date Sent, Date Received, Date Uploaded to SFDC, Related To, Name, Notes
- Assigned to: uLab Clinical Training & Development ✅

### ST-4 — Create Record Type and Page Layout: VIP Training/Onboarding (CSL)
- RT `VIP_Training_Onboarding` — Active: true ✅
- Page Layout "VIP Training_Onboarding": Training field set (Software Version, Software Version Number, Additional Features Reviewed, Training Method, Training Type, Training Date, Patient's Name, Notes)
- Assigned to: uLab Clinical Training & Development ✅

### ST-5 — Create Record Type and Page Layout: VIP Case Review (CSL)
- RT `VIP_Case_Review` — Active: true ✅
- Page Layout "VIP Case Review": Case Review field set (Additional Features Reviewed, Case Review Method, Case Review Date, Patient's Name, Notes)
- Assigned to: uLab Clinical Training & Development ✅

### ST-6 — Create Record Types and Page Layouts: New Account Training + Existing Account Training (IS)
- RT `New_Account_Training` — Active: true ✅
- RT `Existing_Account_Training` — Active: true ✅
- Page Layouts "New Account Training" and "Existing Account Training": Training field set
- Assigned both to: uLab Clinical Training & Development ✅

### ST-7 — Create Record Type and Page Layout: Check In (Clinical Team / RSM)
- RT `Check_In` — Active: true ✅
- Page Layout "Check in": Assigned To, Status, Subject, Priority, Due Date, Related To, Name, IsReminderSet, Date Sent, Engagement Type, Scheduling Options, Communication Preference, Notes
- Communication Preference filtered to: Office phone, Cell, Text, Email, In Person ✅
- Assigned to: uLab uAssist Lead & Sr. Trainer, uLab Area Sales Directors, uLab Sales Manager ✅

### ST-8 — Create Record Type and Page Layout: Request Training (RSM)
- RT `Request_Training` — Active: true ✅
- Page Layout "Request Training": Assigned To, Status, Subject, Priority, Date Sent, Related To, Name, Communication Preference, Printer, Scanner, EasyRx, Pricing Tier, Training Type, Training Goals & Expectations, Additional Account Notes, Notes
- Communication Preference filtered to: Office phone, Cell, Text, Email, In Person ✅
- Assigned to: uLab Area Sales Directors, uLab Sales Manager ✅

### ST-9 — Deactivate legacy Record Types
- `Advanced_Case_Review` — already inactive in org ✅
- `Step_6_Advanced_Training` (label: Advanced Training) — already inactive in org ✅
- `Compass_Tool_Activity` — already inactive in org ✅
- SOQL verified: 0 Task records lost under these RTs ✅

### ST-10 — QA, UAT, and deployment to Staging + Production
- Validate all 8 active Record Types in Dev org — fields verified in Setup UI Layout Editor ✅
- Deploy to Staging: `sf project deploy start --target-org Staging`
- PO sign-off on each layout in Staging
- Deploy to Production: `sf project deploy start --target-org` Production

---

## Deployment Status

| Environment | Fields | Record Types | Layouts | Profiles | Status |
|-------------|--------|--------------|---------|----------|--------|
| Dev | ✅ | ✅ | ✅ | ✅ | Complete |
| Staging | — | — | — | — | Pending |
| Production | — | — | — | — | Pending |

---

## Open Questions

All open questions resolved. No pending items.
