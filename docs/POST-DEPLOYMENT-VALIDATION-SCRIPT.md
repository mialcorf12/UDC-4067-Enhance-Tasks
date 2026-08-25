# UDC-4067 Post-Deployment Validation Script

**Purpose**: Automated validation of UDC-4067 metadata in Production after deployment  
**Expected Duration**: 15-20 minutes  
**Audience**: QA Lead, Architect, Deployment Lead  
**Prerequisites**: Access to `sf` CLI with Production org alias `Prod`

---

## Script Overview

This document provides step-by-step commands to validate the UDC-4067 deployment in Production.

---

## Phase 1: Setup (5 min)

### 1.1 Confirm Production Org Connection

```bash
# Verify Prod alias exists and is connected
sf org list --all | grep Prod

# Expected output:
# Prod | Sandbox | [email protected] | 00D... | Connected
```

### 1.2 Prepare Working Directory

```bash
# Clone repo if not already present
cd /Users/albertocordero/Documents/VSC/UDC-4067\ Enhance\ Tasks

# Verify manifest exists
ls manifest/package.xml

# Verify validation scripts exist
ls docs/POST-DEPLOYMENT-VALIDATION-SCRIPT.md
```

---

## Phase 2: Metadata Verification (10 min)

### 2.1 Verify Custom Fields (Activity)

```bash
# Count custom fields on Activity object
sf data query \
  --target-org Prod \
  --query "SELECT COUNT() FROM CustomField WHERE TableEnumOrId='Activity' AND DeveloperName LIKE '%Training%' OR DeveloperName LIKE '%Communication%' OR DeveloperName LIKE '%Case_Review%'" \
  --json | jq '.result.totalSize'

# Expected output: >= 10 (at least these field groups present)

# More precise: Query each field explicitly
for FIELD in "Training_Date__c" "Training_Method__c" "Communication_Preference__c" "Case_Review_Date__c"; do
  echo "Checking $FIELD..."
  sf data query \
    --target-org Prod \
    --query "SELECT COUNT() FROM CustomField WHERE DeveloperName='$FIELD'" \
    --json | jq -r '.result.totalSize'
done

# Expected: Each should return "1"
```

### 2.2 Verify Page Layouts (Task)

```bash
# Retrieve all Task layouts from manifest
grep -A1 "Layout" manifest/package.xml | grep members | sed 's/.*<members>//; s/<\/members>.*//' | grep Task

# Expected output (8 layouts):
# Task-VIP Training_Onboarding
# Task-VIP Case Review
# Task-VIP Preference
# Task-Check in
# Task-Case Review
# Task-New Account Training
# Task-Existing Account Training
# Task-Request Training
```

### 2.3 Verify Record Types (Task)

```bash
# Query all Task record types
sf data query \
  --target-org Prod \
  --query "SELECT DeveloperName, IsActive FROM RecordType WHERE SObjectType='Task' ORDER BY DeveloperName" \
  --json | jq '.result.records[] | "\(.DeveloperName) (Active: \(.IsActive))"'

# Expected output (11 total: 8 active UDC-4067 + 3 inactive legacy):
# Advanced_Case_Review (Active: false)
# Check_In (Active: true)
# Compass_Tool_Activity (Active: false)
# Existing_Account_Training (Active: true)
# New_Account_Training (Active: true)
# Request_Training (Active: true)
# Step_5_Case_Review (Active: true)
# Step_6_Advanced_Training (Active: false)
# VIP_Case_Review (Active: true)
# VIP_Preference (Active: true)
# VIP_Training_Onboarding (Active: true)

# Count confirmation
sf data query \
  --target-org Prod \
  --query "SELECT COUNT() FROM RecordType WHERE SObjectType='Task'" \
  --json | jq '.result.totalSize'

# Expected: 11
```

### 2.4 Verify Profiles

```bash
# List all profiles in Production
sf data query \
  --target-org Prod \
  --query "SELECT Name FROM Profile WHERE Name LIKE 'uLab%' OR Name='Admin' ORDER BY Name" \
  --json | jq '.result.records[] | .Name'

# Expected output (at least these 5):
# Admin
# uLab Area Sales Directors
# uLab Clinical Training & Development
# uLab Sales Manager
# uLab uAssist Lead & Sr. Trainer
```

---

## Phase 3: Functional Testing (5 min)

### 3.1 Task Creation Validation

Create an Apex script to test Task creation with each record type:

```bash
# Create validation script
cat > /tmp/validate_udc4067_prod.apex <<'EOF'
// UDC-4067 Production Validation — Task Creation Test
Set<String> expectedRTs = new Set<String>{
  'VIP_Preference',
  'VIP_Training_Onboarding',
  'VIP_Case_Review',
  'New_Account_Training',
  'Existing_Account_Training',
  'Step_5_Case_Review',
  'Check_In',
  'Request_Training'
};

List<RecordType> taskRTs = [SELECT Id, Name, DeveloperName, IsActive 
                             FROM RecordType 
                             WHERE SObjectType = 'Task' 
                             AND DeveloperName IN :expectedRTs];

System.debug('=== UDC-4067 Production Validation ===');
System.debug('Record Types Found: ' + taskRTs.size() + ' / 8');

Integer successCount = 0;
for(RecordType rt : taskRTs) {
  try {
    Task t = new Task(
      Subject = 'UDC-4067 Production Validation - ' + rt.DeveloperName,
      RecordTypeId = rt.Id,
      ActivityDate = System.today()
    );
    insert t;
    System.debug('  ✓ ' + rt.DeveloperName);
    successCount++;
    delete t; // rollback
  } catch(Exception e) {
    System.debug('  ✗ ' + rt.DeveloperName + ': ' + e.getMessage());
  }
}

System.debug('Success Rate: ' + successCount + ' / ' + taskRTs.size());
System.debug('Status: ' + (successCount == 8 ? '✓ PASSED' : '✗ FAILED'));
System.debug('=====================================');
EOF

# Run validation in Production
sf apex run --target-org Prod --file /tmp/validate_udc4067_prod.apex
```

