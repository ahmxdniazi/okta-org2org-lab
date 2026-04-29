# ✅ Implementation Checklist

Complete walkthrough of the Okta Org2Org lab — use this as a reference to replicate the setup.

## Phase 1: Custom Attributes

- [x] Navigate to **Directory → Profile Editor → User (default)**
- [x] Click **Add Attribute**
- [x] Create `workingHours` (string, Custom)
- [x] Create `employeeId` (string, Custom)
- [x] Create `departmentCode` (string, Custom)
- [x] Create `tenantRegion` (string, Custom)
- [x] Verify all 4 custom attributes appear in the attribute list alongside base attributes

## Phase 2: Groups

- [x] Navigate to **Directory → Groups**
- [x] Click **Add Group**
- [x] Create `SourceOrg-Engineering` — no description needed
- [x] Create `SourceOrg-Marketing` — no description needed
- [x] Create `TargetOrg-Engineering` — no description needed
- [x] Create `TargetOrg-Marketing` — no description needed
- [x] Verify all 4 groups appear in the Groups list

## Phase 3: Test Users

- [x] Navigate to **Directory → People**
- [x] Create or identify test users with `@cyber.com` domain
- [x] Assign **Sajid Niazi**, **Ayesha Sajid**, **Sarmad Farooq** → `SourceOrg-Engineering`
- [x] Assign **Hamza Khan**, **Ayesha Sajid** → `SourceOrg-Marketing`
- [x] Verify user counts: Engineering = 3, Marketing = 2

## Phase 4: Group Rules

- [x] Navigate to **Directory → Groups → Rules tab**
- [x] Click **Add Rule**
- [x] **Rule 1:** Name = "Push Engineering to Target"
  - IF: User attribute | login | Equals | `SourceOrg-Engineering`
  - THEN: Assign to `TargetOrg-Engineering`
- [x] **Rule 2:** Name = "Push Marketing to Target"
  - IF: User attribute | login | Equals | `SourceOrg-Marketing`
  - THEN: Assign to `TargetOrg-Marketing`
- [x] Activate both rules
- [x] Verify both rules show **Active** status

## Phase 5: Verification

- [x] Open `TargetOrg-Engineering` → confirm 3 users appear (managed by rule)
- [x] Open `TargetOrg-Marketing` → confirm 2 users appear (managed by rule)
- [x] Open a user profile → Groups tab → confirm they show in TargetOrg group with "Managed by Push Engineering to Target"
- [x] ✅ **Lab Complete!**

---

## Quick Reference: User → Group Mapping

```
Sarmad Farooq  ──→ SourceOrg-Engineering ──→ TargetOrg-Engineering ✓
Ayesha Sajid   ──→ SourceOrg-Engineering ──→ TargetOrg-Engineering ✓
               ──→ SourceOrg-Marketing   ──→ TargetOrg-Marketing   ✓
Sajid Niazi    ──→ SourceOrg-Engineering ──→ TargetOrg-Engineering ✓
Hamza Khan     ──→ SourceOrg-Marketing   ──→ TargetOrg-Marketing   ✓
```
