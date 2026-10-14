# PYBOX Process Reference

Process ID: `pybox`

Title: PYBOX

Version: `1.0.0`

Execution: async only

Purpose: simulate dispersal of a gravity-driven pyroclastic density current using a box-model physical description and a Digital Surface Model as topography.

Live description:

`https://voice.pi.ingv.it/geoinquire/processes/pybox?f=json`

Implementation sources and live metadata:

- Plugin: `https://github.com/francescoingv/ingv-pygeoapi-process-plugins/blob/main/ingv_plugin_pygeoapi/process/pybox.py`
- Scientific code: `https://github.com/silviagians/PyBOX-Web`
- Reference manual: `https://doi.org/10.5281/zenodo.2616551`
- PyBOX-Web release DOI: `https://doi.org/10.5281/zenodo.18920969`
- Live author metadata: Silvia Giansante, `https://orcid.org/0009-0008-2803-0873`

## Inputs

PYBOX has eight required top-level inputs:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `lat` | Vent latitude, decimal degrees | `-90 <= value <= 90` |
| `lon` | Vent longitude, decimal degrees | `-180 <= value <= 180` |
| `l0` | Initial horizontal extent/radius, m | `100 <= value <= 2000` |
| `h0` | Initial vertical thickness, m | `100 <= value <= 2000` |
| `theta0` | Initial current temperature, K | `300 <= value <= 1400` |
| `multiple_values` | Particle classes | array, 1 to 21 items |
| `dt` | Time step, s | `0.1 <= value <= 30` |
| `margin` | DSM bounding-box margin from vent, m | `5000 <= value <= 50000` |

### `multiple_values`

Array of particle-class objects.

Each item requires:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `eps0` | Particle volume fraction | `0.001 <= value <= 0.1` |
| `rhos` | Particle density, kg/m3 | `500 <= value <= 3500` |
| `ds` | Particle diameter, m | `0.00001 <= value <= 0.005` |

The documented description says the sum of `eps0` values must be less than `1`. Check this in preflight even if the JSON Schema does not express the sum constraint directly.

The scientific code also rejects an initial current density below ambient air density. This condition depends on `theta0`, all particle fractions, and all particle densities. If it cannot be evaluated confidently, flag it for scientific review rather than claiming the request is physically valid.

## Outputs

| Output | Media type | Meaning |
| --- | --- | --- |
| `input_data` | text/plain | Log of input parameters used |
| `dem` | GeoTIFF and SLD links | Local DSM used for simulation, plus its visualization style |
| `invasion_map` | GeoTIFF and SLD links | 2D invaded area based on energy conoid method, plus its visualization style |
| `spatial_evolution` | chart JSON | Spatial evolution of current mean properties |
| `deposit_thickness` | chart JSON | Deposit thickness with distance from vent |

The `dem` and `invasion_map` outputs are objects containing `geotiff` and `sld` links. Preserve both the raster artifact and its Styled Layer Descriptor; do not summarize them away.

Prefer `reference` for `dem` and `invasion_map`, and request at least one compact output such as `input_data`, `spatial_evolution`, or `deposit_thickness` by `value`. The shared plugin does not implement all-reference result responses.

If the user asks for only reference outputs, do not present that all-reference output object as a viable option. Explain that the current shared plugin cannot format all-reference responses, then propose a compatible output object that preserves requested binary outputs by `reference` and adds at least one compact output, preferably `input_data`, by `value`.

Do not request `spatial_evolution` by reference until the upstream plugin's output-key assignment is verified or corrected. Value transmission is the safe documented choice.

## Verified Request Example

This payload passes the live process schemas. The coordinates match the current live payload and curl examples; they are examples, not defaults. A strict validator against the published OGC execute schemas reports known overlapping-`oneOf` ambiguities for direct integer and array values; see `documentation-audit.md`.

```json
{
  "inputs": {
    "lon": 14.428,
    "lat": 40.82,
    "l0": 150,
    "h0": 150,
    "theta0": 500,
    "multiple_values": [
      {
        "eps0": 0.01,
        "rhos": 1000,
        "ds": 0.0001
      }
    ],
    "dt": 0.5,
    "margin": 5000
  },
  "outputs": {
    "dem": {
      "transmissionMode": "reference"
    },
    "spatial_evolution": {
      "transmissionMode": "value"
    }
  }
}
```

PYBOX top-level inputs are scalars or arrays and are sent directly. Although the generic OGC envelope permits a qualified scalar `value`, the process example labels that form unsupported for this plugin; prefer direct scalar values.

## Execution Notes

- PYBOX is async-only.
- Always use `Prefer: respond-async`.
- Report the returned job ID.
- Poll `/jobs/{jobId}?f=json`.
- Fetch `/jobs/{jobId}/results` after status is `successful`.
- If results are multipart, preserve each part separately.
- Use job URLs under `https://voice.pi.ingv.it/geoinquire/jobs/{jobId}`. Published examples that omit `/geoinquire` are incorrect.

## Preflight Questions To Ask

Ask only for values not already supplied:

- What vent latitude and longitude should be used?
- What initial radius `l0` and initial height `h0` should be used?
- What initial temperature `theta0` should be used?
- How many particle classes are needed?
- For each particle class, what are `eps0`, `rhos`, and `ds`?
- What time step `dt` should be used?
- What DSM margin should be used?
- Which outputs should be returned or preserved?
- Does the combination of temperature, particle fractions, and particle densities produce an initial current denser than ambient air, or is expert review needed?

## Extra Caution

Because this process can return geospatial raster artifacts and may require larger remote work, always make the confirmation prompt explicit:

```text
PYBOX is async-only and may produce GeoTIFF artifacts. Do you want me to submit this job now?
```
