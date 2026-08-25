# UDC-4067 Production Deployment — Quick Reference Guide

**Status**: ✅ Ready for Production  
**Feature Branch**: `feature/udc-4067-pr2-layouts-profiles`  
**Latest Commit**: `687078c`  
**Deployments Validated**: Onboarding (2 waves, 44/44 components)  
**Risk Level**: LOW (declarative-only, no code)

---

## One-Page Summary

| Item | Value |
|------|-------|
| **What's Being Deployed?** | 20 custom fields, 8 page layouts, 11 record types, 5 profiles |
| **Where?** | Production org (`https://ulab.my.salesforce.com`) |
| **When?** | [To be scheduled — recommended before next sprint] |
| **Who Executes?** | Deployment Lead (via CLI command below) |
| **Duration** | ~5-10 min deployment + 30 min validation = 45 min total |
| **Impact** | 4 business profiles get new Task layouts; System Admins get full access |
| **Risk** | LOW — metadata-only, no data changes, no code, fully reversible |
| **Rollback** | ~10 min if needed (org-wide snapshot or CLI revert) |

---

## Pre-Deployment Checklist (Before Deployment Window)

- [ ] **T-5 days**: Notify stakeholders via email + Slack
- [ ] **T-5 days**: Business team reviews 8 new record types (no conflicts with legacy RTs)
- [ ] **T-2 days**: Take org-wide snapshot of Production (Setup → Snapshots, if available)
- [ ] **T-2 days**: Run `sf project deploy start ... --dry-run` (validate metadata, 0 cost)
- [ ] **T-0 hours**: Disable Task-related automations (Flows, etc.) — optional but safe
- [ ] **T-0 hours**: Open Slack incident channel + assign rollback lead as backup

---

## Deployment Command (Copy-Paste Ready)

```bash
# Change to repo directory
cd /Users/albertocordero/Documents/VSC/UDC-4067\ Enhance\ Tasks

# OPTIONAL: Dry-run first (no metadata changes, just validation)
sf project deploy start \
  --target-org Prod \
  --manifest manifest/package.xml \
  --test-level RunLocalTests \
  --dry-run \
  --wait 30

# PRODUCTION DEPLOYMENT (execute if dry-run succeeds)
sf project deploy start \
  --target-org Prod \
  --manifest manifest/package.xml \
  --test-level RunLocalTests \
  --wait 60

# Save Deploy ID from output (format: 0Af...)
```

---

## Expected Output

```
✔ Preparing 290ms
✔ Deploying Metadata 8-15s
   ▸ Components: 44/44 (100%)
✔ Running Tests [if enabled]
✔ Updating Source Tracking
✔ Done

Status: Succeeded
Deploy ID: 0Af1A00000abc123XYZ    ← SAVE THIS
Target Org: [email protected]
Elapsed Time: 45s
```

---

## Immediate Post-Deployment (T+5 min)

1. **Verify in Setup**:
   - Setup → Deploy → Deployment Status → search Deploy ID → confirm **Succeeded**
   - Setup → CustomFields → Activity → count fields (should see 20 new ones)
   - Setup → PageLayouts → Task → verify 8 layouts present
   - Setup → RecordTypes → Task → verify 11 RTs (8 active + 3 inactive pre-existing)

2. **Test Task Creation**:
   - Create a Task → Select one of 8 new RTs → Save → Verify layout renders ✅

3. **Re-Enable Automations**:
   - Reactivate any Task flows/processes disabled before deployment

---

## Validation Tests (T+30 min)

**Run Apex validation**:
```bash
sf apex run --target-org Prod --file /tmp/final_validation.apex
```

**Expected Results**:
- ✅ "Sample Record Types Found: 8"
- ✅ "Task created successfully with RT: Request_Training"
- ✅ "Status: ✓ READY FOR PRODUCTION"

**Test with Business User**:
- Log in as "uLab Area Sales Directors" user
- Create Task → Select Check_In RT → Save → Verify layout + fields ✅

---

## Rollback (If Critical Issue)

**Option A** (Preferred, if org-wide snapshot available):
1. Setup → Snapshots → find pre-deployment snapshot
2. Click "Restore" → confirm
3. Wait ~10-15 min for restore to complete

**Option B** (CLI revert):
```bash
# Revert manifest to pre-deployment state
git checkout HEAD~1 manifest/package.xml force-app/main/default/

# Redeploy old state
sf project deploy start --target-org Prod --manifest manifest/package.xml --wait 30
```

---

## Components Matrix

### Profile Layout Assignments

| Profile | Check_In | Request_Training | VIP_Case_Review | VIP_Preference | VIP_Training | Case_Review | Existing_Account | New_Account |
|---------|----------|------------------|-----------------|----------------|--------------|-------------|------------------|-------------|
| System Administrator | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Area Sales Directors | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Clinical Training & Dev | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Sales Manager | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| uAssist Lead & Sr. Trainer | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ |

*(✓ = Visible + Layout Assigned | ✗ = Hidden from RT picklist)*

---

## Custom Fields (20 Total)

Deployed on Activity object (not Task directly, due to CompSuite managed package). Fully accessible via Task layouts.

**Training Fields**:
- Training_Date__c, Training_Method__c, Training_Type__c, Training_Goals__c

**Preference Fields**:
- Communication_Preference__c, Engagement_Type__c, Scheduling_Options__c, Pricing_Tier__c

**Date/Method Fields**:
- Case_Review_Date__c, Case_Review_Method__c, Date_Received__c, Date_Sent__c

**Other Fields**:
- Additional_Account_Notes__c, Addtional_Feature__c, EasyRx__c, Printer__c, Scanner__c, Software_Version__c, Software_Version_Number__c, Date_Uploaded_to_SFDC__c

---

## Support & Escalation

**During Deployment**:
- Issue? → Contact Deployment Lead immediately
- Can't proceed? → Prepare to rollback (see rollback section)

**Post-Deployment**:
- User question? → Refer to `/docs/UDC-4067-user-story.md`
- Bug found? → Document in JIRA + notify architect
- Profile access issue? → Check Setup → Profiles → Profile → Object Settings → Task

---

## Sign-Off

| Role | Sign-Off | Date |
|------|----------|------|
| Business Analyst | ☐ Reviewed, approved | _____ |
| QA Lead | ☐ Testing plan ready | _____ |
| IT/Security | ☐ Security review passed | _____ |
| Deployment Lead | ☐ Ready to deploy | _____ |

---

**For Full Details**: See `PRODUCTION-DEPLOYMENT-CHECKLIST.md`
