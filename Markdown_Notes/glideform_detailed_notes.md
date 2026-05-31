# ServiceNow GlideForm (g_form) Master Reference Guide

This document serves as your master reference manual for the client-side `g_form` (GlideForm) API. Each section provides details on **Why Used**, **Where Used**, and contains **real-world practical code examples**.

---

## 1. Field Values Manipulation

### A. `g_form.getValue(fieldName)`
* **Why Used:** To read the user's input or current field value in browser memory before applying dynamic form logic, validation, or sending details to the database.
* **Where Used:** Inside `onChange` or `onSubmit` Client Scripts to inspect selections and conditional rules.
* **Code Example:**
  ```javascript
  // Check incident state to verify if additional inputs are needed
  var state = g_form.getValue('state');
  if (state === '3') { // 3 = On Hold
      jslog('Ticket is currently on hold.');
  }
  ```

### B. `g_form.setValue(fieldName, value, displayValue)`
* **Why Used:** To dynamically auto-populate or modify fields based on form events, calculations, or other selections. For reference fields, passing `displayValue` as the third parameter prevents ServiceNow from performing extra database lookups.
* **Where Used:** In `onLoad` templates or `onChange` scripts to direct user paths or fill default data.
* **Code Example:**
  ```javascript
  // Set assignment group and provide display name to prevent query lag
  g_form.setValue('assignment_group', 'd625dccec611227a0000851e3e7f2235', 'Service Desk');
  ```

### C. `g_form.clearValue(fieldName)`
* **Why Used:** To wipe out existing or stale user input from a field when conditions change (e.g., if a sub-category becomes irrelevant after changing the main category).
* **Where Used:** In `onChange` scripts when a parent selector field is modified.
* **Code Example:**
  ```javascript
  // Clear the subcategory if the main category is reset to empty
  if (g_form.getValue('category') === '') {
      g_form.clearValue('subcategory');
  }
  ```

### D. `g_form.getUniqueValue()`
* **Why Used:** To retrieve the unique `sys_id` string of the record currently displayed in the form, without needing to locate a specific field control.
* **Where Used:** In client scripts or client-side UI actions to reference the current record's primary key (e.g., passing it to GlideAjax or building link URLs).
* **Code Example:**
  ```javascript
  // Fetch current incident sys_id to run background processes
  var recordSysID = g_form.getUniqueValue();
  jslog('Processing sys_id: ' + recordSysID);
  ```

### E. `g_form.isNewRecord()`
* **Why Used:** To determine if the record has been saved to the database yet. Returns `true` if it is a brand-new form load, and `false` if it is an existing record being updated.
* **Where Used:** In `onLoad` scripts to prevent execution of certain layout scripts on existing records, or to set defaults on new records.
* **Code Example:**
  ```javascript
  // Auto-populate justification defaults only for new forms
  if (g_form.isNewRecord()) {
      g_form.setValue('description', 'Default template for new tickets:');
  }
  ```


---

## 2. Visibility & Display

### A. `g_form.setDisplay(fieldName, true/false)`
* **Why Used:** Standard best practice to hide/show fields. Setting to `false` hides both the label and field control, and collapses the vertical spacing so the layout remains tight and clean.
* **Where Used:** On conditional field displays (e.g., showing a "Reason" field only if "On Hold" is selected).
* **Code Example:**
  ```javascript
  // Hide justification field unless state is On Hold
  var state = g_form.getValue('state');
  if (state !== '3') {
      g_form.setDisplay('hold_reason', false);
  }
  ```

### B. `g_form.setVisible(fieldName, true/false)`
* **Why Used:** Hides/shows fields, but leaves the blank space where the field was located intact. Used when you want to preserve spacing alignment in multi-column layouts, though generally avoided in favor of `setDisplay()`.
* **Where Used:** onLoad or onChange scripts where column alignment must not be disrupted.
* **Code Example:**
  ```javascript
  // Hide field but preserve alignment grid spacing
  g_form.setVisible('u_secondary_contact', false);
  ```

### C. `g_form.setSectionDisplay(sectionName, true/false)`
* **Why Used:** To show or hide entire form tabs or sections dynamically based on record categories or workflows, rather than showing/hiding individual fields.
* **Where Used:** In `onLoad` or `onChange` Client Scripts when entire groups of fields (e.g., "Resolution Information" or "Hardware Specifications") are not relevant to the current ticket scope.
* **Code Example:**
  ```javascript
  // Hide the "resolution_information" section unless the ticket is resolved/closed
  var state = g_form.getValue('state');
  if (state !== '6' && state !== '7') { // 6=Resolved, 7=Closed
      g_form.setSectionDisplay('resolution_information', false);
  }
  ```

