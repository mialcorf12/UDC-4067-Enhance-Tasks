# UDC-4067 Production Deployment Checklist

**Project**: UDC-4067 — Update and Enhance Tasks in SFDC  
**Target Org**: Production (`https://ulab.my.salesforce.com`)  
**Feature Branch**: `feature/udc-4067-pr2-layouts-profiles`  
**Commit**: `687078c` (post-deployment validation complete)  
**Deployment Type**: Metadata + Configuration (no code/data migration)  
**Risk Level**: **LOW** ✅ (declarative-only, no Apex, no data modifications)  

---

## Pre-Deployment Phase (T-14 to T-5)

### 1. Stakeholder Alignment

- [ ] **Communications Manager**: Notify all users in affected profiles (4 business profiles + System Admins)
  - Affected Profiles: 
    - uLab Area Sales Directors
    - uLab Clinical Training & Development
    - uLab Sales Manager
    - uLab uAssist Lead & Sr. Trainer
  - Message: "New Task record types and page layouts deploying on [DATE]. No action required; layouts auto-apply per role."
  
- [ ] **Business Analysts**: Review 8 new record types with business owners
  - RTs: VIP_Preference, VIP_Training_Onboarding, VIP_Case_Review, New_Account_Training, Existing_Account_Training, Step_5_Case_Review, Check_In, Request_Training
  - Confirm: No naming conflicts with existing legacy RTs (13 pre-existing RTs remain untouched)
  
- [ ] **IT/Security**: Review permission implications
  - Profiles only modified for layout assignments + record type visibility (no permission changes)
  - System Administrator profile newly created with standard permissions
  - No change to FLS, CRUD, or custom permissions

- [ ] **Documentation Team**: Publish user guides
  - Document the 8 new record types (use cases, mandatory fields, workflows)
  - Update admin documentation for profile layout assignments
  - Provide screenshots of Task page layouts with new fields

### 2. Technical Pre-Flight

- [ ] **Source Control**: Verify branch status
  - Branch: `feature/udc-4067-pr2-layouts-profiles` is up-to-date with `main`
  - Latest commit: `687078c` (post-deployment validation report)
  - All validation reports present in `/docs/validations/`

- [ ] **Metadata Validation**: Confirm all components are deployable
  - ✅ 20 custom fields (Activity object) — validated in Onboarding
  - ✅ 8 page layouts (Task object) — validated in Onboarding
  - ✅ 11 record types (Task) — 8 active, 3 inactive — validated in Onboarding
  - ✅ 5 profiles (Admin + 4 business) — validated in Onboarding, filenames corrected
  - ✅ manifest/package.xml includes all components, no conflicts

- [ ] **Backup Strategy**: Prepare rollback plan
  - [ ] Take org-wide snapshot of Production before deployment (Setup → Snapshots, if available)
  - [ ] Export current Task layouts via sf metadata retrieve (fallback recovery)
  - [ ] Export current profile configurations (for quick rollback)
  - [ ] Document rollback steps (see Rollback section below)

- [ ] **Org Health Check**: Validate Production org readiness
  - [ ] Run `sf project deploy status --target-org Prod` → verify no in-flight deployments
  - [ ] Check org storage (Setup → System Overview) → ensure >10% free space
  - [ ] Verify no active Apex jobs or batch processes (Setup → Apex Jobs)
  - [ ] Confirm no active Change Data Capture subscriptions that depend on Task field changes
  - [ ] Validate API usage is below rate limits (Setup → System Overview)

- [ ] **Test Execution in Staging** (optional but recommended)
  - [ ] Deploy same metadata to Staging sandbox (use same manifest/package.xml)
  - [ ] Run same post-deployment validation tests (Apex DML, SOQL queries)
  - [ ] Have business team create sample Tasks in Staging with new RTs (UAT)
  - [ ] Confirm page layouts render correctly and new fields are editable

---

## Deployment Window (T-0)

### 3. Pre-Deployment Tasks (T-2 hours before)

