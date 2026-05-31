# ServiceNow Scripting & APIs: Master Reference Library

Welcome to the **ServiceNow Developer Reference Library**. This repository contains a curated collection of executive-grade study plans, reference manuals, code cheat sheets, and print-ready textbooks. It is designed to take developers from basic javascript concepts to advanced client-side UI and server-side database integration patterns within the ServiceNow platform.

---

## 📂 Repository Structure

The files are organized into two primary folders:
* 📄 **[`PDF_Notes/`](./PDF_Notes)**: Professional, print-ready PDF textbooks styled with executive corporate navy/teal branding.
* 📝 **[`Markdown_Notes/`](./Markdown_Notes)**: Raw markdown (`.md`) source files containing editable script templates and structural definitions.

---

## 📘 Study Guide Catalog & Details

### 1. ServiceNow GlideRecord CRUD Operations Manual
* **Files:** [`ServiceNow_GlideRecord_CRUD_Notes.pdf`](./PDF_Notes/ServiceNow_GlideRecord_CRUD_Notes.pdf) | [`gliderecord_crud_notes.md`](./Markdown_Notes/gliderecord_crud_notes.md)
* **Description:** A comprehensive reference manual covering server-side database operations using the `GlideRecord` class.
* **Contents:** Detailed explanations, use cases, and code templates for performing **CRUD** operations:
  * **C**reate: Using `initialize()` vs. `newRecord()`, and executing database inserts via `insert()`.
  * **R**ead: Retrieving single records using `get()` and multiple records using `addQuery()`, `addEncodedQuery()`, `query()`, and `next()`.
  * **U**pdate: Modifying queried records using `update()` and bulk updating using `updateMultiple()`.
  * **D**elete: Deleting single records using `deleteRecord()` and bulk deleting using `deleteMultiple()`.
  * *Utilities:* Row counts, limiters (`setLimit()`), sorting (`orderBy()`), and getters/setters (`getValue()`, `setValue()`).

### 2. ServiceNow Client Script Triggers Reference Manual
* **Files:** [`ServiceNow_Client_Script_Triggers.pdf`](./PDF_Notes/ServiceNow_Client_Script_Triggers.pdf) | [`client_script_triggers_notes.md`](./Markdown_Notes/client_script_triggers_notes.md)
* **Description:** An exhaustive catalog detailing the execution lifecycles of Client Script types: `onLoad`, `onChange`, `onSubmit`, and `onCellEdit`.
* **Contents:** **40 production-ready real-world script scenarios** (10 examples per trigger type) with detailed summaries, explanation of business logic, and code templates.
  * *onLoad:* VIP visual highlights, dynamic read-only form locks, and context banners.
  * *onChange:* Cascading choice dropdown filters, VIP escalation rules, and date warnings.
  * *onSubmit:* Start/End schedule validation, IPv4 format check regex, and attachment counters.
  * *onCellEdit:* Bulk list update blocks, role-based cell edit restrictions, and task lockdowns.

### 3. GlideForm (`g_form`) Master Reference Manual
* **Files:** [`ServiceNow_GlideForm_Detailed_Notes.pdf`](./PDF_Notes/ServiceNow_GlideForm_Detailed_Notes.pdf) | [`glideform_detailed_notes.md`](./Markdown_Notes/glideform_detailed_notes.md)
* **Description:** Detailed study manual covering all primary and utility methods of the `g_form` class.
* **Contents:** Details on **Why Used**, **Where Used**, and complete script examples for value manipulation, visibility/collapsible displays (`setDisplay` vs `setVisible`), read-only/mandatory flags, `setDisabled`, dynamic labels (`setLabel`, `getLabelOf`), and inline messages (`showFieldMsg`, `hideFieldMsg`).
* **Includes:** Dynamic choice list manipulation (`clearOptions`, `addOption`, `removeOption`) and best practice rules.

### 4. GlideUser (`g_user`) Manual & Code Catalog
* **Files:** [`ServiceNow_Client_GlideUser_Detailed_Notes.pdf`](./PDF_Notes/ServiceNow_Client_GlideUser_Detailed_Notes.pdf) | [`glideuser_client_detailed_notes.md`](./Markdown_Notes/glideuser_client_detailed_notes.md)
* **Description:** Comprehensive guide to checking user sessions, permissions, and roles.
* **Contents:** **45 copy-paste code examples** covering:
  * Static profile attributes: `.userID`, `.userName`, `.firstName`, `.lastName`.
  * Dynamic checking methods: `getFullName()`, `hasRole()`, `hasRoleExactly()`, `hasRoles()`, and `hasRoleFromList()`.

### 5. GlideAjax laymans guide & Script Include Bridge
* **Files:** [`ServiceNow_GlideAjax_Notes.pdf`](./PDF_Notes/ServiceNow_GlideAjax_Notes.pdf) | [`glideajax_tutorial.md`](./Markdown_Notes/glideajax_tutorial.md)
* **Description:** Layman's guide to asynchronous client-server communication using client-callable Script Includes and the `GlideAjax` API.
* **Contents:** The "Waiter and Kitchen" analogy, asynchronous callback logic, query parameter builders, and database lookup templates.

### 6. ServiceNow GlideAjax: Sync vs. Async Master Guide
* **Files:** [`ServiceNow_GlideAjax_Async_Sync_Notes.pdf`](./PDF_Notes/ServiceNow_GlideAjax_Async_Sync_Notes.pdf) | [`glideajax_async_sync_notes.md`](./Markdown_Notes/glideajax_async_sync_notes.md)
* **Description:** Deep dive explaining synchronous vs. asynchronous GlideAjax lookups and best practices.

### 7. JavaScript for ServiceNow Cheat Sheet
* **Files:** [`ServiceNow_JavaScript_Notes.pdf`](./PDF_Notes/ServiceNow_JavaScript_Notes.pdf) | [`servicenow_javascript_notes.md`](./Markdown_Notes/servicenow_javascript_notes.md)
* **Description:** JavaScript fundamentals mapped directly to ServiceNow scripting blocks.
* **Contents:** Quick guides on arrays, data types, variable scopes (`var`, `let`, `const`), loops, string manipulation, and functional syntax.

### 8. 5-Hour Client-Side Scripting Study Plan
* **Files:** [`ServiceNow_Client_Side_Mastery_Study_Plan.pdf`](./PDF_Notes/ServiceNow_Client_Side_Mastery_Study_Plan.pdf) | [`client_side_mastery_study_plan.md`](./Markdown_Notes/client_side_mastery_study_plan.md)
* **Description:** Structured hourly plan detailing the learning path to master client scripting.

---

## ⚡ Client-Side Scripting Best Practices Summary

1. **Check `isLoading` in `onChange` scripts:** Ensure your script starts with `if (isLoading || newValue === '') { return; }` to prevent slow rendering speeds and infinite execution loops.
2. **Asynchronous `getReference` calls:** Always pass a callback function to `g_form.getReference('field', callback)`. Chaining directly like `getReference('caller_id').email` blocks browser thread execution.
3. **Prefer `setDisplay` over `setVisible`:** `setDisplay(field, false)` hides the field and collapses the vertical space in the form layout, leaving a clean UI.
4. **Wrap field names in quotes:** Remember to always pass field names as strings (e.g. `'short_description'`), not raw tokens.
5. **Clear Banner Messages:** Always call `g_form.clearMessages()` before adding new info/error banners to prevent warning messages from stacking at the top of the form.