### D. `g_form.isSectionVisible(sectionName)`
* **Why Used:** Checks whether a specific form section or tab is currently visible to the user.
* **Where Used:** onLoad or onChange scripts where logic should only run if the target section is active or visible.
* **Code Example:**
  ```javascript
  // Only validate resolution fields if that section is visible
  if (g_form.isSectionVisible('resolution_information')) {
      g_form.setMandatory('close_notes', true);
  }
  ```



---

## 3. Mandatory & Read-only Controls

### A. `g_form.setMandatory(fieldName, true/false)`
* **Why Used:** To enforce data completeness. Prevents form submission until the specified field contains a non-empty value.
* **Where Used:** Inside `onChange` or `onSubmit` Client Scripts to dynamically require inputs based on ticket severity or state transitions.
* **Code Example:**
  ```javascript
  // Make Close Notes mandatory if resolving the incident
  var state = g_form.getValue('state');
  if (state === '6') { // 6 = Resolved
      g_form.setMandatory('close_notes', true);
  }
  ```

### B. `g_form.setReadOnly(fieldName, true/false)`
* **Why Used:** To protect data integrity. Locks fields so users can view the data but cannot modify it (e.g., locking creation dates, serial numbers, or read-only metrics).
* **Where Used:** On fields populated by integrations or system properties that should never be overwritten manually.
* **Code Example:**
  ```javascript
  // Lock the Caller field after the record has been submitted and saved
  if (!g_form.isNewRecord()) {
      g_form.setReadOnly('caller_id', true);
  }
  ```

### C. `g_form.setDisabled(fieldName, true/false)`
* **Why Used:** Older alternative to `setReadOnly()`. Completely disables field interaction and gray-scales the display. Note: `setReadOnly()` is standard and preferred for modern accessibility, as users can still select and copy text from read-only fields, whereas disabled fields are fully locked from cursor copy selection.
* **Where Used:** In client scripts when fields must be visually grayed out to signal total inactivity.
* **Code Example:**
  ```javascript
  // Disable the alternate email field completely
  g_form.setDisabled('u_alternate_email', true);
  ```


---

## 4. Labels

### A. `g_form.setLabel(fieldName, labelString)`
* **Why Used:** To dynamically change the display name of a field label on the fly. Improves usability when a field role shifts based on context.
* **Where Used:** On generic reference fields that represent different records depending on task types.
* **Code Example:**
  ```javascript
  // Relabel configuration item field if Category is Software
  if (g_form.getValue('category') === 'software') {
      g_form.setLabel('cmdb_ci', 'Target Software Application');
  }
  ```

### B. `g_form.getLabelOf(fieldName)`
* **Why Used:** To retrieve the current display label string of a field. Highly useful for generating user-friendly error messages that refer to fields by their current display labels dynamically.
* **Where Used:** In onSubmit validation scripts to provide meaningful validation banners.
* **Code Example:**
  ```javascript
  // Construct a validation error using the dynamic label of description field
  var descLabel = g_form.getLabelOf('description');
  g_form.addErrorMessage('Please fill in the required field: ' + descLabel);
  ```


---

## 5. Feedback & Messages

### A. Field-Specific Messages (`showFieldMsg` & `hideFieldMsg`)
* **Why Used:** To guide the user directly where their cursor is focused. Displays inline warnings, instructions, or errors right below the field.
* **Where Used:** For instant validation warnings (e.g., email format errors, character limits).
* **Code Example:**
  ```javascript
  // Display a warning message under the configuration item
  g_form.showFieldMsg('cmdb_ci', 'Checking software licensing restrictions...', 'info');
  // Clear message when validated
  g_form.hideFieldMsg('cmdb_ci');
  ```

### B. Form-Wide Banner Messages (`addInfoMessage`, `addErrorMessage`, `clearMessages`)
* **Why Used:** To show global notifications or blocking errors at the top of the form page.
* **Where Used:** onLoad to greet the user or onSubmit to alert them of general script failures.
* **Code Example:**
  ```javascript
  // Clear stale notifications and throw a critical banner error
  g_form.clearMessages();
  g_form.addErrorMessage('Submission aborted: Selected schedule conflicts with active maintenance window.');
  ```

---

## 6. Reference Field Records Asynchronous Access

