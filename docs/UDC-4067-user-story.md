# UDC-4067 — Update and Enhance Tasks in Salesforce

---

## User Story

**As a** Clinical Training Manager,
**I want** the Salesforce Task object to support new task types (VIP Preference, VIP Training/Onboarding, VIP Case Review, New Account Training, Existing Account Training, Case Review, Check In, and Request Training) with their corresponding Record Types, Page Layouts, and fields,
**so that** I can accurately track and manage customer interactions without relying on workarounds or free-text notes.

---

## Description

The current Task configuration in Salesforce does not reflect the operational workflows used by CSL, IS, RSM, and Clinical teams. Stakeholders need purpose-built task types — each with a dedicated Record Type and Page Layout — to capture interaction details (training methods, software versions, case review dates, engagement types, etc.) in a structured, reportable way.

This story covers the full configuration change: creating 7 new Record Types + Page Layouts, editing the existing "Case Review" Record Type and Page Layout to the new structure, deactivating 3 legacy Record Types, and assigning all layouts to all Profiles.

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
| Case Review RT existing | Edit existing RT and Page Layout to new structure (IS scope); create new RT for VIP Case Review (CSL scope) |

---

## Record Type Inventory

| Record Type Name | Description | Action | Completed By |
|------------------|-------------|--------|--------------|
| VIP Preference | Completed by CSL to capture VIP preferences | Create new | CSL |
| VIP Training/Onboarding | Completed by CSL to capture training/onboarding | Create new | CSL |
| VIP Case Review | Completed by CSL to capture VIP Case Reviews | Create new | CSL |
| New Account Training | Completed by IS to capture initial training | Create new | IS |
| Existing Account Training | Completed by IS to capture training with an existing customer | Create new | IS |
| Case Review | Completed by IS to capture Case Reviews | Edit existing RT + Page Layout | IS |
| Check in | Completed by Clinical Team or RSM to capture check in calls | Create new | Clinical Team / RSM |
| Request Training | Completed by RSM to request training | Create new | RSM |
| Advanced Case Review | — | Deactivate | — |
| Advanced Training | — | Deactivate | — |
| Compass Tool Activity | — | Deactivate | — |

---

## Page Layout Assignment Rule

- Each Record Type has its own Page Layout with the **same name as the Record Type**.
- Page Layout assignments by role:

| Role | Profile(s) |
|------|------------|
| CSL | uLab Clinical Training & Development |
| IS | uLab Clinical Training & Development |
| RSM | uLab Area Sales Directors, uLab Sales Manager |
| Clinical Team | uLab uAssist Lead & Sr. Trainer |

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
  Training Method (picklist: In Person / Online / Combo),
  Training Date, Notes (unlimited text), Patient Name
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
  Case Review Date, Notes (unlimited text), Patient Name
