# Conformance and verification

Use the current files from [bzalk/AIRspec](https://github.com/bzalk/AIRspec):

- `AIRspec.md`
- `schema/`
- `IMPLEMENTATION.md`
- `conformance/manifest.json`
- `conformance/valid/`
- `conformance/invalid/`
- `conformance/broker/`
- `samples/`

## Validation evidence

Run:

```bash
python conformance/runner.py --cmd "<validator command>"
```

The command receives document, catalog, and policy paths and must exit zero to accept or nonzero to reject. Layer-2 and Layer-3 fixtures intentionally pass JSON Schema and require the Host validator.

Execute broker fixtures separately through the actual source-adapter boundary and deep-compare returned rows with `expectedRows`.

## Required security tests

Prove the Host rejects:

- unknown sources and fields;
- unauthorized fields and excessive limits;
- URLs, raw queries, credentials, HTML, and expression strings;
- identifier-encoded formulas;
- malformed or cyclic structured derivations;
- unstable binding output contracts;
- unregistered routes and missing interaction targets;
- incomplete Dataset rows, including rollup/total rows;
- stale responses from prior parameter revisions.

## Required rendering tests

Prove:

- unknown components degrade safely;
- one component failure does not remove siblings;
- loading, empty, authorization, and source errors are distinct;
- charts use exact measured dimensions and re-layout on resize;
- tooltips use deterministic formatted metadata and escape output;
- mirrored chart halves use identical explicit domains;
- supported AIRMark cases match upstream fixtures and goldens.

## Completion report

Report:

1. supported AIRspec version and conformance class;
2. passing conformance and broker case counts;
3. test, type-check, lint, and build commands with results;
4. explicitly unsupported vocabulary;
5. next fixture needed for the following capability.
