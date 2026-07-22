# Framework-agnostic implementation

Use this path for Vue, Svelte, Angular, vanilla JavaScript, Canvas, server-rendered SVG, native mobile, desktop, or non-JavaScript implementations.

## Module boundaries

Implement separate modules for:

```text
types
schema validator
semantic validator
authorization validator
AIRMark validator
runtime validator
parameter store
binding resolver
Dataset client and cache
server-side Data Broker
source adapters
component registry
layout walker
interaction dispatcher
chart scene-graph adapter
```

Do not combine validation, fetching, state mutation, and drawing in one module.

## State contract

Track immutable document/version identity, validated parameter values, resolved bindings, Dataset request states, selection state, and a monotonically increasing revision.

Build Dataset cache keys from version ID, Dataset ID, normalized affecting parameters, pagination, and viewer scope. Coalesce requests and discard responses from an older revision.

Apply multi-action interactions atomically, resolve dependencies once, and execute each affected Dataset at most once per revision.

## Platform rendering

The scene graph contains positioned `group`, `rect`, `line`, `path`, `circle`, and `text` nodes. Walk this tree and draw only allowlisted primitives. Reject unknown node types.

Use node metadata as data:

- `meta.tooltip`: already formatted label/value entries;
- `meta.selection` and `meta.fields`: interaction routing;
- `meta.datum`: the trusted broker row represented by the mark;
- `plot`: computed inner plot bounds for overlays and sibling alignment.

Keep drawing thin. Geometry, axes, ticks, scales, formatting, and tooltip content belong in the layout engine, not each platform adapter.

## Required platform tests

- Render a wide chart in a short cell and prove no node escapes its measured box.
- Resize the cell and prove the engine re-runs instead of scaling old geometry.
- Render and escape native or custom tooltip content.
- Prove a missing encoded field on any row throws a named contract error.
- Compare supported AIRMark cases against upstream golden fixtures.
- Prove unsupported scene nodes fail explicitly.
