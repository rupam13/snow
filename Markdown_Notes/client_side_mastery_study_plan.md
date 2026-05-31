# ServiceNow Client-Side Scripting: Intensive 5-Hour Mastery Study Plan

This study plan is structured as an intensive, hour-by-hour guide to help you master client-side scripting and APIs in ServiceNow today. 

---

## 📅 Study Plan Overview

```
 ┌────────────────────────────────────────────────────────┐
 │                      STUDY PATH                        │
 ├────────┬───────────────────────────────────────────────┤
 │ Hour 1 │ Form Control (g_form) & Notifications         │
 ├────────┼───────────────────────────────────────────────┤
 │ Hour 2 │ User Profile (g_user) & Client Script Types   │
 ├────────┼───────────────────────────────────────────────┤
 │ Hour 3 │ UI Policies vs. Client Scripts (Execution)    │
 ├────────┼───────────────────────────────────────────────┤
 │ Hour 4 │ Client-Server Bridge (GlideAjax & Scratchpad) │
 ├────────┼───────────────────────────────────────────────┤
 │ Hour 5 │ Navigation, Modals, & JavaScript Executor     │
 └────────┴───────────────────────────────────────────────┘
```

---

## ⏰ Hour 1: Form Control (`g_form`) & Banner Notifications

### 📘 Theory (30 Minutes)
`g_form` (GlideForm) is the primary class used to manipulate field properties and display messages on forms.
* **Visibility vs. Display:** 
  * `g_form.setDisplay('field', false)` hides the field and collapses the space (recommended).
  * `g_form.setVisible('field', false)` hides the field but leaves an empty space.
* **Notification Mechanics:** Banners appear at the top of the form, while field messages appear directly under the input field.

### 🛠️ Practical Exercise (30 Minutes)
Create an `onChange` Client Script on the **Incident** table. When the **Urgency** changes to **1 (High)**:
1. Make **Description** mandatory.
2. Hide the **Configuration Item** (`cmdb_ci`) field.
3. Show an error banner at the top of the form.
4. Add a field message below the **Short Description** field.

#### Solution Template:
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }
    
    if (newValue === '1') { // 1 = High Urgency
        g_form.setMandatory('description', true);
        g_form.setDisplay('cmdb_ci', false);
        g_form.addErrorMessage('Urgency is set to High! Please fill in the description.');
        g_form.showFieldMsg('short_description', 'Ensure summary is specific', 'info');
    } else {
        g_form.setMandatory('description', false);
        g_form.setDisplay('cmdb_ci', true);
        g_form.clearMessages();
        g_form.hideFieldMsg('short_description');
    }
}
```

---

## ⏰ Hour 2: User Profiles (`g_user`) & Script Trigger Types

### 📘 Theory (30 Minutes)
* **Client Script Types:**
  1. **`onLoad`**: Runs when the form finishes loading. Great for setting field visibility or adding greetings.
  2. **`onChange`**: Runs when a specific field value changes. Includes variables `oldValue` and `newValue`.
  3. **`onSubmit`**: Runs when the user clicks Save or Submit. Return `false` to abort the save database operation.
  4. **`onCellEdit`**: Runs on list views when double-clicking cells.
* **Role Check Difference:** `g_user.hasRole('itil')` checks the role but auto-returns true for Admins, whereas `g_user.hasRoleExactly('itil')` checks only the explicit assignment.

### 🛠️ Practical Exercise (30 Minutes)
Create an `onSubmit` Client Script on the **Change Request** table:
* Check if the user is an **Admin** using `g_user.hasRoleExactly('admin')`.
* If the user is **not** an Admin, and the **Planned end date** is empty, block the form submission and display an error message.

#### Solution Template:
```javascript
function onSubmit() {
    var isAdmin = g_user.hasRoleExactly('admin');
    var plannedEnd = g_form.getValue('planned_end_date');
    
    if (!isAdmin && plannedEnd === '') {
        g_form.addErrorMessage('Only Admins can submit change requests without a Planned End Date.');
        return false; // Aborts submission!
    }
    return true; // Allows submission
}
```

---

## ⏰ Hour 3: UI Policies vs. Client Scripts

### 📘 Theory (30 Minutes)
* **UI Policies:** Condition-based rules that run client-side. They do not require code to make fields mandatory, read-only, or visible.
* **Performance:** UI Policies are lighter and execute faster than Client Scripts.
* **Execution Order:** 
  1. `onLoad` Client Scripts run first.
  2. UI Policies run second (overriding Client Scripts).
  3. `onChange` Client Scripts run when values update.

```
                  ┌───────────────────────────────┐
                  │ 1. onLoad Client Scripts Run  │
                  └───────────────┬───────────────┘
                                  ▼
                  ┌───────────────────────────────┐
                  │     2. UI Policies Run        │
                  └───────────────┬───────────────┘
                                  ▼
                  ┌───────────────────────────────┐
                  │ 3. Form is Ready for Inputs   │
                  └───────────────────────────────┘
