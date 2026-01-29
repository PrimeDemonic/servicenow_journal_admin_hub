# ServiceNow Journal Admin Hub (JAHBLESS)

A guided ServiceNow UI for deleting or redacting journal entries (Additional comments / Work notes / other journal_input fields) with audit-aligned handling.

> ⚠️ Disclaimer  
> This is provided for learning and experimentation.  
> Do not deploy to production without thorough testing in your own environments, validation of policy requirements, and stakeholder approval.

---

## What it does

- Select a table and record via a guided UI
- View all journal fields for that record as tabs
- Select one or more journal entries
- Perform one of:
  - Delete entry
  - Redact in full
  - Redact specific text
- Requires a reason and presents a confirmation modal that shows exactly what will be affected
- Logs a server-side system log entry per journal entry action

---

## Architecture

This solution uses two parts:

1. **Scoped app (x_cros2_journal_0)**
   - UI (Service Portal widget + page)
   - Record picker logic and journal browsing
   - Calls into the global executor for mutations

2. **Global Script Include**
   - Executes the actual update/delete operations against system tables
   - Avoids needing to change core configuration of sys_ tables
   - Returns success/failure with best-effort error messaging

---

## Install

### Step 1: Import the scoped update set
Import the scoped update set XML first.

ServiceNow:
- System Update Sets → Retrieved Update Sets
- Import Update Set from XML
- Upload:
  `update_sets/scoped/Journal_Administration_Hub_Bulk_Log_Entry_Sanitisation_Suite.xml`
- Preview Update Set
- Commit Update Set

### Step 2: Import the global update set
Then import the global update set XML.

ServiceNow:
- System Update Sets → Retrieved Update Sets
- Import Update Set from XML
- Upload:
  `update_sets/global/JAHBLESS_Global.xml`
- Preview Update Set
- Commit Update Set

### Step 3: Configure properties
The app supports these sys_properties:

- `allowed_roles`
  - Comma-separated roles allowed to use the tool
  - Empty means no restriction
- `use_secure_lookup`
  - `true` uses GlideRecordSecure for lookups
  - `false` uses GlideRecord

---

## Usage

1. Open **Journal Admin Hub** module
2. Select the **Table**
3. Select the **Record**
4. Choose a journal field tab
5. Select entries
6. Pick an action
7. Review confirmation modal and provide a reason
8. Confirm

---

## Notes / gotchas

- Journal manipulation is sensitive. Test in sub-prod first.
- Always validate that the journal and audit behavior meets your organisation’s expectations.
- If you use `use_secure_lookup=true`, table and field visibility will follow user ACLs.

---

## Repo contents

- `update_sets/scoped/`  
  The scoped application update set export

- `update_sets/global/`  
  The global Script Include update set export

---

## Contributing

This is just a bit of fun I'll probably never look at again :)

---

## License

MIT