- [ ] **Notify Stakeholders**: Send final notification
  - Announce: "Deployment window: [TIME] - [TIME+30 min]"
  - Impact: "Task layouts will be updated for 4 user profiles; no interruption expected"
  - Contact: Provide deployment lead contact for issues

- [ ] **Disable Automations** (if applicable)
  - [ ] Temporarily disable any Task-related automations (Flows, Process Builder, Apex triggers)
    - Reason: Prevent interference during layout/RT metadata changes
    - Duration: During deployment only (~5-10 minutes)
  - [ ] Document which automations were disabled (for re-enable step)

- [ ] **Prepare Deployment Command**
  - Verify target org alias: `sf org list --all` → confirm `Prod` is listed
  - Validate manifest: `cat manifest/package.xml` → confirm all 5 profiles + 20 fields + 8 layouts + 11 RTs present
  - Dry-run test (optional): `sf project deploy start --target-org Prod --manifest manifest/package.xml --test-level RunLocalTests --dry-run`

- [ ] **Deployment Team Readiness**
  - [ ] Deployment lead confirms SSH/VPN access to production systems
  - [ ] Slack/incident channel open and monitored
  - [ ] Rollback lead standing by with snapshot/rollback scripts
  - [ ] Backup of org-wide snapshot stored in secure location (S3 bucket or org snapshot)

### 4. Deployment Execution (T-0)

**Deployment Command**:
```bash
# PRODUCTION DEPLOYMENT — UDC-4067
cd /path/to/UDC-4067\ Enhance\ Tasks

# Validate deployment first (no-op, shows what would deploy)
sf project deploy start \
  --target-org Prod \
  --manifest manifest/package.xml \
  --test-level RunLocalTests \
  --dry-run \
  --wait 30

# If dry-run succeeds, execute real deployment
sf project deploy start \
  --target-org Prod \
  --manifest manifest/package.xml \
  --test-level RunLocalTests \
  --wait 60

# Capture deploy ID from output (format: 0Af...)
# Example: Deploy ID: 0Af1A00000abc123XYZ
```

**Success Criteria**:
- [ ] Deploy ID issued (confirm in output: "Deploy ID: 0Af...")
- [ ] Status: **Succeeded** (NOT Succeeded with warnings, NOT Failed, NOT Canceled)
- [ ] Component count: **44/44 components deployed** (20 fields + 8 layouts + 11 RTs + 5 profiles)
- [ ] Test result: **All tests passed** (if RunLocalTests was enabled)
- [ ] Duration: <2 minutes (metadata deployments typically finish in 30-60 seconds)

**What to Expect During Deployment**:
- 0-10 sec: Preparing metadata
- 10-30 sec: Deploying to Production (progress bar: 0% → 100%)
- 30-45 sec: Running tests (if enabled) or completing
- 45-60 sec: Updating source tracking
- Final output: "Status: Succeeded" + Deploy ID

**Critical Actions During Deployment**:
- ⚠️ **DO NOT**: Close terminal, cancel Ctrl+C, or refresh CLI
- ⚠️ **DO NOT**: Start another deploy while this one is in progress
- ⚠️ **DO NOT**: Make manual changes in Setup while deploying
- ✅ **DO**: Monitor terminal for progress
- ✅ **DO**: Save Deploy ID immediately (needed for verification + potential rollback)

---

## Post-Deployment Phase (T+5 min to T+60 min)

### 5. Immediate Verification (T+5 min)

- [ ] **Deployment Status Confirmation**
  - [ ] Capture Deploy ID from terminal output: `________________`
  - [ ] Log into Production org
  - [ ] Setup → Deploy → Deployment Status → search for Deploy ID
  - [ ] Confirm status: **Succeeded** with 0 errors, 0 warnings
  - [ ] Note timestamp of deployment completion