**Expected Output**:
```
✓ Check_In
✓ Existing_Account_Training
✓ New_Account_Training
✓ Request_Training
✓ Step_5_Case_Review
✓ VIP_Case_Review
✓ VIP_Preference
✓ VIP_Training_Onboarding
Success Rate: 8 / 8
Status: ✓ PASSED
```

### 3.2 Profile Access Validation (Manual)

**Test as System Administrator**:
1. Log into Production org
2. Create a new Task (any record)
3. In Record Type picklist → Verify all 8 new RTs visible
4. Select "VIP_Training_Onboarding" → Verify layout loads without errors
5. Verify new custom fields visible on layout (spot-check 3-4):
   - Training_Date__c
   - Communication_Preference__c
   - Case_Review_Date__c
6. Save the Task → Verify success (no validation errors)

**Test as Business User** (uLab Area Sales Directors):
1. Switch to user with "uLab Area Sales Directors" profile
2. Create a new Task
3. In Record Type picklist → Verify ONLY assigned RTs visible (not VIP_Preference, VIP_Training_Onboarding)
4. Select "Check_In" → Verify layout loads
5. Verify permitted fields are editable (per profile FLS)
6. Save the Task → Verify success

---

## Phase 4: Verification Summary (2 min)

### 4.1 Consolidate Results

Create summary report:

```bash
# Save validation results
cat > /tmp/udc4067_validation_summary.txt <<'EOF'
UDC-4067 Production Deployment Validation Summary
================================================

Component Checklist:
[ ] ✓ 20 custom fields present on Activity
[ ] ✓ 8 page layouts deployed on Task object
[ ] ✓ 11 record types present (8 active UDC-4067 + 3 inactive legacy)
[ ] ✓ 5 profiles present (Admin + 4 business)

Functional Tests:
[ ] ✓ Task creation successful with all 8 UDC-4067 record types
[ ] ✓ System Administrator can access all record types and layouts
[ ] ✓ Business user profile correctly restricts/permits record types
[ ] ✓ Page layouts render without errors
[ ] ✓ Custom fields accessible and editable per profile permissions

Validation Status: ✓ PASSED
Date: $(date)
Validator: [Your Name]

Next Step: Sign off with business and IT teams
EOF

# Display summary
cat /tmp/udc4067_validation_summary.txt
```

### 4.2 Final Checklist

- [ ] All metadata components verified in Production
- [ ] All 8 record types creatable via Task
- [ ] All profiles correctly configured
- [ ] No errors in debug logs
- [ ] System Administrator and business user access verified
- [ ] Rollback plan documented (if needed)
- [ ] Sign-off collected from QA and Business teams

---

## Troubleshooting

### Issue: "Record type not found"

**Symptom**: Query returns 0 record types or fewer than expected

**Resolution**:
```bash
# Check if legacy RTs still exist (they should)
sf data query --target-org Prod --query "SELECT COUNT() FROM RecordType WHERE SObjectType='Task'" --json

# If count < 11, check which RTs are missing
sf data query --target-org Prod --query "SELECT DeveloperName FROM RecordType WHERE SObjectType='Task'" --json
```

**Action**: If UDC-4067 RTs are missing, deployment may have failed. Check Setup → Deploy → Deployment Status for errors.

### Issue: "Task creation fails with 'Record type unavailable'"

**Symptom**: Apex test fails to create Task with new RT

**Resolution**:
```bash
# Verify record type is active
sf data query --target-org Prod --query "SELECT Id, IsActive FROM RecordType WHERE DeveloperName='VIP_Preference' LIMIT 1" --json

# If IsActive=false, activate via Setup → RecordTypes → Task → [RT] → Active
```

### Issue: "Profile not found"

**Symptom**: Query returns fewer than 5 profiles

**Resolution**:
```bash
# List ALL profiles (including hidden/deprecated ones)
sf data query --target-org Prod --query "SELECT Name FROM Profile LIMIT 500" --json | jq '.result.records[] | .Name' | grep -i ulab

# If profiles missing, check deployment status for errors
```

---

## Command Reference (Quick Copy-Paste)

```bash
# Validate fields
sf data query --target-org Prod --query "SELECT COUNT() FROM CustomField WHERE TableEnumOrId='Activity'" --json

# Validate record types
sf data query --target-org Prod --query "SELECT COUNT() FROM RecordType WHERE SObjectType='Task'" --json

# Validate profiles
sf data query --target-org Prod --query "SELECT COUNT() FROM Profile WHERE Name LIKE 'uLab%'" --json

# Run Apex validation
sf apex run --target-org Prod --file /tmp/validate_udc4067_prod.apex
```

---

## Success Metrics

✅ **Validation PASSED** when:
- [x] 20+ custom fields present on Activity
- [x] 8 page layouts deployed on Task
- [x] 11 record types present (8 active)
- [x] 5 profiles present in org
- [x] All 8 Tasks created successfully in Apex test
- [x] No errors in Production debug logs
- [x] Manual UI testing passed (System Admin + Business User)

✗ **Validation FAILED** if any of above criteria not met

---

**Validation Complete**: [Date/Time]  
**Validator Name**: ___________________  
**Sign-Off**: ✓ All criteria passed / ✗ Issues found (details below)

**Issues Found** (if applicable):
```
[Document any issues and resolution steps taken]
```

---

**Reference**: See `PRODUCTION-DEPLOYMENT-CHECKLIST.md` for full deployment context.
