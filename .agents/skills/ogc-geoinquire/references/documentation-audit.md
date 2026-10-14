# Geo-INQUIRE Documentation Audit

Audit date: 2026-06-11

This file records source precedence, example validation, and known upstream inconsistencies. It prevents the skill from silently treating every published example as authoritative.

## Source Precedence

When sources disagree, use this order and report material conflicts:

1. Live process description for current input/output IDs and JSON Schema constraints.
2. Published OGC API Processes execute schemas for the request envelope.
3. Actual response headers and bodies for result handling.
4. INGV process-plugin source for implementation constraints and known extensions.
5. Scientific code/manuals for model behavior not represented in the API schema.
6. Embedded examples as evidence, not as defaults.
7. Generated HTML and OpenAPI response hints, which are currently incomplete.

Do not use this precedence to settle scientific ambiguity. Ask the user or an expert scientist.

## Audited Sources

### Live service

- `https://voice.pi.ingv.it/geoinquire?f=json`
- `https://voice.pi.ingv.it/geoinquire/conformance?f=json`
- `https://voice.pi.ingv.it/geoinquire/openapi?f=json`
- `https://voice.pi.ingv.it/geoinquire/processes?f=json`
- all three process descriptions in JSON, JSON-LD, and HTML
- jobs and sampled successful job results

### Standards

- OGC API Processes Part 1 execute, input, output, and transmission schemas under `https://schemas.opengis.net/ogcapi/processes/part1/1.0/openapi/schemas/`

### Implementation

- `https://github.com/francescoingv/ingv-pygeoapi-processing-platform`
- `https://github.com/francescoingv/ingv-pygeoapi-process-plugins`
- `https://github.com/francescoingv/generic-processor-provider`
- `https://github.com/silviagians/PyBOX-Web`

### Scientific resources

- process-linked CIV pages, manuals, DOI records, and publications were catalogued
- PYBOX reference/user guide v0.9 was inspected for input meaning
- PYBOX-Web v1.0.0 source and README were inspected
- PYBOX live metadata exposes the PyBOX-Web release DOI `10.5281/zenodo.18920969` and author ORCID metadata

The full CONDUIT and SOLWCAD scientific literature has not been scientifically adjudicated by this audit. Parameter interpretation and normalization decisions remain subject to expert review.

## Example Inventory

| Process | Metadata field | Entries | Distinct request behavior |
| --- | --- | ---: | --- |
| CONDUIT | `examples` | 5 | qualified object inputs; output object and non-standard output-list forms; mixed value/reference |
| SOLWCAD | `examples` | 4 | qualified `swinput.data`; direct `sw.data` array |
| PYBOX | `example` | 5 | direct scalar/array inputs; output object and non-standard output-list forms; qualified scalar marked unsupported |

Inspect both singular `example` and plural `examples`. Generated HTML exposes neither, and generated OpenAPI currently includes only PYBOX's singular field.

## Validation Findings

All documented example input values pass their respective live process schemas after qualified inputs are unwrapped for process-level validation.

Service-compatible request forms:

- CONDUIT qualified object inputs plus output object
- CONDUIT mixed value/reference output object
- SOLWCAD qualified object plus the direct array form used by the service
- PYBOX direct scalar/array inputs plus output object, as used by the service

Strict-schema caveat:

- The published OGC `execute.yaml` defines a direct inline array and a separate array-of-inline-values branch under `oneOf`. Because `inputValueNoObject.yaml` already includes `type: array`, a direct array matches both branches. That schema also places `number` and `integer` in the same `oneOf`, although JSON integers satisfy both. Strict JSON Schema validation therefore fails for ordinary service-compatible inputs. SOLWCAD exposes the array overlap, while PYBOX can expose the integer/number overlap before its array is checked.

Local extension:

- CONDUIT and PYBOX output-name arrays are accepted by plugin code but violate the published OGC execute schema. The skill must not generate them.

## Known Upstream Inconsistencies

1. CONDUIT and SOLWCAD use `examples`; PYBOX uses `example`.
2. Generated HTML pages do not display process examples.
3. Only PYBOX examples appear in generated OpenAPI.
4. Some examples use output arrays although `execute.yaml` defines an output object.
5. Published job-status/result examples omit `/geoinquire` and return `404`.
6. PYBOX labels a qualified scalar `value` example as unsupported even though it is structurally valid OGC.
7. The shared plugin cannot format an all-reference response.
8. PYBOX plugin source appears to assign a referenced `spatial_evolution` URL under the `invasion_map` key.
9. PYBOX plugin-level JSON Schema validation is commented out; client-side validation is required.
10. PYBOX enforces `sum(eps0) < 1` and an initial-density condition outside the published schema.
11. SOLWCAD source metadata is mistakenly titled as a Conduit schema.
12. SOLWCAD example row 2 oxide fractions sum to `0.9599`; do not silently normalize.
13. CONDUIT example oxide fractions sum to `0.9985`, likely rounding; do not silently normalize.
14. CONDUIT and PYBOX OpenAPI `200` response schemas do not represent all declared outputs or multipart responses.
15. The published OGC execute schemas contain overlapping `oneOf` branches for direct arrays and for integers versus numbers; strict validators can reject service-compatible SOLWCAD and PYBOX values.

## Operational Rules Derived From The Audit

- Wrap object-valued inputs in `value`; send scalar and array inputs directly unless a process-specific reference says otherwise.
- Report the known OGC array/integer `oneOf` ambiguities instead of altering service-compatible values automatically.
- Generate output objects, never output-name arrays.
- Include at least one value-transmitted output.
- Preserve `/geoinquire` in job URLs.
- Validate against live process schemas before requesting confirmation.
- Apply process-specific runtime checks recorded in the references.
- Trust actual response `Content-Type` and process output metadata over incomplete OpenAPI response hints.
- Surface scientific composition anomalies and request expert/user direction.