- [ ] **Component Verification** (Metadata Level)
  - [ ] Setup → CustomFields → Activity → count custom fields
    - Expected: 20 new fields created/updated (Training_Date__c, Training_Method__c, etc.)
  - [ ] Setup → PageLayouts → Task → list all layouts
    - Expected: 8 layouts present (Task-VIP Training_Onboarding, Task-Check in, etc.)
  - [ ] Setup → RecordTypes → Task → list all RTs
    - Expected: 11 total (8 active UDC-4067 RTs + 3 inactive pre-existing RTs)
  - [ ] Setup → Profiles → verify each profile has Task layout assignments
    - Expected: Admin profile + 4 business profiles updated with 8 layouts each

- [ ] **Profile Layout Assignments** (UI Verification)
  - [ ] Go to Setup → Profiles → "uLab Area Sales Directors"
    - Scroll to Record Type Picklist Values → Check_In should be **Visible** ✅
    - Scroll to Object Settings → Task → Record Type Tab Settings → confirm 8 assignments
  - [ ] Repeat for: "uLab Clinical Training & Development", "uLab Sales Manager", "uLab uAssist Lead & Sr. Trainer"
  - [ ] Go to Setup → Profiles → "System Administrator"
    - Verify all 8 Task layouts assigned with record type visibility

- [ ] **Re-Enable Automations**
  - [ ] Restore any Task automations disabled before deployment
  - [ ] Confirm Flows/Process Builder status: **Active** ✅

### 6. Functional Testing (T+10 to T+30 min)

**Test with System Administrator profile**:
1. [ ] Create a new Task (Setup → Objects and Fields → Accounts/Contacts → Task)
2. [ ] In Subject field, note the Task page layout name (should match one of 8 new layouts)
3. [ ] Verify that new custom fields are visible on the page layout:
   - [ ] Training_Date__c field present
   - [ ] Communication_Preference__c field present
   - [ ] Case_Review_Date__c field present
   - [ ] (Spot-check 3-4 fields; all 20 should be accessible)
4. [ ] Select each of the 8 new record types from the Record Type picklist:
   - [ ] VIP_Preference
   - [ ] VIP_Training_Onboarding
   - [ ] VIP_Case_Review
   - [ ] New_Account_Training
   - [ ] Existing_Account_Training
   - [ ] Step_5_Case_Review
   - [ ] Check_In
   - [ ] Request_Training
   - Verify: Layout changes per RT, no errors, fields remain visible
5. [ ] Save a sample Task (do NOT delete; leave for audit trail)
6. [ ] Verify Task record created successfully in list view with correct RT

**Test with Business User profile** (e.g., "uLab Area Sales Directors"):
1. [ ] Log out of admin; log in as test user with "uLab Area Sales Directors" profile
2. [ ] Create a new Task
3. [ ] Verify: Only **visible** record types appear in RT picklist (per profile assignment)
   - Expected visible for Sales Directors: Check_In, Request_Training, VIP_Case_Review (plus others assigned to this profile)
4. [ ] Select one assigned RT; verify layout loads correctly
5. [ ] Verify: Fields have proper permissions per profile (editable if FLS allows, read-only otherwise)
6. [ ] Try to access a **hidden** RT (should NOT appear in picklist)

### 7. Post-Deployment Validation (T+30 to T+60 min)

- [ ] **Run Apex Validation Script** (same as Onboarding validation)
  ```bash
  sf apex run --target-org Prod --file /path/to/final_validation.apex
  ```
  Expected output:
  - [ ] "Sample Record Types Found: 8" (all UDC-4067 RTs present)
  - [ ] "Profiles in Production: [count]" (at least 5 profiles listed: Admin + 4 business)
  - [ ] "Task created successfully with RT: Request_Training" ✅
  - [ ] "Task verified in database" ✅
  - [ ] "Cleanup completed (rollback)" ✅
  - [ ] **Status: ✓ READY FOR PRODUCTION**

- [ ] **Check Error Logs**
  - [ ] Setup → Debug Logs → verify no ERROR or FATAL logs related to Task object
  - [ ] Logs should only contain INFO/DEBUG entries (if any)

