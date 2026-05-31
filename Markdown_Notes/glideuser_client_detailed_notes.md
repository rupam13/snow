# ServiceNow Client-Side GlideUser (g_user): Complete Reference & 45 Code Examples

This reference manual provides an exhaustive breakdown of the client-side `g_user` API. For every property and method, you will find its **meaning**, **when to use it**, and **exactly 5 real-world practical ServiceNow code examples with individual explanations**.

---

## 1. Properties (Static Values)

---

### A. `g_user.firstName`
* **Meaning:** Retrieves the first name string of the logged-in user from the `sys_user` table.

#### 1. Welcome Banner Greeting
* **Explanation:** Displays a personalized greeting banner at the top of the form on load.
```javascript
function onLoad() {
    g_form.addInfoMessage("Welcome back, " + g_user.firstName + "! Please verify assignments.");
}
```

#### 2. Auto-Populate Description Intro
* **Explanation:** Pre-fills the description field on a new record prompting the user.
```javascript
function onLoad() {
    if (g_form.isNewRecord()) {
        g_form.setValue('description', "Hi " + g_user.firstName + ",\nDetail your request here: ");
    }
}
```

#### 3. Inline Field Validation Message
* **Explanation:** Prompts the user to update critical contact info with an inline message.
```javascript
function onLoad() {
    g_form.showFieldMsg('caller_id', "Hello " + g_user.firstName + ", verify your phone details.", 'info');
}
```

#### 4. Developer Client Logging
* **Explanation:** Writes a trace log to the browser console when the script runs.
```javascript
function onLoad() {
    jslog("Page loaded by user: " + g_user.firstName);
}
```

#### 5. Abort Warning Banner
* **Explanation:** Displays an error banner asking a user to update details.
```javascript
function onSubmit() {
    g_form.clearMessages();
    jslog("Submission checked for " + g_user.firstName);
    return true;
}
```

---

### B. `g_user.lastName`
* **Meaning:** Retrieves the last name string of the logged-in user.

#### 1. Audit Text Note Builder
* **Explanation:** Inserts a sign-off stamp containing the user's last name into a text field.
```javascript
function onSubmit() {
    var desc = g_form.getValue('description');
    g_form.setValue('description', desc + "\n[Validated by employee: " + g_user.lastName + "]");
    return true;
}
```

#### 2. Dynamic Field Help Display
* **Explanation:** Shows last name context dynamically under a reference field.
```javascript
function onLoad() {
    g_form.showFieldMsg('u_reviewer', "Signed off by family name: " + g_user.lastName, 'info');
}
```

#### 3. Log Updates Trace
* **Explanation:** Write trace messages tracking changes inside user sessions.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    jslog("Value changed. Updated by user last name: " + g_user.lastName);
}
```

#### 4. Pre-fill Escalation Summary
* **Explanation:** Templates the Short Description to specify the reporting officer.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    if (newValue === '1') { // High priority
        g_form.setValue('short_description', "Escalation initiated by: " + g_user.lastName);
    }
}
```

#### 5. Verify Formal User Session
* **Explanation:** Confirms session data loaded matches valid fields.
```javascript
function onLoad() {
    if (!g_user.lastName) {
        g_form.addErrorMessage("Session Error: Logged-in profile lacks a family name.");
    }
}
```

---

### C. `g_user.userID`
* **Meaning:** The 32-character database `sys_id` of the logged-in user.

#### 1. Self-Assignment Check
* **Explanation:** Displays an informational banner when the user assigns a ticket to themselves.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    if (newValue === g_user.userID) {
        g_form.showFieldMsg('assigned_to', 'Assigned to yourself.', 'info');
    }
}
```

#### 2. Lock Record for Non-Owners
* **Explanation:** Locks down the Description field if the logged-in user is not the assignee.
```javascript
function onLoad() {
    var assignee = g_form.getValue('assigned_to');
    if (assignee !== "" && assignee !== g_user.userID) {
        g_form.setReadOnly('description', true);
    }
}
```

#### 3. Caller Warning Banner
* **Explanation:** Warns users not to log Critical (P1) incidents on their own behalf.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    if (newValue === g_user.userID && g_form.getValue('priority') === '1') {
        g_form.addErrorMessage("Warning: You are logging a Critical ticket for yourself.");
    }
}
```

#### 4. Auto-populate Requestor
* **Explanation:** Fills the caller field with the current user's ID on new forms.
```javascript
function onLoad() {
    if (g_form.isNewRecord()) {
        g_form.setValue('caller_id', g_user.userID);
    }
}
```

#### 5. Abort Self-Approvals
* **Explanation:** onSubmit validation blocking users from self-approving changes.
```javascript
function onSubmit() {
    var approvalAssigned = g_form.getValue('u_approver');
    if (approvalAssigned === g_user.userID) {
        g_form.addErrorMessage("You cannot be the approver on your own ticket!");
        return false;
    }
    return true;
}
```

---

### D. `g_user.userName`
* **Meaning:** The login User ID (e.g. `admin`).

