# Chat-driven report generation

Use this reference when a Host includes conversational requirements gathering, AI document generation, model selection, or generation progress. Keep this workflow separate from Dataset execution: the AI produces declarative report structure, while the Host validates, authorizes, executes, and renders it.

## Separate the two AI operations

Implement two server-side operations:

```text
requirements chat → structured requirements
structured requirements → validated AIRspec document
```

The requirements operation asks focused questions and summarizes the requested report. The generation operation creates AIRspec JSON only after requirements are ready. Keep their prompts, response types, limits, logs, and tests independent.

## Own chat state

Store these values in one chat controller or framework-equivalent state module:

- stable session ID;
- ordered user and assistant messages;
- readiness state and structured requirements summary;
- selected model capability or configured model ID;
- chat, generation, and persistence status;
- generated report and version IDs;
- recoverable error and progress state.

Use an explicit response contract instead of parsing prose:

```json
{
  "message": "Which date range should this report cover?",
  "status": "gathering",
  "requirements": null
}
```

When ready, return `status: "ready"` and a structured requirements object. Let the UI reveal generation only from this typed state.

## Gather requirements from allowed capabilities

Build the requirements assistant's context from:

- the viewer-appropriate Source Catalog;
- registered routes and available interactions;
- Host component and export capabilities;
- target conformance class;
- presentation preferences supplied by the user.

Ask one to three useful questions at a time. Preserve exact logical source and field IDs from the catalog. Summarize ambiguous decisions for confirmation before generation.

## Generate on the server

Run document generation behind a server endpoint or serverless function. Give it:

- the current versioned AIRspec JSON Schema;
- the viewer-appropriate Source Catalog;
- Host limits and supported conformance class;
- concise AIRMark vocabulary guidance;
- a small set of current, valid examples;
- the structured requirements;
- the current validated document when editing an existing report.

Prefer provider-supported structured output constrained by the schema. Treat generated output as untrusted until it passes all applicable validation layers.

## Validate and retry

Use a bounded loop:

```text
generate candidate
→ parse JSON
→ validate schema and semantics
→ validate catalog references and authorization
→ validate AIRMark and Host capabilities
→ return machine-readable errors to the generator
→ retry up to the configured limit
```

Accept and persist only a candidate that passes the required layers. Return a typed failure containing the final validation errors when the retry limit is reached.

Keep validation errors structured:

```json
{
  "layer": 2,
  "code": "unknown_field",
  "path": "/datasets/0/dimensions/0/field",
  "message": "Field 'region_name' is not in source 'sales'.",
  "suggestion": "Use 'region'."
}
```

## Support model choice through adapters

Define one internal provider interface and keep provider request formats inside adapters:

```text
generate({ model, system, messages, responseSchema, signal })
→ { text, usage, providerRequestId }
```

Load allowed model IDs and defaults from server configuration. Select models by declared capabilities such as structured output, context size, and vision rather than embedding provider-specific behavior in UI components.

Keep provider credentials server-side. Record the selected provider, model, latency, token usage, retry count, schema version, and validation result with each generation version.

## Stream progress with a typed protocol

For long generations, stream newline-delimited JSON or the project's established event protocol:

```text
{ "type": "progress", "stage": "generating", "attempt": 1 }
{ "type": "progress", "stage": "validating", "attempt": 1 }
{ "type": "complete", "reportId": "...", "versionId": "..." }
```

Define `progress`, `complete`, and `error` events as a closed union. Parse partial chunks with a buffer, support cancellation, and close every stream deterministically.

## Persist immutable versions

Keep chat sessions, reports, and report versions distinct. Store each accepted document as an immutable version with:

- report and session identity;
- AIRspec schema version;
- requirements summary;
- validated document;
- provider and model metadata;
- generation and validation metadata;
- creator and creation time.

Point the report record at its current version. Load the stored version on the server before Dataset execution.

## Connect generation to rendering

After generation succeeds:

1. Load the persisted version through the application service layer.
2. Validate it again at the Host boundary.
3. Initialize parameter defaults and bindings.
4. Fetch Dataset rows through the server-side Data Broker.
5. Render through the trusted component registry and AIRMark engine.

Keep chat state and runtime report state separate so regenerating a report does not leak stale parameters, selections, or Dataset responses into the new version.

## Verify the workflow

Test:

- gathering, ready, generating, retrying, complete, cancelled, and failed states;
- structured readiness and requirements preservation;
- model adapter selection and configuration;
- malformed JSON and validation feedback retries;
- retry exhaustion without persistence;
- immutable version creation and reload;
- progress-stream chunk boundaries and cancellation;
- authorization changes between chat, generation, and execution;
- a generated document through the complete validation-to-render path.