- [ ] **Monitoring Alerting** (if available)
  - [ ] Check monitoring dashboards for org health (CPU, API limits, storage)
  - [ ] Expected: All metrics normal; no spikes post-deployment
  - [ ] No Apex governor limit violations in debug logs

- [ ] **Team Sign-Off**
  - [ ] Business Analyst: "Layouts and record types verified in Production" ✅
  - [ ] QA Lead: "Functional testing passed; no blockers" ✅
  - [ ] IT/Security: "No security or compliance issues detected" ✅
  - [ ] Deployment Lead: "Deployment successful; ready for user communication" ✅

---

## Post-Deployment Communication (T+60 min)

### 8. User Notification

- [ ] **Send Go-Live Announcement**
  - To: All users in affected profiles (send via email + Chatter)
  - Subject: "Task Enhancements Now Live in Production"
  - Content:
    ```
    Hi Team,

    UDC-4067 Task enhancements have been successfully deployed to Production.

    What's New:
    - 8 new Task record types for specialized workflows
    - Enhanced Task layouts with new fields (Training Date, Communication Preference, etc.)
    - Automated layout assignment based on your role

    No action required. Your Task page layouts and record type options have been automatically updated based on your profile.

    Questions? Contact: [IT Contact]
    Documentation: [Link to User Guide]

    Deployment Details:
    - Deploy ID: [Insert from Step 5]
    - Deployment Time: [Insert timestamp]
    ```

- [ ] **Publish Documentation**
  - [ ] Post user guide in Confluence/SharePoint for the 8 new record types
  - [ ] Add Task tips to company wiki/knowledge base
  - [ ] Update admin documentation with layout assignment matrix

### 9. Post-Deployment Monitoring (T+1 to T+7 days)

- [ ] **Daily Monitoring** (first 7 days)
  - [ ] Check Setup → Deploy → Deployment Status for any errors reported post-deployment
  - [ ] Monitor Apex Jobs for any failures related to Task
  - [ ] Review user feedback (Slack, email) for layout/RT issues
  - [ ] Confirm no spike in Task creation errors

- [ ] **User Adoption Tracking** (first 2 weeks)
  - [ ] Verify users are creating Tasks with new RTs
  - [ ] Check Task count/distribution across new RTs (should see usage if ROI expected)
  - [ ] Gather feedback: Are fields visible? Are layouts clear? Any confusion?

- [ ] **Rollback Readiness** (first 24 hours only)
  - [ ] Keep org-wide snapshot and rollback scripts on standby
  - [ ] If critical issue discovered within 24 hours, execute rollback (see below)
  - [ ] After 24 hours with no issues, archive rollback materials

---

## Rollback Plan (If Needed)

**Triggers for Rollback**:
- ❌ Deploy failed with errors (unlikely if dry-run passed)
- ❌ Critical layout rendering issue preventing Task creation
- ❌ Profiles/FLS mismatch blocking user access
- ❌ Cascading failures in dependent processes (flows, integrations)

**Rollback Steps** (execute only if critical issue confirmed):

1. **Immediate Communication**
   ```
   Slack: "Rolling back UDC-4067 due to [SPECIFIC ISSUE]"
   Email: Notify all affected users that layouts will revert
   ```

2. **Execute Rollback**
   ```bash
   # Option A: Use org-wide snapshot (preferred if available)
   # Restore from snapshot in Setup → Snapshots
   # (Admin user navigates UI; ~10-15 min to restore)

   # Option B: Retrieve and revert via CLI (if snapshot unavailable)
   cd /path/to/repo
   git checkout HEAD~1 manifest/package.xml force-app/main/default/profiles/ force-app/main/default/layouts/
   sf project deploy start --target-org Prod --manifest manifest/package.xml --wait 30
   ```

3. **Verification**
   - [ ] Confirm layouts reverted to pre-deployment state
   - [ ] Confirm profiles returned to original assignments
   - [ ] Test Task creation with legacy record types only

