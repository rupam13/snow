# ServiceNow GlideForm (g_form) Reference Manual

`g_form` (GlideForm) is the primary client-side class in ServiceNow used to control the presentation of forms, inspect user input, manipulate field properties, and display feedback messages.

---

## 📌 Core Concepts
* **Purpose:** Client-side API for manipulating form fields and form presentation dynamically.
* **Scope:** Available inside **Client Scripts**, **UI Policies (Script fields)**, and **UI Actions (Client-side)**.
* **Key Use Cases:** Show/hide fields, set read-only/mandatory flags, dynamically set/clear values, change labels, and display inline field or banner messages.

---

## 🔑 Commonly Used Methods

### A. Value Manipulation

#### 1. `g_form.getValue(fieldName)`
Retrieves the current value of a field on the form as a string.
* **Example:**
  ```javascript
  var priority = g_form.getValue('priority');
  if (priority === '1') {
      // Do something for High Priority
  }
  ```

#### 2. `g_form.setValue(fieldName, value, displayValue)`
Sets the value of a field. For reference fields, it is best practice to pass the `displayValue` as the third parameter to prevent additional database lookups.
* **Example:**
  ```javascript
  g_form.setValue('short_description', 'Updated via automated script');
  // For reference fields:
  g_form.setValue('assigned_to', '5137153ec611227c0000851e3e7f2235', 'John Doe');
  ```

#### 3. `g_form.clearValue(fieldName)`
Clears the current value of the field.
* **Example:**
  ```javascript
  if (g_form.getValue('u_custom_checkbox') === 'false') {
      g_form.clearValue('u_custom_details');
  }
  ```

---

### B. Visibility & Display

#### 1. `g_form.setDisplay(fieldName, true/false)`
Shows or hides the specified field. When hiding, it collapses the space entirely so that no blank gap is left in the layout (preferred standard).
* **Example:**
  ```javascript
  g_form.setDisplay('work_notes', false); // Hides and collapses the layout
  ```

#### 2. `g_form.setVisible(fieldName, true/false)`
Shows or hides the specified field, but leaves the empty space in the layout intact when hidden (causes awkward blank gaps).
* **Example:**
  ```javascript
  g_form.setVisible('configuration_item', false); // Hides but leaves blank space
  ```

---

### C. Mandatory & Read-only

#### 1. `g_form.setMandatory(fieldName, true/false)`
Sets whether the field is mandatory (required before submitting).
* **Example:**
  ```javascript
  g_form.setMandatory('close_notes', true); // Forces close notes to be filled
  ```

#### 2. `g_form.setReadOnly(fieldName, true/false)`
Sets whether the field is read-only (grayed out / locked).
* **Example:**
  ```javascript
  g_form.setReadOnly('opened_by', true); // User cannot change who opened the ticket
  ```

---

### D. Labels & Inline Messages

#### 1. `g_form.setLabel(fieldName, 'New Label')`
Changes the display label of a field dynamically.
* **Example:**
  ```javascript
  g_form.setLabel('u_phone', 'Alternate Phone Number');
  ```

#### 2. `g_form.showFieldMsg(fieldName, 'Message', 'type')`
Displays an inline feedback message directly underneath the input control. The `type` can be `'info'`, `'error'`, or `'warning'`.
* **Example:**
  ```javascript
  g_form.showFieldMsg('short_description', 'Please keep description brief and to the point.', 'info');
  ```

#### 3. `g_form.hideFieldMsg(fieldName, clearAll)`
Removes the inline feedback message underneath the input control. If `clearAll` is `true`, all messages for the field are cleared.
* **Example:**
  ```javascript
  g_form.hideFieldMsg('short_description');
  ```

---

### E. Reference Fields & Asynchronous Retrieval

#### 1. `g_form.getReference(fieldName, callback)`
Fetches the entire record for a reference field from the database asynchronously. **Always** use a callback function to prevent performance degradation (synchronous queries freeze the browser).
* **Example:**
  ```javascript
  g_form.getReference('caller_id', function(caller) {
      // Access columns on the caller's sys_user record
      g_form.setValue('u_alternate_email', caller.email);
      g_form.setValue('u_caller_phone', caller.phone);
  });
  ```

---

## 🧩 Utility Methods

### 1. `g_form.getFieldNames()`
Returns an array of all field names present on the form.
* **Example:**
  ```javascript
  var fields = g_form.getFieldNames();
  jslog("Field list count: " + fields.length);
  ```

### 2. `g_form.isMandatory(fieldName)`
Returns `true` if the field is currently mandatory, otherwise `false`.
* **Example:**
  ```javascript
  if (g_form.isMandatory('description')) {
      // Logic if field is mandatory
  }
  ```

### 3. `g_form.isVisible(fieldName)`
Returns `true` if the field is visible, otherwise `false`.
* **Example:**
  ```javascript
  var showDetails = g_form.isVisible('u_details');
  ```

### 4. `g_form.isReadOnly(fieldName)`
Returns `true` if the field is currently read-only, otherwise `false`.
* **Example:**
  ```javascript
  var locked = g_form.isReadOnly('number');
  ```

---

## ⚡ Best Practices & Guidelines

1. **Always Wrap Fields in Quotes:** Field name parameters must always be specified as literal strings enclosed in single or double quotes (e.g. `'short_description'`).
2. **Prefer `setDisplay` over `setVisible`:** Use `g_form.setDisplay()` to keep the form layout clean and structured without awkward empty spaces.
3. **Always Callback on `getReference`:** Never call `g_form.getReference('field')` without a callback parameter. Without a callback, the platform runs a synchronous database read, blocking the browser execution thread.
4. **Client-side Date Handling:** Always use JavaScript native `Date()` objects for client-side date computations. Never try to use `GlideDateTime` in Client Scripts; it is a server-side API only and will result in runtime errors.
5. **Check Form Loading:** In `onChange` scripts, check `isLoading` or `newValue === ''` to prevent infinite loops or unwanted execution on initial load.