### A. `g_form.getReference(fieldName, callback)`
* **Why Used:** To retrieve data from tables referenced by fields on the current form (e.g., Caller's department, VIP status, or manager email).
* **Where Used:** onLoad or onChange scripts needing related user profile or asset details.
* **Chaining Warning (getReference().property):** Chaining directly like `g_form.getReference('caller_id').email` runs a **synchronous** database lookup that blocks the browser execution thread, freezing the user's screen. If run inside scoped applications, it will throw error alerts. **Best Practice is to access properties asynchronously inside the callback function.**
* **Code Example:**
  ```javascript
  // Recommended Asynchronous Email Retrieval
  g_form.getReference('caller_id', function(callerRecord) {
      // Safely access fields like .email or .phone off the callback object
      var emailAddress = callerRecord.email; 
      if (emailAddress) {
          g_form.setValue('u_alternate_email', emailAddress);
      }
  });
  ```


## 7. Choice List Manipulation

### A. `g_form.clearOptions(fieldName)`
* **Why Used:** To remove all options from a dropdown choice field. Usually done right before adding new filtered choices to prevent option duplicates or stale selections.
* **Where Used:** In `onChange` scripts where parent fields dynamically determine the valid set of children choices.
* **Code Example:**
  ```javascript
  // Wipe out the subcategory choices when main category shifts
  g_form.clearOptions('subcategory');
  ```

### B. `g_form.addOption(fieldName, choiceValue, choiceLabel, choiceIndex)`
* **Why Used:** Dynamically appends a choice option to a dropdown list. The optional `choiceIndex` specifies the exact 0-indexed position to place the choice (placed at the end if omitted).
* **Where Used:** In `onChange` scripts to rebuild custom choice lists on demand.
* **Code Example:**
  ```javascript
  // Add customized options based on category selection
  g_form.addOption('subcategory', 'email_issue', 'Email & Collaboration', 0);
  g_form.addOption('subcategory', 'network_issue', 'VPN & Networks', 1);
  ```

### C. `g_form.removeOption(fieldName, choiceValue)`
* **Why Used:** Dynamically deletes a single choice option from a dropdown list.
* **Where Used:** To restrict options depending on user roles or current ticket status (e.g. disabling "Resolved" category for end-users).
* **Code Example:**
  ```javascript
  // Remove "Closed" state option for standard users
  if (!g_form.hasRoleExactly('admin')) {
      g_form.removeOption('state', '7'); // 7 = Closed
  }
  ```

---

## 8. Utility Methods

### A. `g_form.getFieldNames()`
* **Why Used:** To retrieve a full list of all fields present on the current layout, which is useful when bulk locking or clearing values.
* **Code Example:**
  ```javascript
  // Make all fields on the form read-only dynamically
  var fields = g_form.getFieldNames();
  for (var i = 0; i < fields.length; i++) {
      g_form.setReadOnly(fields[i], true);
  }
  ```

### B. State Check Methods (`isMandatory`, `isVisible`, `isReadOnly`)
* **Why Used:** To check the current state of a field before modifying it.
* **Code Example:**
  ```javascript
  // Avoid clearing values if the field is locked/read-only
  if (!g_form.isReadOnly('description')) {
      g_form.clearValue('description');
  }
  ```

---

## 9. Form Actions

### A. `g_form.save()` & `g_form.submit()`
* **Why Used:**
  * `save()` sends data to the server and saves the record but keeps the user on the current page.
  * `submit()` saves the record and redirects the user back to the list view or previous page.
* **Where Used:** Used inside custom client-side UI Actions or custom validation scripts.
* **Code Example:**
  ```javascript
  // Force a form save after updating descriptions programmatically
  g_form.setValue('short_description', 'Auto-generated on ticket click');
  g_form.save();
  ```

---

## 10. Advanced Script Combinations


### A. Conditional Mandatory Logic
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;
    
    // Require reasoning justification only if priority is set to Critical
    if (newValue === '1') {
        g_form.setMandatory('u_justification_reason', true);
        g_form.setDisplay('u_justification_reason', true);
    } else {
        g_form.setMandatory('u_justification_reason', false);
        g_form.setDisplay('u_justification_reason', false);
    }
}
```

### B. Custom Field Length Form Validation
```javascript
function onSubmit() {
    var desc = g_form.getValue('description');
    
    // Enforce description minimum character length of 20
    if (desc.trim().length < 20) {
        g_form.clearMessages();
        g_form.addErrorMessage('Form submission failed: Description must contain at least 20 characters.');
        g_form.showFieldMsg('description', 'Provide more description details.', 'error');
        return false; // Aborts submission to database
    }
    return true; // Allows save
}
```
