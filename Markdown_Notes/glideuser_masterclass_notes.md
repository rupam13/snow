# ServiceNow GlideUser API: Ultimate Masterclass Reference Guide

This reference manual provides a detailed, scratch-to-advanced explanation of the **GlideUser API** in ServiceNow. It covers both the client-side (`g_user`) and server-side (`gs.getUser()`) contexts, explaining every property, method, and use case from easy to hard.

---

## 1. GlideUser Architecture & Scope Boundaries

In ServiceNow, information about the currently logged-in user is accessed differently depending on where the script is running:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        GLIDEUSER API BOUNDARY                          │
├───────────────────────────────────┬────────────────────────────────────┤
│     CLIENT-SIDE (g_user)          │     SERVER-SIDE (gs.getUser())     │
├───────────────────────────────────┼────────────────────────────────────┤
│ - Runs in browser memory.         │ - Runs on ServiceNow cloud server. │
│ - Instant access, no lag.         │ - Accesses database profiles.      │
│ - Used for UI changes (g_form).   │ - Used for database rules & ACLs.  │
│ - Untrusted (can be bypassed).    │ - Secure & trusted.                │
└───────────────────────────────────┴────────────────────────────────────┘
```

---

## 2. Client-Side GlideUser (`g_user`)

The client-side `g_user` class is a pre-instantiated global object representing the user currently viewing a form.

---

### A. Granular Properties (Variables)
Properties do not require parentheses `()` and represent static variables loaded during the page session.

#### 1. `g_user.firstName`
* **Detailed Explanation:** Returns the first name string of the currently logged-in user, as configured on the `sys_user` record.
* **Value Type:** String

#### 2. `g_user.lastName`
* **Detailed Explanation:** Returns the last name string of the currently logged-in user, as configured on the `sys_user` record.
* **Value Type:** String

#### 3. `g_user.userID`
* **Detailed Explanation:** Returns the unique 32-character database primary key (`sys_id`) of the currently logged-in user. **This is a property, not a method.**
* **Value Type:** String

#### 4. `g_user.userName`
* **Detailed Explanation:** Returns the user's login ID (the username used to log in, e.g. `admin` or `john.smith`).
* **Value Type:** String

---

### B. Granular Methods (Functions)
Methods are executable functions that calculate or check conditions and must end with parentheses `()`.

#### 1. `g_user.getFullName()`
* **Detailed Explanation:** Combines `firstName` and `lastName` with a space to return the display name.
* **Returns:** String

#### 2. `g_user.hasRole(roleName)`
* **Detailed Explanation:** Returns `true` if the user has the specified role **OR if the user is a System Administrator (`admin`)**. Admin users bypass all role checks and always return `true` for this method.
* **Returns:** Boolean (`true` / `false`)

#### 3. `g_user.hasRoleExactly(roleName)`
* **Detailed Explanation:** Returns `true` **only** if the user has been explicitly assigned the specified role. **Administrators do not bypass this check.** If an admin is not explicitly assigned the role, it returns `false`.
* **Returns:** Boolean (`true` / `false`)

#### 4. `g_user.hasRoles()`
* **Detailed Explanation:** Checks if the user is assigned any roles at all (excluding the standard default `public` role). Useful for distinguishing internal agents from external end-users.
* **Returns:** Boolean (`true` / `false`)

#### 5. `g_user.hasRoleFromList(rolesList)`
* **Detailed Explanation:** Accepts a comma-separated string list of roles (e.g. `'itil, asset, catalog_admin'`) and returns `true` if the user holds at least one of these roles (or is an Admin).
* **Returns:** Boolean (`true` / `false`)

---

### C. Client Use Cases: Easy to Hard

#### 🟢 Use Case 1: Display Greeting Banner (Easy)
* **Goal:** Display a custom info greeting banner on the Incident form when it loads, addressing the user by their full name.
* **Client Script (onLoad):**
```javascript
function onLoad() {
    var fullName = g_user.getFullName();
    g_form.addInfoMessage("Welcome back, " + fullName + "! Please update your open assignments.");
}
```

#### 🟡 Use Case 2: Field Controls Based on Roles (Medium)
* **Goal:** When the Incident form loads, check if the user is a licensed ITIL agent. If not, hide the internal "Work Notes" field and make the public "Additional Comments" field mandatory.
* **Client Script (onLoad):**
```javascript
function onLoad() {
    // If user has 'itil' role (or is Admin), they can see work notes
    if (g_user.hasRole('itil')) {
        g_form.setDisplay('work_notes', true);
    } else {
        g_form.setDisplay('work_notes', false);
        g_form.setMandatory('comments', true);
    }
}
```

#### 🔴 Use Case 3: Admin Lockout & Explicit Roles (Hard)
* **Goal:** A custom field `u_secure_override` is only editable by users who explicitly hold the `security_admin` role. Even standard Administrators (`admin`) should be locked out unless they explicitly have `security_admin`.
* **Client Script (onLoad):**
```javascript
function onLoad() {
    // hasRoleExactly prevents standard admin bypass
    var hasSecureAccess = g_user.hasRoleExactly('security_admin');
    
    if (hasSecureAccess) {
        g_form.setReadOnly('u_secure_override', false);
        g_form.showFieldMsg('u_secure_override', 'Authorized security session active', 'info');
    } else {
        g_form.setReadOnly('u_secure_override', true);
    }
}
```

---

## 3. Server-Side GlideUser (`gs.getUser()`)

On the server, you retrieve the current user's database object using the GlideSystem API call: `var currentUser = gs.getUser();`.

---

### A. Granular Profile Methods

#### 1. `gs.getUserID()` (Shorthand)
* **Detailed Explanation:** Returns the Sys ID string of the logged-in user. (Equivalent to `gs.getUser().getID()`).
* **Returns:** String

#### 2. `gs.getUserName()` (Shorthand)
* **Detailed Explanation:** Returns the login username string of the logged-in user. (Equivalent to `gs.getUser().getName()`).
* **Returns:** String

#### 3. `gs.getUser().getDisplayName()`
* **Detailed Explanation:** Returns the full display name of the user from their profile card.
* **Returns:** String

#### 4. `gs.getUser().getEmail()`
* **Detailed Explanation:** Retrieves the user's primary email address string.
* **Returns:** String

#### 5. `gs.getUser().getDepartmentID()`
* **Detailed Explanation:** Returns the database Sys ID (`sys_id`) of the user's Department record (`cmn_department`).
* **Returns:** String

#### 6. `gs.getUser().getCompanyID()`
* **Detailed Explanation:** Returns the database Sys ID (`sys_id`) of the user's Company record (`core_company`).
* **Returns:** String

#### 7. `gs.getUser().getManagerID()`
* **Detailed Explanation:** Returns the database Sys ID (`sys_id`) of the user's Manager record.
* **Returns:** String

---

### B. Granular Group & Role Check Methods

#### 1. `gs.getUser().isMemberOf(group)`
* **Detailed Explanation:** Checks if the user is a member of a specific group. You can pass the Group Name (String) or the Group Sys ID.
* **Returns:** Boolean (`true` / `false`)

#### 2. `gs.getUser().getMyGroups()`
* **Detailed Explanation:** Retrieves a Java `ArrayList` containing the Sys IDs of all groups the user is a member of.
* **Returns:** Java ArrayList of Strings

#### 3. `gs.getUser().getRoles()`
* **Detailed Explanation:** Retrieves a Java `ArrayList` containing all roles assigned to the user.
* **Returns:** Java ArrayList of Strings

---

### C. Server Use Cases: Easy to Hard

#### 🟢 Use Case 4: Populate Department & Company on Creation (Easy)
* **Goal:** When a user creates a new record on a custom table, automatically populate the Company and Department fields based on the logged-in user's profile.
* **Business Rule (Before Insert):**
```javascript
(function executeRule(current, previous /*null when async*/) {
    var userObj = gs.getUser();
    
    // Auto-populate fields using profile IDs
    current.setValue('u_department', userObj.getDepartmentID());
    current.setValue('u_company', userObj.getCompanyID());
})(current, previous);
```

#### 🟡 Use Case 5: Restrict Queue to Group Members (Medium)
* **Goal:** A specialized Incident category `Database Security` can only be assigned to the `DB Analysis` group. A Business Rule should prevent users from saving a ticket assigned to this group unless the logged-in user is a member of `DB Analysis`.
* **Business Rule (Before Insert/Update):**
```javascript
(function executeRule(current, previous /*null when async*/) {
    var assignmentGroup = current.getDisplayValue('assignment_group');
    
    if (assignmentGroup === 'DB Analysis') {
        var isMember = gs.getUser().isMemberOf('DB Analysis');
        
        if (!isMember) {
            gs.addErrorMessage('Only members of the DB Analysis group can assign incidents to this queue.');
            current.setAbortAction(true); // Aborts database save
        }
    }
})(current, previous);
```

#### 🔴 Use Case 6: Custom ACL Record Ownership Filter (Hard)
* **Goal:** Write an Access Control List (ACL) rule. A user can only write to (update) records on the Custom Asset table if:
  1. The user is a member of the record's `u_ownership_group`.
  2. **OR** the user is the direct manager of the record's Assignee (`assigned_to.manager`).
* **ACL Script (Server):**
```javascript
// ACL evaluates to true if 'answer' is set to true
answer = false;

var ownershipGroup = current.getValue('u_ownership_group');
var assigneeManager = '';

// Get manager ID using a reference query
var assigneeGR = new GlideRecord('sys_user');
if (assigneeGR.get(current.getValue('assigned_to'))) {
    assigneeManager = assigneeGR.getValue('manager');
}

// 1. Check if logged-in user is a member of ownership group
if (ownershipGroup && gs.getUser().isMemberOf(ownershipGroup)) {
    answer = true;
}

// 2. Check if logged-in user is assignee's manager
if (assigneeManager && gs.getUserID() === assigneeManager) {
    answer = true;
}
```
