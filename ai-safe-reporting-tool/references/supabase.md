# Supabase implementation

Use this reference only when the Host already uses or explicitly selects Supabase. Adapt names to the project's conventions.

## Place responsibilities

Use Supabase Edge Functions for server-side chat, generation, and Dataset execution:

```text
report-chat       requirements gathering
report-generate   AIRspec generation, validation, and persistence
data-broker       authorized Dataset execution
```

Keep provider keys and backend credentials in Edge Function secrets. Resolve the authenticated viewer in each function, then load only the catalog entries, reports, versions, and Datasets available to that viewer.

Keep raw Supabase calls inside the project's service and repository layers. UI components call domain operations such as `sendChatMessage`, `generateReport`, `loadReportVersion`, and `executeDataset`.

## Organize persistence

Use tables equivalent to:

```text
report_generation_sessions
report_generation_messages
reports
report_versions
```

Associate every row with the project's ownership or tenant identity. Enable row-level security and define policies that match the application's authenticated roles and ownership model. Store accepted AIRspec documents in immutable `report_versions`; keep `reports.current_version_id` as the mutable pointer.

Persist generation metadata separately from the document, including model, provider request ID, attempt count, validation result, and schema version. Store machine-readable validation errors for failed attempts when the product requires audit history.

## Implement report chat

In `report-chat`:

1. Authenticate the caller and load the authorized session.
2. Load the viewer-appropriate Source Catalog and Host capabilities.
3. Construct the requirements-gathering context.
4. Call the configured model adapter.
5. Validate the typed chat response.
6. Persist the new messages and requirements state.
7. Return the typed chat response.

## Implement report generation

In `report-generate`:

1. Authenticate the caller and load the authorized session or report.
2. Load the current schema, Source Catalog, Host limits, and examples.
3. Generate a candidate through the configured model adapter.
4. Run the bounded validation-and-retry workflow.
5. Persist the accepted document as a new immutable version.
6. Update the report's current-version pointer transactionally.
7. Return the report and version IDs through the chosen response or stream protocol.

Use a database function or transaction-capable server path for version numbering and current-version updates when concurrent generations are possible.

## Implement the Data Broker

In `data-broker`, accept report version ID, Dataset ID, validated parameter values, and pagination state. Load the stored version and locate its Dataset server-side. Apply the authorization and execution sequence from the main skill before returning alias-keyed rows.

Scope cache keys and stored results by viewer or tenant authorization context. Reject stale client responses with the Host's state revision.

## Verify Supabase integration

Test Edge Functions with authenticated identities representing allowed and denied access. Verify row-level policies independently from function logic. Exercise concurrent version creation, generation retry exhaustion, Dataset authorization, immutable version reload, and the production build against a non-production Supabase project.
