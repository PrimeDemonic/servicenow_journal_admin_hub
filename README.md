# ServiceNow Journal Admin Hub (JAHBLESS)

A guided ServiceNow UI for deleting or redacting journal entries (Additional comments / Work notes / other `journal_input` fields) with audit-aligned handling.

> ⚠️ Disclaimer  
> This is provided for learning and experimentation.  
> Do not deploy to production without thorough testing in your own environments, validation of policy requirements, and stakeholder approval.

---

## What it does

- Browse and manage journal entries for a specific record
- Works with any table that has `journal_input` fields (including inherited tables)
- Shows journal fields as tabs (newest entries first)
- Multi-select one or more journal entries
- Perform one of:
  - Delete entry
  - Redact in full
  - Redact specific text (per-entry target text)
- Requires a reason and presents confirmation UX showing exactly what will be affected
- Writes a server-side log entry for each journal entry action

---

## Architecture

This solution uses two parts:

1. **Scoped app (x_cros2_journal_0)**
   - UI (Service Portal widget + page)
   - Table and record selection
   - Journal field discovery and entry browsing
   - Calls into the global executor for mutations

2. **Global Script Include**
   - Executes the actual update/delete operations against system tables (`sys_journal_field`, `sys_audit`, and history refresh)
   - The reason for this approach is to not have to change the core config of your `sys_` tables
   - Returns success/failure with best-effort error messaging

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
The app supports these `sys_properties`:

- `allowed_roles`
  - Comma-separated roles allowed to use the tool
  - Empty means no restriction
- `use_secure_lookup`
  - `true` uses `GlideRecordSecure` for lookups and filters journal fields by field-level read access (best effort)
  - `false` uses `GlideRecord`

> Tip: Start with `use_secure_lookup=true` in most environments unless you have a strong reason not to.

---

## Usage

### Standard usage (pickers visible)

1. Open **Journal Admin Hub** module
2. Select the **Table**
3. Select the **Record**
4. Choose a journal field tab
5. Select one or more entries
6. Pick an action
7. Review the confirmation UX and provide a reason
8. Confirm

---

## Embedding and Inputs (Options + URL parameters)

The widget supports being embedded inside other pages/widgets where the `table` and `record` are already known. In this mode you can:

- Hide the table picker
- Hide the record picker
- Pass `table` and `sys_id` as widget options
- Or pass them via URL query parameters

### Precedence rules

The widget resolves its initial `table` and `record` using this logic:

1. **If widget options provide values, use those**
2. **Else if URL parameters provide values, use those**
3. **Else show pickers and allow user selection**

This allows you to:
- Hard-wire a specific record in a widget instance (options)
- Or build a reusable embedded page that drives the widget via URL

---

## Widget Options

Add these in the widget instance **Options** JSON.

### Options reference

| Option | Type | Default | Description |
|---|---:|---:|---|
| `hide_table_picker` | boolean | `false` | Hides the table picker and shows a read-only table display instead |
| `hide_record_picker` | boolean | `false` | Hides the record picker and shows a read-only record display instead |
| `table` | string | `""` | Table name to preselect, e.g. `incident` |
| `sys_id` | string | `""` | Record sys_id to preselect |
| `record` | string | `""` | Alias for `sys_id` (supported for convenience) |

### Example: fully embedded (both pickers hidden)

```json
{
  "hide_table_picker": true,
  "hide_record_picker": true,
  "table": "incident",
  "sys_id": "57af7aec73d423002728660c4cf6a71c"
}
```

### Example: embedded table, but allow record selection

```json
{
  "hide_table_picker": true,
  "hide_record_picker": false,
  "table": "incident"
}
```

### Example: no embedding, but pre-fill a starting point

```json
{
  "hide_table_picker": false,
  "hide_record_picker": false,
  "table": "incident"
}
```

---

## URL Parameters

If widget options do not provide table and sys_id, the widget will look for URL parameters.
Supported URL parameters

  table
  Example: incident

  record (preferred)
  Example: 57af7aec73d423002728660c4cf6a71c

  sys_id (alias)
  Example: 57af7aec73d423002728660c4cf6a71c

### Example URLs

  Preselect incident + record:

    ...?table=incident&record=57af7aec73d423002728660c4cf6a71c

  Preselect incident + record (sys_id alias):

    ...?table=incident&sys_id=57af7aec73d423002728660c4cf6a71c

  Only preselect table, allow record picking:

    ...?table=incident

  Note: URL parameters are only used when options do not provide values.

---

## Notes / gotchas

  Journal manipulation is sensitive. Test in sub-prod first.

  Always validate that the journal and audit behavior meets your organisation’s expectations.

  If you use use_secure_lookup=true, table and field visibility will follow user ACLs (best effort).

  Deleting or modifying journal entries can have policy, compliance, and audit implications depending on your organisation.

  Consider restricting access via allowed_roles even in non-production environments.

---

## Repo contents

  /scoped/
  The scoped application update set export

  /global/
  The global Script Include update set export
    
---

## Contributing

This is just a bit of fun I'll probably never look at again :)

---

## License

MIT
