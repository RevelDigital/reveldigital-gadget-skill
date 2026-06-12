# Data Table Reference (gadgets)

Revel Digital **data tables** are structured row/column data managed in the CMS. Gadgets can read
them **live** — with filtering, sorting, schema introspection, and **real-time row-change events** —
through the `@reveldigital/client-sdk` data-table shim.

> **Gadget-only capability.** The shim wraps the player-injected global
> `window.gadgets['reveldigital.datatable']`. The player provides this library to **gadgets**, not to
> full-screen webapps — so this API works in a gadget but would throw in a webapp. The device's
> registration key is resolved automatically, so **no API key** is embedded in the gadget.

## Enabling the feature

The data-table library is only injected when the gadget requests the feature. Add it to the gadget's
`requirements` in `gadget.yaml`:

```yaml
requirements:
  - reveldigital
  - datatable      # enables window.gadgets['reveldigital.datatable']
```

> **Confirm the exact requirement key** against the current gadget docs — the SDK references a
> "datatable feature," but verify the string (`datatable` vs `reveldigital.datatable`) before relying
> on it.

If the feature isn't enabled (or you're running off-device during development), `createDataTable()`
**throws** — always wrap calls in `try/catch` and degrade gracefully (e.g. show a placeholder).

## Basic usage — fetch rows

```typescript
import { createPlayerClient } from '@reveldigital/client-sdk'

const client = createPlayerClient()

try {
  const dt = client.createDataTable('tbl_menu_items')

  const result = await dt.getRows({ sort: 'price', sortDir: 'asc', pageSize: 50 })
  // result: { data, totalCount, continuationToken, cacheUntil, notModified? }

  for (const row of result.data) {
    // row: { id, sortOrder, data: { …columnKey: value… }, updatedAt }
    console.log(row.data.itemName, row.data.price)
  }
} catch {
  // datatable feature not available (e.g. running off-device) — use a fallback
}
```

Use `getVisibleRows(params?)` for rows with hidden-column values stripped, and `getSchema()` /
`getVisibleColumns()` to introspect the table structure.

## Filtering & sorting

`getRows(params)` accepts `IDataTableQueryParams`:

| Param | Notes |
|-------|-------|
| `filter` | Filter map keyed by column key (see below) |
| `sort` / `sortDir` | Column key + `'asc'` \| `'desc'` |
| `pageSize` | Rows per page (max 100) |
| `continuationToken` | Cursor from a previous result, for the next page |
| `fields` | Comma-separated column keys to include |

**Filter forms:**

```typescript
{ category: 'Entree' }                              // simple equality
{ price: { op: 'lte', value: 25 } }                 // operator expression
{ price: { op: 'inRange', from: 5, to: 20 } }       // range
{ category: 'Entree', price: { op: 'lte', value: 25 } }   // AND (multiple keys)
{ $or: [{ status: 'active' }, { status: 'pending' }] }    // OR group
{ $or: [{ status: 'active' }, { status: 'pending' }], price: { op: 'gt', value: 0 } }  // mixed
```

**Operators:** `eq`, `neq`, `isEmpty`, `isNotEmpty`, `contains`, `notContains`, `gt`, `gte`, `lt`,
`lte`, `positive`, `negative`, `inRange`, `outOfRange`, `beforeNow`, `afterNow`, `isToday`.

## Real-time updates

Keep the gadget in sync as the CMS data changes — **without redeploying**:

```typescript
// 1. Push events (SignalR)
dt.on('rowUpdated', (change) => { /* change: { tableId, rowId, data?, action } */ })
dt.on('rowCreated', (change) => { /* … */ })
dt.on('rowDeleted', (change) => { /* … */ })

// 2. Polling (fallback, or in addition)
dt.startPolling(30000)   // ms; default 30000
// dt.stopPolling()
```

On any change, re-query (`getRows`) and re-render. Disable real-time entirely by passing
`{ signalRUrl: null }` to `createDataTable`.

## Driving a table from a gadget preference

Let signage designers pick the table (and filter/sort) at design time via a `datatable` preference,
then bind to their choice with `createDataTableFromPref`:

```yaml
# gadget.yaml — confirm the exact datatype name against the current gadget docs
prefs:
  - name: rdDataTable
    display_name: Data source
    datatype: datatable
```

```typescript
const prefs = client.getPrefs()
const cfg = client.createDataTableFromPref(prefs.getString('rdDataTable'))

const result = await cfg.getFilteredRows()   // filter + sort from the preference applied

cfg.dataTable.on('rowUpdated', (change) => { /* … */ })   // underlying DataTableRef
cfg.dispose()
```

## Cleanup

Always release resources on teardown (stops polling, closes the real-time connection, removes
listeners):

```typescript
dt.dispose()   // or cfg.dispose() for a pref-bound table
```

Call this in React's `useEffect` cleanup, Vue's `onUnmounted`, or Angular's `ngOnDestroy`.

## Options

`createDataTable(tableId, options?)` / `createDataTableFromPref(prefValue, options?)` accept
`IDataTableOptions`:

- `registrationKey?` — override the auto-resolved device key.
- `baseUrl?` — override the API base URL.
- `signalRUrl?` — real-time hub URL; set to `null` to disable real-time updates.

## Type reference

```typescript
import type {
  DataTableRef, DataTablePrefRef,
  IDataTableQueryParams, IDataTableResult, IDataTableRow,
  IDataTableSchema, IDataTableColumn, IDataTableChangeEvent,
  IDataTableFilter, IDataTablePref,
} from '@reveldigital/client-sdk'
```

Key shapes:

- **`IDataTableRow`** — `{ id: string; sortOrder: number; data: { [key]: any }; updatedAt: string }`
- **`IDataTableResult`** — `{ data: IDataTableRow[]; totalCount: number; continuationToken: string | null; cacheUntil: string; notModified?: boolean }`
- **`IDataTableColumn`** — `{ id; name; key; type: 'text'|'number'|'boolean'|'date'|'image'|'hidden'; required; sortable; options }`
- **`IDataTableChangeEvent`** — `{ tableId; rowId; data?; action: 'update'|'create'|'delete'|'poll' }`

## Angular note

Angular gadgets use `@reveldigital/player-client`. Confirm whether that library exposes equivalent
`createDataTable` methods on `PlayerClientService`; if not, use the data-table shim via the global
`gadgets['reveldigital.datatable']` directly, or add `@reveldigital/client-sdk` for this feature.