```

### 🛠️ Practical Exercise (30 Minutes)
1. Configure a **UI Policy** on the Incident table: Condition is `State IS On Hold`. Actions: Make `hold_reason` field mandatory and visible.
2. Create an `onChange` Client Script on `state` that changes `hold_reason` to optional.
3. Observe how they conflict and how the UI Policy takes precedence.

---

## ⏰ Hour 4: The Client-Server Bridge (`GlideAjax` & `g_scratchpad`)

### 📘 Theory (30 Minutes)
* **GlideAjax:** Best practice for fetching database records without freezing the UI. Runs asynchronously via a callback function.
* **`g_scratchpad`:** A server-side object that runs during form load (via a Display Business Rule) and passes values directly to the client script, reducing the need for GlideAjax calls.

### 🛠️ Practical Exercise (30 Minutes)
Create a Client Script and Script Include to retrieve the Caller's Manager:
1. Create a client-callable Script Include `GetUserInfo`.
2. Write a Client Script that triggers on change of `caller_id` and auto-populates the supervisor field.

#### Script Include (Server):
```javascript
var GetUserInfo = Class.create();
GetUserInfo.prototype = Object.extendsObject(AbstractAjaxProcessor, {
    getManager: function() {
        var userId = this.getParameter('sysparm_user_id');
        var userGR = new GlideRecord('sys_user');
        if (userGR.get(userId)) {
            return userGR.getDisplayValue('manager');
        }
        return '';
    },
    type: 'GetUserInfo'
});
```

#### Client Script (Client):
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;

    var ga = new GlideAjax('GetUserInfo');
    ga.addParam('sysparm_name', 'getManager');
    ga.addParam('sysparm_user_id', newValue);
    ga.getXMLAnswer(function(answer) {
        g_form.setValue('u_manager_name', answer);
    });
}
```

---

## ⏰ Hour 5: Redirections, Modals, & The Javascript Executor

### 📘 Theory (30 Minutes)
* **`g_navigation`**: Provides client redirection API options (e.g. `g_navigation.open(url, target)`).
* **`GlideModal`**: Displays professional popup overlays without blocking the browser window like raw alerts.
* **JavaScript Executor (`Ctrl + Shift + J`):** Opens a scratchpad terminal directly on your active form to debug client scripts instantly.

### 🛠️ Practical Exercise (30 Minutes)
Open your personal developer instance, load an Incident record, press **`Ctrl + Shift + J`**, and run this script to open a modal popup:

```javascript
// Test this in JavaScript Executor (Ctrl + Shift + J)
var modal = new GlideModal('glide_confirm_basic');
modal.setTitle('Critical Confirmation');
modal.setPreference('title', 'Are you sure you want to escalate this Incident?');
modal.setPreference('onPromptComplete', function() {
    g_form.addInfoMessage('Escalation confirmed!');
});
modal.render();
```
