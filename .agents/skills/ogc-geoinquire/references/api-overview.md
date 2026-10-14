# Geo-INQUIRE API Overview

Base URL:

`https://voice.pi.ingv.it/geoinquire`

The service is a pygeoapi deployment exposing OGC API Processes. Use the live API as the source of truth whenever possible.

For audited source precedence and known upstream inconsistencies, see `documentation-audit.md`.

## Important Endpoints

| Purpose | Method | URL |
| --- | --- | --- |
| Landing page | GET | `/` |
| Conformance | GET | `/conformance?f=json` |
| OpenAPI document | GET | `/openapi?f=json` |
| Process list | GET | `/processes?f=json` |
| Process description | GET | `/processes/{processId}?f=json` |
| Execute process | POST | `/processes/{processId}/execution?f=json` |
| Job list | GET | `/jobs?f=json` |
| Job status | GET | `/jobs/{jobId}?f=json` |
| Job results | GET | `/jobs/{jobId}/results?f=json` |
| Cancel/delete job | DELETE | `/jobs/{jobId}` |

## Current Processes

As of the latest documentation audit, the service exposes:

| ID | Title | Version | Job control |
| --- | --- | --- | --- |
| `solwcad` | SOLWCAD | `1.0.1` | sync and async |
| `conduit` | CONDUIT | `2.2.0` | sync and async |
| `pybox` | PYBOX | `1.0.0` | async only |

Confirm this with `/processes?f=json` before relying on it.

## Execute Request Shape

OGC API Processes execution uses a JSON body with `inputs` and optional `outputs`.

Scalar and array inputs are sent directly:

```json
{
  "inputs": {
    "scalar_input": 1.0,
    "array_input": [1, 2, 3]
  }
}
```

The published OGC execute schemas have strict-validation ambiguities. A direct array matches both `inlineOrRefData.yaml` and the separate array branch. A JSON integer also matches both the `number` and `integer` branches in `inputValueNoObject.yaml`. The live service examples and process schemas use these values normally. Treat these as schema conflicts and report them; do not rewrite the request merely to silence the generic validator.

Object-valued inputs use the OGC qualified input form:

```json
{
  "inputs": {
    "object_input": {
      "value": {
        "property": "value"
      }
    }
  }
}
```

Do not send an arbitrary object directly as a top-level input value. A direct object is interpreted as a qualified input or link and must contain `value` or `href`.

Explicit output shape:

```json
{
  "inputs": {
    "input_name": "input_value"
  },
  "outputs": {
    "output_name": {
      "transmissionMode": "value"
    }
  }
}
```

Use `transmissionMode: "value"` for direct values when practical. Use `transmissionMode: "reference"` for large or binary outputs when supported.

Always generate `outputs` as an object. The INGV plugin also accepts an output-name array as a local extension, but that form does not conform to the published OGC `execute.yaml` schema.

The shared plugin currently rejects requests where every requested output uses `reference`. Request at least one output using `value`.

## Example Discovery

Examples are stored inconsistently in live process descriptions:

- CONDUIT: plural `examples`
- SOLWCAD: plural `examples`
- PYBOX: singular `example`

Inspect both fields. The generated HTML pages do not display these examples, and the generated OpenAPI currently includes only the PYBOX singular field.

## Execution Modes

Use the `Prefer` header to express the desired execution behavior:

| Header | Meaning |
| --- | --- |
| `Prefer: respond-sync` | Request synchronous execution if supported |
| `Prefer: respond-async` | Request asynchronous job creation |

`pybox` is async-only. For async execution, expect a job resource and poll `/jobs/{jobId}?f=json` until status is terminal. Preserve the `/geoinquire` base path; example URLs that use `https://voice.pi.ingv.it/jobs/...` return `404`.

Common terminal statuses include:

- `successful`
- `failed`
- `dismissed`

## Result Handling

Results may be:

- JSON arrays,
- chart JSON objects,
- text/plain strings,
- text/csv strings,
- GeoTIFF/binary outputs,
- multipart responses with multiple artifacts.

Do not assume `/results?f=json` always returns a simple JSON object. If parsing fails, inspect headers and preserve the raw response.

Do not rely exclusively on the OpenAPI operation's `200` response schema. It is incomplete for CONDUIT and PYBOX. Determine expected outputs from the live process description, then inspect the actual response `Content-Type`.

## Preflight Validation Checklist

Before execution:

- Confirm selected process.
- Fetch live process description.
- Confirm required inputs exist.
- Validate JSON Schema constraints.
- Resolve `oneOf` branches.
- Check enum values.
- Check minimum, maximum, exclusiveMinimum, and exclusiveMaximum.
- Check array minItems and maxItems.
- Check units.
- Check percent versus fraction ambiguity.
- Check object-valued inputs are wrapped in `value`.
- Treat the published OGC array and integer `oneOf` errors as known schema ambiguities, not as proof that a process value is invalid.
- Check `outputs` is an object and includes at least one `value` transmission.
- Check implementation constraints recorded in the relevant process reference.
- Check whether output choices are appropriate for text, charts, CSV, or binary artifacts.
- Ask the user for missing values.
- Ask for explicit confirmation before POST.

## Suggested Curl Patterns

Describe a process:

```bash
curl -sS 'https://voice.pi.ingv.it/geoinquire/processes/conduit?f=json'
```

Execute asynchronously:

```bash
curl -sS \
  -H 'Content-Type: application/json' \
  -H 'Prefer: respond-async' \
  -d @payload.json \
  'https://voice.pi.ingv.it/geoinquire/processes/conduit/execution?f=json'
```

Poll a job:

```bash
curl -sS 'https://voice.pi.ingv.it/geoinquire/jobs/{jobId}?f=json'
```

Fetch results:

```bash
curl -sS 'https://voice.pi.ingv.it/geoinquire/jobs/{jobId}/results?f=json'
```