4. **Post-Rollback**
   - [ ] Document issue in JIRA with "rollback" tag
   - [ ] Schedule retro/root cause analysis
   - [ ] Plan re-deployment after fix (if applicable)

---

## Deployment Timeline Estimate

| Phase | Duration | Owner |
|-------|----------|-------|
| Pre-Deployment (T-14 to T-5) | 2-3 days | Business + Tech Team |
| Technical Pre-Flight (T-5 to T-2) | 1-2 days | DevOps/Architect |
| Deployment Window Prep (T-2 hours) | 2 hours | Deployment Lead |
| Deployment Execution | 5-10 min | Deployment Lead |
| Immediate Verification (T+5 min) | 10-15 min | QA + Architect |
| Functional Testing (T+10 min) | 15-20 min | Business Analyst + QA |
| Post-Deployment Validation (T+30 min) | 15-20 min | Tech Team |
| Monitoring + Comms (T+60 min onwards) | 1 week | All Teams |

**Total Deployment Window**: 30-45 minutes (T-0 to T+45 min)  
**Total Pre-Deployment Work**: 2-3 days

---

## Appendix A: Component Inventory

**Components Being Deployed to Production:**

| Type | Count | Details |
|------|-------|---------|
| **Custom Fields** | 20 | Additional_Account_Notes__c, Addtional_Feature__c, Case_Review_Date__c, Case_Review_Method__c, Communication_Preference__c, Date_Received__c, Date_Sent__c, Date_Uploaded_to_SFDC__c, EasyRx__c, Engagement_Type__c, Pricing_Tier__c, Printer__c, Scanner__c, Scheduling_Options__c, Software_Version_Number__c, Software_Version__c, Training_Date__c, Training_Goals__c, Training_Method__c, Training_Type__c |
| **Page Layouts** | 8 | Task-Case Review, Task-Check in, Task-Existing Account Training, Task-New Account Training, Task-Request Training, Task-VIP Case Review, Task-VIP Preference, Task-VIP Training_Onboarding |
| **Record Types** | 11 | VIP_Preference, VIP_Training_Onboarding, VIP_Case_Review, New_Account_Training, Existing_Account_Training, Step_5_Case_Review, Check_In, Request_Training (active); Advanced_Case_Review, Step_6_Advanced_Training, Compass_Tool_Activity (inactive) |
| **Profiles** | 5 | Admin, uLab Area Sales Directors, uLab Clinical Training & Development, uLab Sales Manager, uLab uAssist Lead & Sr. Trainer |
| **TOTAL** | **44** | All components validated in Onboarding sandbox |

---

## Appendix B: Contacts & Escalation

| Role | Name | Email | Phone |
|------|------|-------|-------|
| Deployment Lead | [Name] | [Email] | [Phone] |
| Rollback Lead | [Name] | [Email] | [Phone] |
| Business Owner | [Name] | [Email] | [Phone] |
| IT Manager | [Name] | [Email] | [Phone] |
| Salesforce Architect | [Name] | [Email] | [Phone] |

---

## Appendix C: Success Metrics

**Deployment Success Metrics:**
- ✅ Deploy ID issued without errors
- ✅ 44/44 components deployed successfully
- ✅ 0 errors, 0 warnings in deployment summary
- ✅ All 8 UDC-4067 record types visible in Production Task object
- ✅ All 5 profiles updated with correct layout assignments
- ✅ Apex validation tests pass in Production
- ✅ Business users can create Tasks with new RTs and layouts
- ✅ No post-deployment errors in debug logs
- ✅ Monitoring dashboards show normal org health

**Success Criteria Met**: ✅ Ready for User Adoption

---

**Checklist Version**: 1.0  
**Created**: 2026-08-25  
**Last Updated**: 2026-08-25  
**Prepared by**: Salesforce Architect  
**Approved by**: [Pending - to be filled before deployment]  
**Final Sign-Off**: [Pending - to be filled after deployment]

---

**Next Step**: Print this checklist, assign owners to each section, and schedule deployment window.
