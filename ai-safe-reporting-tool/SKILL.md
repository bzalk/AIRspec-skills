---
name: ai-safe-reporting-tool
description: "Use when building, integrating, debugging, or reviewing an AI-powered reporting, dashboard, or data-visualization application where AI generates report structure. Covers secure AIRspec report generation, validation, viewer-authorized server-side data execution, trusted rendering, interactions, responsive AIRMark charts, React integration, and conformance testing. Doesn't apply to ordinary dashboards or charts that do not use AI-generated report definitions."
---

# Build AI-safe reporting tools

Implement AIRspec as a complete Host boundary, not merely a chart component. Treat the current [AIRspec repository](https://github.com/bzalk/AIRspec) as normative. For JavaScript and TypeScript hosts, use the published packages from the [AIRMark Engine repository](https://github.com/bzalk/airmark-engine) by default; do not rebuild chart layout or scene-graph logic unless the target platform cannot use them or the project explicitly requires a custom implementation.

## Select the implementation path

Inspect the existing project before changing code.

- For React, read [references/react.md](references/react.md).
- For every other framework, platform, or language, read [references/generic.md](references/generic.md).
- When implementing requirements chat or AI document generation, read [references/chat-generation.md](references/chat-generation.md).
- When the project uses Supabase, read [references/supabase.md](references/supabase.md).
- Always follow [references/conformance.md](references/conformance.md).

Do not create a parallel application when the project already has routing, authentication, state, API, design-system, or test conventions.

## Establish the project brief

Determine these facts from the project or ask for the missing business-specific values:

```yaml
language_and_framework: ""
package_manager_and_commands: ""
client_and_server_runtime: ""
data_sources: ""
authentication_and_authorization: ""
registered_routes: ""
target_conformance_class: "A | AV | AVI"
minimum_generated_airspec_version: "1.1"
first_source: ""
first_report: ""
test_typecheck_lint_build_commands: ""
```

Never invent credentials, permissions, source IDs, field mappings, physical columns, or routes. Continue with independent implementation work when one missing answer blocks only part of the task.

## Preserve the security boundary

Apply these rules without exception:

1. Treat every AIRspec document as untrusted declarative configuration.
2. Validate before storage, rendering, and Dataset execution.
3. Never evaluate document strings as formulas, templates, callbacks, code, property paths, imports, SQL, HTML, or CSS.
4. Reject document-provided URLs, credentials, connection strings, raw queries, inline Dataset execution definitions, and unknown non-extension properties.
5. Execute data only through a server-side Data Broker using server-held credentials and the current viewer's authorization.
6. Give report-generating AI the schema and viewer-appropriate Source Catalog, not credentials or unrestricted business data.
7. Resolve navigation through registered route IDs. Never construct a URL from document content.
8. Fail closed on unsupported vocabulary and fail soft at component boundaries.

## Build one vertical slice first

Implement in this order:

1. Define a Source Catalog with logical source and field names.
2. Add runtime types and structured validation errors.
3. Implement validation Layers 1–5.
4. Implement one server-side source adapter and Data Broker endpoint.
5. Load one immutable AIRspec document.
6. Build the trusted component registry and recursive layout walker.
7. Render one metric, one table, and one chart.
8. Exercise a parameter, visibility condition, empty state, and source error.
9. Add generator context and validate-and-retry only after the Host boundary works.
10. Convert every defect into a regression fixture before fixing it.

## Implement validation as separate layers

Return machine-readable errors containing `layer`, `code`, `path`, `message`, and an optional `suggestion`.

- Layer 1: for stored documents, select the supported versioned Draft 2020-12 JSON Schema from `document.airspec`; for newly generated documents, require the deployment's resolved generation schema and version.
- Layer 2: validate IDs, references, sources, fields, operators, aliases, stable outputs, bindings, derived fields, calculated metrics, components, selections, interactions, and routes.
- Layer 3: apply viewer source/field/classification/limit policy, including derived-field classification roll-up.
- Layer 4: enforce AIRMark's closed vocabulary, Dataset output fields, URL/expression prohibition, and complexity limits.
- Layer 5: validate parameters, broker response shape, row completeness, row limits, and stale state revisions.

TypeScript types or framework prop validation do not replace these runtime layers.

## Keep data execution server-side

Expose an endpoint equivalent to:

```text
POST /api/reports/{versionId}/datasets/{datasetId}/execute
body: { parameters, page }
```

The server must load the immutable stored document and Dataset. Do not accept a Dataset body from the client.

Use this execution order:

```text
authenticate → load version → locate Dataset → authorize viewer/source/fields
→ validate parameters → resolve bindings → structured derivations → filters
→ aggregation → calculated metrics → sort/limit/page → alias-keyed rows
→ remove unauthorized fields → clamp response
```

Map logical catalog fields to physical backend fields only inside source adapters. Compile structured arithmetic through fixed operators; never concatenate document strings into executable backend expressions. Reject incomplete rollup/total rows.

## Render through trusted code

Use a closed component registry for `stack`, `grid`, `section`, `heading`, `text`, `divider`, `metric`, `table`, `filterBar`, `emptyState`, and `chart`.

Evaluate `visibleWhen` before rendering a node. Containers recurse through the layout walker. Unknown types render a safe diagnostic. Wrap data components in local error boundaries. Provide loading, empty, invalid, authorization, source-error, and success states.

Never turn document values into component names, modules, HTML, class names, or arbitrary styles.

## Render AIRMark at exact pixels

For JavaScript or TypeScript, install compatible current releases from the public npm registry with the project's package manager:

```bash
npm install @airspec/airmark-engine
npm install @airspec/airmark-react       # React hosts
npm install @airspec/airmark-svg         # SVG output or export
```

Use the equivalent command for pnpm, Yarn, Bun, or the project's established package manager. Use the npm package `@airspec/airmark-engine` for deterministic layout, the framework adapter that matches the host, and `@airspec/airmark-svg` for SVG output or export. Do not replace these packages with copied source, a Git dependency, or host-specific chart geometry unless the project explicitly requires it. Build a custom adapter only when an existing adapter cannot target the required rendering platform.

For other languages, implement the AIRMark Scene Graph contract and verify it against the upstream golden fixtures.

Always use:

```text
validated graphic + complete alias-keyed rows + measured content box + resolved theme
→ deterministic scene graph → thin platform renderer
```

Layout pixels must equal display pixels. Measure the real content box and re-layout on resize. Never CSS-stretch an SVG calculated at another size. Display preformatted `meta.tooltip` entries without reformatting them. Dispatch selections through declared AIRspec interactions only.

## Resolve and lock the generation version

Resolve the current AIRspec generation version during implementation, dependency updates, or deployment—not independently inside each generation request:

1. Read `AIRspec.md` from the default branch of the normative AIRspec repository.
2. Extract its current **Schema ID convention** URL and fetch that versioned schema.
3. Read the schema's `properties.airspec.const`; treat that value as the current document version.
4. Verify the resolved version is at least `1.1`. Stop the update if it is older, unavailable, or internally inconsistent with the schema `$id`.
5. Store the resolved version, schema URL, and schema content hash in deployment configuration or a checked dependency lock artifact.
6. Refresh this lock deliberately when adopting a newer AIRspec release.

Use the locked schema and version for every newly generated document, prompt, structured-output constraint, example, retry, and generation gate. The AI does not select its AIRspec version. Reject a generated candidate whose `airspec` value differs from the locked generation version, even when an older schema remains supported for reading existing documents.

## Integrate the generator last

Provide the locked generation schema and version, Source Catalog, Host limits, concise vocabulary guidance, and examples validated against that same schema. Prefer constrained structured output when available. Validate the result against the locked schema, return machine-readable errors, and retry with a strict cap.

Require the generator to use:

- logical source IDs and real catalog fields;
- declared aliases for every metric and dimension output;
- structured `derived.expr` for row-level arithmetic;
- calculated metrics for ratios of aggregates;
- identical explicit domains and `scale.reverse` for mirrored pairs;
- titled, formatted tooltip arrays for hover-oriented charts.

Do not expose credentials or unrestricted data tools to the generator.

## Finish with evidence

Do not claim completion because a sample renders. Run the conformance runner against the implementation, execute broker fixtures, test component failure states, verify exact chart sizing, and run the project's tests, type checks, lint, and production build.

At handoff, report files changed, commands and results, supported conformance class, unsupported vocabulary, and the next recommended fixture. Do not claim support for untested behavior.