#### 1. Admin System Warning
* **Explanation:** Banners a warning if a System Administrator logs into a production form.
```javascript
function onLoad() {
    if (g_user.userName === 'admin') {
        g_form.addErrorMessage("SYSTEM WARNING: Admin session active. Take care on updates.");
    }
}
```

#### 2. Lock Forms for Integration Accounts
* **Explanation:** Makes all fields read-only if logged-in user is a generic integration account.
```javascript
function onLoad() {
    if (g_user.userName.startsWith('int_')) {
        g_form.setReadOnly('short_description', true);
        g_form.addInfoMessage("Read-only session: Integration account active.");
    }
}
```

#### 3. Log Updates by Username
* **Explanation:** Tracks UI actions for debugging.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    jslog("Field changed. Triggered by username: " + g_user.userName);
}
```

#### 4. Auto-Fill System Log Header
* **Explanation:** Adds user login handle to short descriptions.
```javascript
function onLoad() {
    if (g_form.isNewRecord()) {
        g_form.setValue('short_description', "[" + g_user.userName + "]: ");
    }
}
```

#### 5. onSubmit Test Profile Verification
* **Explanation:** Prevents submission using a test account name profile.
```javascript
function onSubmit() {
    if (g_user.userName.includes('test')) {
        g_form.addErrorMessage("Test accounts cannot submit records.");
        return false;
    }
    return true;
}
```

---

## 2. Methods (Functions)

---

### A. `g_user.getFullName()`
* **Meaning:** Combines first name and last name into a single display string.

#### 1. Pre-populate Description Header
* **Explanation:** Pre-fills descriptions with the reporting user's full name.
```javascript
function onLoad() {
    if (g_form.isNewRecord()) {
        g_form.setValue('short_description', "Reported by " + g_user.getFullName() + " - ");
    }
}
```

#### 2. Confirm Escalation Dialog
* **Explanation:** Injects full name into confirmation modals on submission.
```javascript
function onSubmit() {
    if (g_form.getValue('priority') === '1') {
        var conf = confirm("Escalating P1 on behalf of " + g_user.getFullName() + "?");
        return conf;
    }
    return true;
}
```

#### 3. Checklist Sign-Off Block
* **Explanation:** Appends full name to work notes on verification check.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    if (newValue === 'true') {
        g_form.setValue('work_notes', "Sign-off checklist verified by: " + g_user.getFullName());
    }
}
```

#### 4. Header Field Info Message
* **Explanation:** Inline message updating tracking names.
```javascript
function onLoad() {
    g_form.showFieldMsg('caller_id', "Active ticket session for: " + g_user.getFullName(), 'info');
}
```

#### 5. Log Sign-Off Event
* **Explanation:** Writes logs to trace user workflows.
```javascript
function onSubmit() {
    jslog("Submission finalized. Certified by: " + g_user.getFullName());
    return true;
}
```

---

### B. `g_user.hasRole(roleName)`
* **Meaning:** Checks if the user holds the role (or is an Admin).

#### 1. Display Internal Work Notes Field
* **Explanation:** Hides/displays internal fields based on ITIL role.
```javascript
function onLoad() {
    if (g_user.hasRole('itil')) {
        g_form.setDisplay('work_notes', true);
    } else {
        g_form.setDisplay('work_notes', false);
    }
}
```

#### 2. Make Category Mandatory for Agents
* **Explanation:** Makes category fields mandatory only for ITIL agents.
```javascript
function onLoad() {
    if (g_user.hasRole('itil')) {
        g_form.setMandatory('category', true);
    }
}
```

#### 3. Show Configuration Item Field
* **Explanation:** Toggles CMDB asset lookup field based on agent roles.
```javascript
function onLoad() {
    if (g_user.hasRole('itil')) {
        g_form.setDisplay('cmdb_ci', true);
    } else {
        g_form.setDisplay('cmdb_ci', false);
    }
}
```

#### 4. Unlocking Assignment Groups
* **Explanation:** Only lets fulfillers update the assignment group.
```javascript
function onLoad() {
    if (!g_user.hasRole('itil')) {
        g_form.setReadOnly('assignment_group', true);
    }
}
```

#### 5. Specialized Banner Notification
* **Explanation:** Shows target agent help message on form load.
```javascript
function onLoad() {
    if (g_user.hasRole('itil')) {
        g_form.addInfoMessage("Agent workspace active. Assign groups accordingly.");
    }
}
```

---

### C. `g_user.hasRoleExactly(roleName)`
* **Meaning:** Checks explicit role assignment (no Admin bypass).

#### 1. Financial Override Lock
* **Explanation:** Restricts financial input fields strictly to the `financial_approver` role.
```javascript
function onLoad() {
    if (!g_user.hasRoleExactly('financial_approver')) {
        g_form.setReadOnly('u_financial_limit', true);
    }
}
```

