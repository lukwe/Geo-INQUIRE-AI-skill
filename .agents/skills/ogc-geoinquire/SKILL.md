---
name: ogc-geoinquire
description: Use when a user wants an LLM or agent to inspect, prepare, validate, or execute Geo-INQUIRE OGC API Processes at voice.pi.ingv.it/geoinquire, including SOLWCAD, CONDUIT, and PYBOX. The skill enforces preflight review, missing-parameter questions, explicit user confirmation before execution, async job polling, and result summarization with artifact preservation.
---

# Geo-INQUIRE OGC API Processes

Use this skill to help a user work with the Geo-INQUIRE pygeoapi service:

`https://voice.pi.ingv.it/geoinquire`

The service exposes OGC API Processes for volcanology models. Treat it as a scientific execution service, not as a casual lookup API.

## Core Rules

- Never execute a process immediately from a natural-language request.
- Always perform a preflight review before execution.
- Always list the candidate process, endpoint, execution mode, input payload, units, inferred values, missing values, and validation concerns.
- Ask for missing required values before asking for execution confirmation.
- Do not invent scientific inputs. If pressure, temperature, composition, geometry, location, particle classes, or model mode are missing, ask the user.
- You may suggest example values only when clearly labeled as examples, not defaults.
- POST to `/execution` only after the user explicitly confirms.
- Prefer live process descriptions over embedded reference notes whenever network access is available.
- Preserve raw outputs and artifacts. Do not replace GeoTIFF, CSV, chart JSON, or text outputs with only a prose summary.
- Avoid scientific overinterpretation. Explain what the model returned, units, and obvious structure; state uncertainty when domain interpretation would require expert judgment.

## References

Load only the files needed for the current task:

- API workflow and request format: `references/api-overview.md`
- SOLWCAD input/output guide: `references/process-solwcad.md`
- CONDUIT input/output guide: `references/process-conduit.md`
- PYBOX input/output guide: `references/process-pybox.md`
- Documentation sources, precedence, and known upstream inconsistencies: `references/documentation-audit.md`

## Standard Workflow

1. Classify the user's intent.
   - Use `solwcad` for H2O-CO2 saturation and volatile partitioning in silicate melts.
   - Use `conduit` for one-dimensional magma flow in volcanic conduits or fissures.
   - Use `pybox` for pyroclastic density current dispersal and invasion-area simulations.
   - If the process is unclear, ask the user to choose between the plausible processes.

2. Fetch live service metadata when possible.
   - Process list: `GET https://voice.pi.ingv.it/geoinquire/processes?f=json`
   - Process description: `GET https://voice.pi.ingv.it/geoinquire/processes/{processId}?f=json`
   - OpenAPI document: `GET https://voice.pi.ingv.it/geoinquire/openapi?f=json`
   - Conformance: `GET https://voice.pi.ingv.it/geoinquire/conformance?f=json`
   - Inspect both `example` and `examples` fields in process descriptions. CONDUIT and SOLWCAD currently use `examples`; PYBOX uses `example`.
   - Use `references/documentation-audit.md` when live sources disagree. Surface the conflict instead of silently selecting a convenient example.

3. Build a candidate execution payload.
   - Use the OGC API Processes execute envelope.
   - Send scalar and array inputs directly.
   - Wrap object-valued inputs as `{"value": {...}}`.
   - Always generate `outputs` as an object keyed by output ID. Do not generate the plugin's non-standard output-list extension.
   - If strict validation rejects a direct array or integer because it matches overlapping branches of the published OGC `oneOf`, report the known schema ambiguity. Do not rewrite service-compatible values solely to silence that validator.

```json
{
  "inputs": {
    "scalar_input": 1.0,
    "array_input": [1, 2],
    "object_input": {
      "value": {
        "property": "value"
      }
    }
  },
  "outputs": {
    "output_name": {
      "transmissionMode": "value"
    }
  }
}
```

   - If the user has not requested specific outputs, include all outputs unless that would be unusually large or binary-heavy.
   - Prefer `reference` for large binary outputs and `value` for compact text/chart outputs.
   - Request at least one output by `value`; the current shared plugin cannot return an all-reference result.
   - For PYBOX, do not request `spatial_evolution` by reference until the documented upstream output-key defect is verified or fixed.

4. Validate the candidate payload.
   - Check required inputs.
   - Check `oneOf` branches and enum values.
   - Check numeric ranges and exclusive bounds.
   - Check units and whether user-provided percentages must be converted to fractions.
   - Check array lengths and required object properties.
   - Check cross-field and runtime constraints not expressed by JSON Schema, including PYBOX `sum(eps0) < 1`.
   - Do not normalize or repair scientific compositions silently. Report suspicious totals and ask an expert/user what to do.
   - Check process execution mode: sync, async, or async-only.

5. Present preflight feedback.
   Include:
   - selected process and reason,
   - endpoint,
   - execution mode,
   - requested outputs,
   - complete candidate payload,
   - values inferred from the request,
   - values still missing,
   - validation warnings,
   - exact question or confirmation prompt.

6. If inputs are missing, ask only for the missing or ambiguous values.
   - Keep questions grouped by model section.
   - Do not ask for optional values unless they affect the user's stated goal.
   - Do not proceed to execution confirmation until required inputs are complete.

7. Ask for explicit execution confirmation.
   A suitable prompt:

```text
The request is complete and passes the documented schema checks. Do you want me to execute this Geo-INQUIRE job now?
```

8. Execute after confirmation.
   - Sync-capable processes may use `Prefer: respond-sync` if the user wants immediate results.
   - Async-capable processes may use `Prefer: respond-async`.
   - `pybox` is async-only.
   - Execution endpoint: `POST https://voice.pi.ingv.it/geoinquire/processes/{processId}/execution?f=json`

9. Track jobs.
   - Job status: `GET https://voice.pi.ingv.it/geoinquire/jobs/{jobId}?f=json`
   - Job results: `GET https://voice.pi.ingv.it/geoinquire/jobs/{jobId}/results?f=json`
   - Report job ID, status, progress, message, timestamps, and result links.

10. Summarize results.
   - Keep the raw result available.
   - Identify output names and media types.
   - For tables, report row count, column meanings if documented, and a small preview.
   - For charts, report domain, series names, units, and point counts.
   - For GeoTIFFs or binary outputs, preserve links or saved files and describe what each artifact represents.
   - Trust the process output metadata and actual response `Content-Type` over the incomplete per-operation OpenAPI 200-response schemas.

## Preflight Response Template

Use this structure before any execution:

````markdown
**Preflight**
Process: `{processId}` - {short reason}
Endpoint: `{executionEndpoint}`
Mode: `{sync-or-async}`

Inputs:
| Section | Parameter | Value | Unit | Source | Status |
| --- | --- | --- | --- | --- | --- |

Outputs:
| Output | Transmission | Notes |
| --- | --- | --- |

Validation:
- {schema/range/unit finding}

Missing Or Ambiguous:
- {only if needed}

Candidate Payload:
```json
{
  "...": "complete candidate payload"
}
```

{Question for missing information or explicit execution confirmation}
````

## Error Handling

- If the live service is unavailable, say that live schema validation could not be completed and use local references only for planning.
- If the API returns an error, show the HTTP status, response body summary, and likely correction path.
- If a job fails, retrieve job details and report the job message. Do not retry automatically unless the user asks.
- If result content is multipart or binary, avoid forcing it through JSON parsing. Preserve/download artifacts where possible.
