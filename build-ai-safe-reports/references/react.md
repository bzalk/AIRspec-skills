# React implementation

Use the project's existing router, query/state library, design system, error boundaries, and server architecture. Do not create a parallel React application.

## Suggested structure

```text
airspec/
  types.ts
  validation/{schema,semantic,authorization,airmark,runtime}.ts
  state/{ReportProvider,parameters,bindings,interactions}.tsx
  data/{datasetClient,cacheKey}.ts
  renderer/
    ReportRenderer.tsx
    LayoutWalker.tsx
    componentRegistry.ts
    ComponentBoundary.tsx
    components/
server/dataBroker/{executeDataset,sourceAdapters}/
```

Adapt paths to existing conventions. Reuse equivalent modules.

## Trusted registry

Use a compile-time registry such as:

```tsx
const componentRegistry = {
  stack: AirStack,
  grid: AirGrid,
  section: AirSection,
  heading: AirHeading,
  text: AirText,
  divider: AirDivider,
  metric: AirMetric,
  table: AirTable,
  filterBar: AirFilterBar,
  emptyState: AirEmptyState,
  chart: AirChart,
} satisfies Record<SupportedComponentType, React.ComponentType<AirComponentProps>>;
```

`LayoutWalker` evaluates visibility, selects the trusted component, and renders a safe diagnostic for unknown types. Containers invoke the same walker for children. Use per-component error boundaries.

## Report state

Integrate with existing state. Own:

- immutable document/version ID;
- validated parameters and defaults;
- resolved Dataset and graphic bindings;
- Dataset loading/error/data state;
- selection state;
- deduplication and cache keys;
- atomic interaction dispatch;
- state revisions and stale-response rejection.

Avoid one state variable per component and request effects that can loop. Prefer normalized state and pure resolvers.

## Chart component

Install and use `@airspec/airmark-engine` with `@airspec/airmark-react`; do not implement chart geometry, scales, axes, marks, or tooltip formatting in React components. Use `AirmarkChartAuto` inside report grid cells:

```tsx
<AirmarkChartAuto
  graphic={validatedGraphic}
  rows={brokerRows}
  theme={resolvedTheme}
  selectionState={selectionState}
  transitionMs={prefersReducedMotion ? undefined : 250}
  onSelect={handleSelect}
/>
```

Requirements:

- Let the card/grid determine the available box.
- Do not copy a hardcoded `height: 360px` unless the Host deliberately resolves that exact height.
- Do not stretch a differently sized SVG with `width: 100%`.
- Give the container a meaningful minimum height and `overflow: hidden` as a safety net.
- Respect `prefers-reduced-motion`.
- Use engine-generated native tooltips or read the same `meta.tooltip` entries for a styled overlay.
- For exports, use the same deterministic scene graph with `@airspec/airmark-svg`.

## React tests

- Test loading, empty, invalid, unauthorized, source-error, and success states.
- Test unknown components and local error boundaries.
- Test parameter defaults, visibility, bindings, atomic actions, request deduplication, and stale responses.
- Mount a span-12 chart in a wide, short cell and assert the SVG remains inside it.
- Test native tooltip text, formatting, and escaping.
- Test reduced motion and prove transitions do not alter scene-graph output.
- Run unit tests, type checking, lint, and the production build.