```

### Scenario 5: Check In Page Layout renders correctly
```
Given a user creates a task using the "Check In" Record Type
When the task form loads
Then the following fields are displayed:
  Assigned To, Status, Subject, Priority, Due Date, Related To (Company name),
  Name (office contact), Reminder Set (checkbox), Date, Time,
  Engagement Type (picklist: 14 day / 30 day / 60 day / 90 day),
  Scheduling options (multi-select picklist: Schedule VIP Preference Review /
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
  Assigned To (standard single-user field), Status,
  Subject (picklist: New Account Training / Existing Account Training / VIP Training),
  Priority, Date Sent, Related To (Company name), Name (office contact),
  Communication Preference (picklist: Office phone / Cell / Text / Email / In Person),
  Printer (text), Scanner (picklist: Alliedstar / Medit / 3Shape / iTero / Shining / Other),
  EasyRx (picklist: Yes / No),
  Pricing Tier (currency field — manually entered by RSM),
  Training Goals & Expectations (unlimited text),
  Additional Account Notes (unlimited text)
```

### Scenario 8: Deactivated Record Types preserve existing Task records
```
Given the Record Types Advanced Case Review, Advanced Training, and Compass Tool Activity are deactivated
When an admin queries existing Task records with those Record Types
Then the records remain intact and queryable via SOQL and Reports
And no data migration or deletion occurs
And no active Flows or validation rules reference the deactivated Record Types
```

### Scenario 9: Page Layout assignment covers all Profiles
```
Given a Page Layout is assigned for a given Record Type
When any user from any Profile creates a Task with that Record Type
Then they see the correct Page Layout for that Record Type
And no Profile receives the wrong layout or a blank layout
```

### Salesforce Non-Functional AC
- [ ] Record Types configured with `Active = true`; deactivated RTs set to `Active = false` — existing records are not deleted
- [ ] Page Layouts named identically to their Record Type for traceability
- [ ] All new custom fields comply with FLS: read-only profiles cannot edit them
- [ ] Multi-select picklist for "Additional Features Reviewed" uses standard SF multi-select picklist type (reportable)
- [ ] No automation (Flow/Apex) added in this story — declarative metadata only
- [ ] Deployment via metadata (Record Types, Page Layouts, Page Layout Assignments) — no change sets if SF CLI is available

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

**13 points** — 7 new Record Types + 1 edited Record Type + 8 Page Layouts + 21 custom fields across 4 field sets + multi-select picklist configuration + deactivation of 3 legacy RTs + layout assignments to 4 profiles. Purely declarative, but high layout volume.

---

## Subtasks

### ST-1 — Create custom fields shared across layouts
Create all net-new custom fields on the Task object before building layouts:
- `Date_Sent__c` — Date
- `Date_Received__c` — Date
- `Date_Uploaded_to_SFDC__c` — Date
- `Software_Version__c` — Picklist (Cloud, Desktop, uDesign 9.1, Earlier version)
- `Software_Version_Number__c` — Text(50)
- `Additional_Features_Reviewed__c` — Multi-Select Picklist (Account Settings, SmartRx Forms, Integrations, Refinement, IDB)
- `Training_Method__c` — Picklist (In Person, Online, Combo)
- `Training_Date__c` — Date
- `Patient_Name__c` — Text(80)
- `Case_Review_Method__c` — Picklist (In Person, Online, Combo)
- `Case_Review_Date__c` — Date
- `Reminder_Set__c` — Checkbox
- `Engagement_Type__c` — Picklist (14 day, 30 day, 60 day, 90 day)
- `Scheduling_Options__c` — Multi-Select Picklist (Schedule VIP Preference Review, Schedule VIP Onboarding/Training, Schedule VIP Case Review, Schedule Training, Schedule New Feature Training, Schedule Case Review, Schedule RSM Visit, Team Educational Meeting, Schedule Check In, Other)
- `Communication_Preference__c` — Picklist (Office phone, Cell, Text, Email, In Person)
- `Printer__c` — Text(100)
- `Scanner__c` — Picklist (Alliedstar, Medit, 3Shape, iTero, Shining, Other)
- `EasyRx__c` — Picklist (Yes, No)
- `Pricing_Tier__c` — Currency
- `Training_Goals__c` — Long Text Area(32768)
- `Additional_Account_Notes__c` — Long Text Area(32768)

### ST-2 — Edit existing Record Type and Page Layout: Case Review (IS)
- Update RT description: "Completed by IS to capture Case Reviews"
- Rebuild Page Layout "Case Review" with fields: Assigned To, Priority, Subject, Status, Due Date, Related To, Name, Additional Features Reviewed, Case Review Method, Case Review Date, Notes, Patient Name
- Assign Page Layout "Case Review" to: uLab Clinical Training & Development

### ST-3 — Create Record Type and Page Layout: VIP Preference (CSL)
- Create RT "VIP Preference" — Description: "Completed by CSL to capture VIP preferences" — Active: true
- Create Page Layout "VIP Preference" with fields: Assigned To, Priority, Subject, Status, Date Sent, Date Received, Date Uploaded to SFDC, Related To, Name, Notes
- Assign to: uLab Clinical Training & Development

### ST-4 — Create Record Type and Page Layout: VIP Training/Onboarding (CSL)
- Create RT "VIP Training/Onboarding" — Description: "Completed by CSL to capture training/onboarding" — Active: true
- Create Page Layout "VIP Training/Onboarding" using Training field set (see ST-1)
- Assign to: uLab Clinical Training & Development

### ST-5 — Create Record Type and Page Layout: VIP Case Review (CSL)
- Create RT "VIP Case Review" — Description: "Completed by CSL to capture VIP Case Reviews" — Active: true
- Create Page Layout "VIP Case Review" using Case Review field set (see ST-1)
- Assign to: uLab Clinical Training & Development

### ST-6 — Create Record Types and Page Layouts: New Account Training + Existing Account Training (IS)
- Create RT "New Account Training" — Description: "Completed by IS to capture initial training" — Active: true
- Create RT "Existing Account Training" — Description: "Completed by IS to capture training with an existing customer" — Active: true
- Create Page Layout "New Account Training" using Training field set
- Create Page Layout "Existing Account Training" using Training field set
- Assign both layouts to: uLab Clinical Training & Development

### ST-7 — Create Record Type and Page Layout: Check In (Clinical Team / RSM)
- Create RT "Check in" — Description: "Completed by Clinical Team or RSM to capture check in calls" — Active: true
- Create Page Layout "Check in" with fields: Assigned To, Status, Subject, Priority, Due Date, Related To, Name, Reminder Set, Date, Time, Engagement Type, Scheduling Options, Communication Preference, Notes
- Assign to: uLab uAssist Lead & Sr. Trainer, uLab Area Sales Directors, uLab Sales Manager

### ST-8 — Create Record Type and Page Layout: Request Training (RSM)
- Create RT "Request Training" — Description: "Completed by RSM to request training" — Active: true
- Create Page Layout "Request Training" with fields: Assigned To, Status, Subject, Priority, Date Sent, Related To, Name, Communication Preference, Printer, Scanner, EasyRx, Pricing Tier, Training Goals & Expectations, Additional Account Notes
- Assign to: uLab Area Sales Directors, uLab Sales Manager

### ST-9 — Deactivate legacy Record Types
- Set Active = false on: Advanced Case Review, Advanced Training, Compass Tool Activity
- Run SOQL to confirm existing Task records with those RTs are intact: `SELECT Id, RecordType.Name FROM Task WHERE RecordType.Name IN ('Advanced Case Review','Advanced Training','Compass Tool Activity')`
- Confirm no active Flow references these Record Types before deactivation

### ST-10 — QA, UAT, and deployment
- Validate all 8 active Record Types in sandbox with representative users (CSL, IS, RSM, Clinical Team)
- Verify correct Page Layout renders per Record Type for each Profile
- PO sign-off on each layout
- Deploy to production via SF CLI (`sf project deploy start`) or change set

---

## Open Questions

All open questions resolved. No pending items.
