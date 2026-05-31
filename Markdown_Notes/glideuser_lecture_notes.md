# ServiceNow Training Lecture Notes: Client-Server Architecture & GlideUser API

This document details the core concepts and examples covered in the ServiceNow training lecture transcript regarding ServiceNow's Client-Server architecture and the **GlideUser (`g_user`)** client-side API.

---

## 1. ServiceNow Client vs. Server Architecture

ServiceNow divides its script executions and data flows into two distinct environments:

```
                  ┌─────────────────────────────────────────┐
                  │          CLIENT SIDE (Browser)          │
                  │  - Takes user inputs (e.g., forms)     │
                  │  - Uses JavaScript + Client-Side APIs  │
                  └────────────────────┬────────────────────┘
                                       │ (Submit / Save / Update)
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │          SERVER SIDE (Cloud)            │
                  │  - Handles database storage (sys_user)  │
                  │  - Uses JavaScript + Server-Side APIs  │
                  └─────────────────────────────────────────┘
```

### Key Concept Check: Boundary Rules
* **Client-side APIs** (like `g_form` or `g_user`) **can only** be used in client-side scripts. They will not execute and will crash on the server.
* **Server-side APIs** (like `GlideRecord` or `gs`) **can only** be used in server-side scripts. They will not execute and will crash on the client.

### Standard JavaScript vs. ServiceNow APIs
* **JavaScript:** The general programming language. It is used in many platforms (Salesforce, Power BI, Tableau, web apps).
* **ServiceNow APIs:** Custom classes and libraries built by ServiceNow (e.g., `GlideUser`, `GlideForm`, `GlideAjax`). They are **exclusive** to ServiceNow and cannot be run anywhere else.

---

## 2. Important Client-Side APIs
These APIs are designed specifically for the browser interface:
1. **`g_user`** (GlideUser): Fetches information about the currently logged-in user and their roles.
2. **`g_form`** (GlideForm): Controls fields, sections, and messages on the form.
3. **`g_scratchpad`**: Transports data from the server-side to the client-side on form load.
4. **`GlideAjax`**: Executes asynchronous database calls to a Script Include.
5. **`g_navigation`** (GlideNavigation): Handles window redirects.
6. **`GlideDialogWindow`** / **`GlideModal`** / **`GlideModalForm`**: Opens pop-ups and overlay forms.
7. **`g_list`** (GlideList): Manipulates list layouts.
8. **`GlideFlow`**: Invokes Flow Designer flows directly from client scripts.

---

## 3. Deep Dive: The GlideUser Client-Side API (`g_user`)

The `g_user` object provides properties and methods that describe the user currently viewing a form.

> [!TIP]  
> **How to Test Client Scripts on a Form:**
> Open any record form (like Incident or Shipping Case) and press **`Ctrl + Shift + J`**. This opens ServiceNow's built-in **Client-Side JavaScript Executor**. You can paste client-side code blocks here and click "Run" to test them instantly.

---

### A. Properties (Fields) vs. Methods
* **Properties:** Store static variables describing the user. Do not use parenthesis `()` to call them.
* **Methods:** Executable functions that run a process and return a value. Must end with parenthesis `()`.

| Member Name | Type | Description |
| :--- | :--- | :--- |
| **`g_user.userID`** | Property | Returns the Sys ID (`sys_id`) of the current user. |
| **`g_user.userName`** | Property | Returns the login ID (username) of the current user (e.g. `admin`). |
| **`g_user.firstName`** | Property | Returns the user's first name. |
| **`g_user.lastName`** | Property | Returns the user's last name. |
| **`g_user.getFullName()`** | Method | Combines first and last names and returns the full name. |

#### Code Example: Retrieving User Details
```javascript
// Test this in JavaScript Executor (Ctrl + Shift + J)
var firstName = g_user.firstName;
var lastName = g_user.lastName;
var fullName = g_user.getFullName();
var loginName = g_user.userName;
var userSysId = g_user.userID;

// Display results in browser pop-ups
alert("First Name: " + firstName);
alert("Last Name: " + lastName);
alert("Full Name: " + fullName);
alert("Login Name: " + loginName);
alert("User Record Sys ID: " + userSysId);
```

---

### B. User Role Checks

#### 1. `g_user.hasRole(roleName)`
* **How it works:** 
  1. Checks if the user is explicitly assigned the role. If yes, returns `true`.
  2. If the user does not have the role, it checks if they have the **`admin`** role. If they are an administrator, it automatically returns `true`.
* **Example:**
```javascript
// Returns true for ITIL agents AND Admins (who bypass role checks)
var isAgent = g_user.hasRole('itil');
alert("Has ITIL or Admin: " + isAgent);
```

#### 2. `g_user.hasRoleExactly(roleName)`
* **How it works:** 
  * Checks if the user has that specific role explicitly assigned to them.
  * **Does NOT automatically return true for Admins.** If the user is an admin but doesn't have this role explicitly, it returns `false`.
* **Example:**
```javascript
// Only returns true if the user has 'security_admin' explicitly assigned
var isSecurityAdmin = g_user.hasRoleExactly('security_admin');
alert("Has exact Security Admin role: " + isSecurityAdmin);
```

#### 3. `g_user.hasRoles()`
* **How it works:** Returns `true` if the user has any roles assigned to them (excluding the default `public` role). Returns `false` if they have zero roles (typically end-users).
* **Example:**
```javascript
var hasAnyRole = g_user.hasRoles();
alert("Has roles assigned: " + hasAnyRole);
```

#### 4. `g_user.hasRoleFromList(rolesList)`
* **How it works:** Returns `true` if the user has at least one of the roles listed in a comma-separated string list. Just like `hasRole()`, it will automatically return `true` for administrators.
* **Example:**
```javascript
var hasFulfillmentRole = g_user.hasRoleFromList('itil, catalog_admin, asset');
alert("Has fulfillment roles: " + hasFulfillmentRole);
```

---

## 4. Key Takeaways for Developers

1. **Client-Side Popups:** `alert()` is a standard JavaScript command that freezes the browser screen until "OK" is clicked. It is useful for developer debugging but should be replaced by `g_form.addInfoMessage()` in final production code.
2. **Server Commands Fail on Client:** Do not use `gs.print()` or `gs.log()` inside client-side scripts. They will cause execution errors.
3. **Role Checks on Impersonation:** When you impersonate another user (e.g. impersonating a non-admin user), `g_user` values automatically update to reflect that user's profile, fields, and roles.