#### 2. Secure Token Modification Control
* **Explanation:** Restricts critical token configurations to explicit `security_admin` users.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    if (!g_user.hasRoleExactly('security_admin')) {
        g_form.setValue('u_secure_token', oldValue);
        g_form.addErrorMessage("Security token modifications require explicit security_admin session.");
    }
}
```

#### 3. Critical Record Deletion Checkbox
* **Explanation:** Limits deletion requests to explicit `catalog_deleter` role.
```javascript
function onLoad() {
    if (!g_user.hasRoleExactly('catalog_deleter')) {
        g_form.setDisplay('u_request_deletion', false);
    }
}
```

#### 4. Strict Compliance Sign-off Block
* **Explanation:** Requires explicit compliance role to toggle checklist approvals.
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    if (newValue === 'true' && !g_user.hasRoleExactly('compliance_auditor')) {
        g_form.setValue('u_compliance_signed', false);
        g_form.addErrorMessage("Aborted: Compliance sign-off requires compliance_auditor role.");
    }
}
```

#### 5. Scoped App Configuration Lock
* **Explanation:** Limits application parameters fields to explicit app managers.
```javascript
function onLoad() {
    if (!g_user.hasRoleExactly('x_custom_app.admin')) {
        g_form.setReadOnly('u_app_endpoints', true);
    }
}
```

---

### D. `g_user.hasRoles()`
* **Meaning:** Checks if the user is assigned any roles (excluding 'public').

#### 1. Hide Agent-Only Sections
* **Explanation:** Toggles entire sections based on agent status.
```javascript
function onLoad() {
    if (g_user.hasRoles()) {
        g_form.setSectionDisplay('agent_metrics_section', true);
    } else {
        g_form.setSectionDisplay('agent_metrics_section', false);
    }
}
```

#### 2. Portal-Friendly Instructions Display
* **Explanation:** Shows standard help message only to external end-users (no roles).
```javascript
function onLoad() {
    if (!g_user.hasRoles()) {
        g_form.addInfoMessage("Please fill in contact details. An agent will respond shortly.");
    }
}
```

#### 3. Mandatory Checklist for Agents
* **Explanation:** Requires checklist inputs on onSubmit if user has any roles.
```javascript
function onSubmit() {
    if (g_user.hasRoles() && g_form.getValue('u_checklist') === '') {
        g_form.addErrorMessage("Agents must complete verification checklist before closing.");
        return false;
    }
    return true;
}
```

#### 4. Hide Technical Variables
* **Explanation:** Hides advanced tracking configurations from public users.
```javascript
function onLoad() {
    if (!g_user.hasRoles()) {
        g_form.setDisplay('u_technical_class', false);
    }
}
```

#### 5. Routing Ticket Submissions
* **Explanation:** Blocks public users from submitting Change requests directly.
```javascript
function onSubmit() {
    if (!g_user.hasRoles()) {
        g_form.addErrorMessage("Public accounts cannot submit Change Requests.");
        return false;
    }
    return true;
}
```

---

### E. `g_user.hasRoleFromList(rolesList)`
* **Meaning:** Checks if user has at least one role from a comma-separated list.

#### 1. Shared Department Form Section Access
* **Explanation:** Shows department button to HR, ITIL, or Facilities managers.
```javascript
function onLoad() {
    var hasAccess = g_user.hasRoleFromList('hr_agent, facilities_agent, itil');
    if (hasAccess) {
        g_form.setSectionDisplay('shared_fulfillment_details', true);
    } else {
        g_form.setSectionDisplay('shared_fulfillment_details', false);
    }
}
```

#### 2. Unlock Request Assignment Fields
* **Explanation:** Makes assignment fields editable for target fulfillment divisions.
```javascript
function onLoad() {
    var isFulfiller = g_user.hasRoleFromList('asset, catalog_admin, itil');
    if (isFulfiller) {
        g_form.setReadOnly('assigned_to', false);
    } else {
        g_form.setReadOnly('assigned_to', true);
    }
}
```

#### 3. Target Banner Instructions Display
* **Explanation:** Shows help banner if user belongs to ITIL or Security divisions.
```javascript
function onLoad() {
    if (g_user.hasRoleFromList('itil, security_analyst')) {
        g_form.addInfoMessage("Fulfillment Queue Active. Log security details.");
    }
}
```

#### 4. Mandatory Justification Fields
* **Explanation:** Forces explanation input if user is a Change Manager or Auditor.
```javascript
function onSubmit() {
    var holdsTargetRole = g_user.hasRoleFromList('change_manager, auditor');
    if (holdsTargetRole && g_form.getValue('u_justification') === '') {
        g_form.addErrorMessage("Justification is mandatory for change review.");
        return false;
    }
    return true;
}
```

#### 5. Restrict Dropdown Choices List
* **Explanation:** Limits dropdown category inputs for specific fulfillment groups.
```javascript
function onLoad() {
    var hasAdminRights = g_user.hasRoleFromList('admin, catalog_admin');
    if (!hasAdminRights) {
        g_form.removeOption('category', 'database_override');
    }
}
```
