# ServiceNow Journal Admin Hub (JAHBLESS)

A guided ServiceNow UI for deleting or redacting journal entries (Additional comments / Work notes / other journal_input fields) with audit aligned handling.

> ⚠️ Disclaimer  
> This is provided for learning and experimentation.  
> Do not deploy to production without thorough testing in your own environments, validation of policy requirements, and stakeholder approval.

---

## What it does

- Select a table and record via a guided UI (or drive it via options and URL params)
- View all journal fields for that record as tabs
- Select one or more journal entries
- Perform one of:
  - Delete entry
  - Redact in full
  - Redact specific text
- Requires a reason and presents a confirmation modal that shows exactly what will be affected
- Logs a server side system log entry per journal entry action

---

## Architecture

This solution uses two parts:

1. **Scoped app (x_cros2_journal_0)**
   - UI (Service Portal widget + page)
   - Record picker logic and journal browsing
   - Calls into the global executor for mutations

2. **Global Script Include**
   - Executes the actual update delete operations against system tables
   - The reason for this approach is to not have to change the core config of your sys_ tables
   - Returns success failure with best effort error messaging

---

## Install

### Step 1: Import the scoped update set
Import the scoped update set XML first.

ServiceNow:
- System Update Sets → Retrieved Update Sets
- Import Update Set from XML
- Upload:
  `/scoped/Journal_Administration_Hub_Bulk_Log_Entry_Sanitisation_Suite.xml`
- Preview Update Set
- Commit Update Set

### Step 2: Import the global update set
Then import the global update set XML.

ServiceNow:
- System Update Sets → Retrieved Update Sets
- Import Update Set from XML
- Upload:
  `/global/JAHBLESS_Global.xml`
- Preview Update Set
- Commit Update Set

### Step 3: Configure properties
The app supports these sys_properties:

- `allowed_roles`
  - Comma separated roles allowed to use the tool
  - Empty means no restriction
- `use_secure_lookup`
  - `true` uses GlideRecordSecure for lookups
  - `false` uses GlideRecord

---

## Usage

### Default guided mode
1. Open **Journal Admin Hub** module
2. Select the **Table**
3. Select the **Record**
4. Choose a journal field tab
5. Select entries
6. Pick an action
7. Review confirmation modal and provide a reason
8. Confirm

---

## Embedded mode, options, and URL params

The widget supports three ways to provide the target table and record:

### Precedence
1. **Widget instance options** (if provided)  
2. **URL parameters** (if no options provided)  
3. **Manual selection** (if neither options nor URL params provide values)

### Widget instance options

You can hide either picker and or preseed the table and record.

Supported options:
- `hide_table_picker` (true false)
- `hide_record_picker` (true false)
- `table` (table name, e.g. `incident`)
- `sys_id` (record sys_id)
- `record` (alias for sys_id)

Example widget instance options:
```json
{
  "hide_table_picker": true,
  "hide_record_picker": true,
  "table": "incident",
  "sys_id": "57af7aec73d423002728660c4cf6a71c"
}
