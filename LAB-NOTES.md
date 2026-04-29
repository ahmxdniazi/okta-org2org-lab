# 🧠 Lab Notes & Learnings

## Okta Org2Org — Implementation Notes

**Date:** April 29, 2026  
**Environment:** Okta Developer Org (Trial)  
**Domain:** `@cyber.com`

---

## What I Learned

### 1. Custom Attributes in Universal Directory
- Okta's **Profile Editor** allows adding custom attributes beyond defaults (email, firstName, lastName, etc.)
- Attributes have data types: `string`, `boolean`, `integer`, `number`, `array`
- Variable naming convention: camelCase (e.g., `departmentCode`, `tenantRegion`)
- The "Enum" option restricts values to a predefined list — useful for standardizing department codes
- The "Restriction" option (`Value must be unique per user`) enforces uniqueness — ideal for `employeeId`

### 2. Group Architecture for Org2Org
- The **naming convention matters** — I used `SourceOrg-*` and `TargetOrg-*` prefix pattern
- Source groups = **manually managed** (admin assigns users)
- Target groups = **rule-managed** (system auto-assigns)
- Having clear naming prevents confusion when users appear in multiple groups

### 3. Group Rules — The Core Logic
- Rules follow **IF → THEN → EXCEPT** structure
- Conditions can use:
  - `User attribute` (e.g., `login`, `email`, `departmentCode`)
  - Operators: `Equals`, `Contains`, `Starts with`, `Matches regex`
  - **Okta Expression Language (OEL)** for advanced conditions
- In this lab, the condition was: `user.login Equals "SourceOrg-Engineering"`
  > *Note: This conditions on login value matching — in a real Org2Org, you'd more likely condition on group membership via `isMemberOfGroupName()` expression*
- Rules are **evaluated in order** — important when users could match multiple rules

### 4. EXCEPT clause
- The EXCEPT clause lets you manually exclude specific users from a rule
- Useful for edge cases: e.g., a contractor who is in the Engineering group but should NOT be pushed to the target org

### 5. "This rule will not add users to a group they've been manually removed from"
- This is a **safety mechanism** in Okta — prevents infinite loops where an admin removes a user but the rule keeps adding them back
- To force re-add, you'd need to remove the exception manually

### 6. Pending vs Active Status
- `Active` — User has activated their Okta account (clicked email link)
- `Pending user action` — User was created/invited but hasn't activated yet
- Group rules still apply to Pending users — they'll be in the group when they activate

---

## Gotchas & Observations

| Gotcha | Explanation |
|---|---|
| Rule condition matches `login`, not group membership | In production, use `isMemberOfGroupName()` OEL expression for true group-based conditions |
| Dual-group users (Ayesha Sajid) appear in both target groups | Users can satisfy multiple rules simultaneously — this is expected behavior |
| TargetOrg-Marketing showed 0 users initially | Rules only run automatically on save/activate or on a schedule — triggering manually via "Run Rule" ensures immediate population |
| `Pending user action` users still get assigned | Group assignment is separate from account activation — users can be "in" a group before they've logged in |

---

## Architecture Decisions

**Why separate Source and Target groups instead of one set?**
- Clear separation of concerns: Source = "who should have access", Target = "who currently has access"
- Allows rollback: if you remove a rule, target group empties but source group is untouched
- Auditing: you can compare Source vs Target to spot provisioning gaps

**Why custom attributes?**
- `departmentCode` allows programmatic, API-driven group assignment (not just login-based)
- `tenantRegion` enables geo-specific provisioning in multi-region orgs
- These attributes can be **mapped and pushed** to the Target Org via Org2Org provisioning — so the target gets the metadata too

---

## Real-World Extensions

In a production Org2Org setup, you would also:

1. **Configure the Org2Org app** — Install Okta's Org2Org application, provide the target org's API token
2. **Set up attribute mapping** — Map source user attributes to target org's schema
3. **Enable Group Push** — Push entire groups (not just individual users) to the target org
4. **Configure Profile Push** — Sync profile attribute changes in real-time (e.g., if `departmentCode` changes)
5. **Set deprovisioning rules** — What happens when a user leaves the source org? Deactivate, suspend, or delete in target?
6. **Lifecycle management** — Reactivation, profile sync on update, etc.

---

## Commands / API Reference

```bash
# Okta Management API — List groups
GET /api/v1/groups

# Get group members
GET /api/v1/groups/{groupId}/users

# List group rules
GET /api/v1/groups/rules

# Create a group rule
POST /api/v1/groups/rules
{
  "type": "group_rule",
  "name": "Push Engineering to Target",
  "conditions": {
    "expression": {
      "value": "user.login==\"SourceOrg-Engineering\"",
      "type": "urn:okta:expression:1.0"
    }
  },
  "actions": {
    "assignUserToGroups": {
      "groupIds": ["<TargetOrg-Engineering-GroupId>"]
    }
  }
}

# Activate a rule
POST /api/v1/groups/rules/{ruleId}/lifecycle/activate
```

---

## References

- [Okta Org2Org Documentation](https://help.okta.com/en-us/content/topics/provisioning/org2org/org2org-main.htm)
- [Okta Group Rules](https://help.okta.com/en-us/content/topics/users-groups-profiles/usgp-create-group-rules.htm)
- [Okta Expression Language](https://developer.okta.com/docs/reference/okta-expression-language/)
- [Okta Groups API](https://developer.okta.com/docs/reference/api/groups/)
- [Universal Directory Overview](https://help.okta.com/en-us/content/topics/directory/eu-about-universal-directory.htm)
