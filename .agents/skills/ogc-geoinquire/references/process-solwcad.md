# SOLWCAD Process Reference

Process ID: `solwcad`

Title: SOLWCAD

Version: `1.0.1`

Execution: sync and async

Purpose: compute the saturation surface and volatile partitioning of H2O-CO2 fluids in silicate melts of arbitrary composition.

Live description:

`https://voice.pi.ingv.it/geoinquire/processes/solwcad?f=json`

Implementation source:

`https://github.com/francescoingv/ingv-pygeoapi-process-plugins/blob/main/ingv_plugin_pygeoapi/process/solwcad.py`

## Inputs

SOLWCAD has two required top-level inputs:

- `swinput.data`
- `sw.data`

### `swinput.data`

Object describing the desired computation. The schema uses `oneOf`; select exactly one branch according to `kl`.

#### Branch `kl = 0`

Computation at user-defined pressure-temperature conditions in `sw.data`.

Required:

| Parameter | Type | Notes |
| --- | --- | --- |
| `ndat1` | integer | First item in `sw.data` to compute |
| `ndat2` | integer | Last item in `sw.data` to compute |
| `kl` | integer enum | Must be `0` |

#### Branch `kl = 1`

Computation based on item `ndat1`, at constant user-defined temperature and pressure from user-defined pressure to atmospheric pressure.

Required:

| Parameter | Type | Notes |
| --- | --- | --- |
| `ndat1` | integer | Item in `sw.data` |
| `kl` | integer enum | Must be `1` |
| `iopen` | integer enum | `0` closed system, `1` open system |

Optional:

| Parameter | Type | Notes |
| --- | --- | --- |
| `fopen` | numeric string | Weight fraction of fluid phase lost at each step; used only when `iopen = 1` |

#### Branch `kl = 2`

Like `kl = 1`, but fixed pressure from `sw.data` and temperature steps toward `tlimit`.

Required:

| Parameter | Type | Notes |
| --- | --- | --- |
| `ndat1` | integer | Item in `sw.data` |
| `kl` | integer enum | Must be `2` |
| `iopen` | integer enum | `0` closed system, `1` open system |
| `dt` | numeric string | Temperature step, positive or negative |
| `tlimit` | numeric string | Target temperature |

Optional:

| Parameter | Type | Notes |
| --- | --- | --- |
| `fopen` | numeric string | Weight fraction of fluid phase lost at each step; used only when `iopen = 1` |

#### Branch `kl = -1`

H2O and CO2 in `sw.data` represent amounts dissolved in the melt phase. Commonly used for melt inclusion analysis.

Required:

| Parameter | Type | Notes |
| --- | --- | --- |
| `ndat1` | integer | First item in `sw.data` to compute |
| `ndat2` | integer | Last item in `sw.data` to compute |
| `kl` | integer enum | Must be `-1` |

### `sw.data`

Array of one or more item arrays. Each item must contain exactly 14 numeric strings:

1. pressure, Pa
2. temperature, K
3. H2O content, weight fraction
4. CO2 content, weight fraction
5. SiO2, volatile-free melt composition weight fraction
6. TiO2, volatile-free melt composition weight fraction
7. Al2O3, volatile-free melt composition weight fraction
8. Fe2O3, volatile-free melt composition weight fraction
9. FeO, volatile-free melt composition weight fraction
10. MnO, volatile-free melt composition weight fraction
11. MgO, volatile-free melt composition weight fraction
12. CaO, volatile-free melt composition weight fraction
13. Na2O, volatile-free melt composition weight fraction
14. K2O, volatile-free melt composition weight fraction

Important ambiguity:

- For `kl = 0`, `1`, or `2`, H2O and CO2 may refer to total amounts in the two-phase magma.
- For `kl = -1`, H2O and CO2 refer to dissolved amounts in the melt phase.
- If the user gives wt% instead of fractions, ask whether to convert. Do not silently convert.
- Preserve each value as a decimal numeric string accepted by the schema, including Fortran `D` notation where supplied. Do not convert the strings to JSON numbers in the execution payload.
- Confirm `ndat1` and `ndat2` refer to valid one-based item positions in the submitted `sw.data` array.
- Check and report the sum of the ten volatile-free oxide fractions for every row. Do not silently normalize them.
- The second row of the published example sums to `0.9599`; treat that as a scientific review flag, not as permission to repair it automatically.

## Verified Request Example

This payload passes the live process schemas and reproduces the service example, including its second-row composition warning. A strict validator against the published OGC execute schemas reports the known overlapping-`oneOf` ambiguity for the direct `sw.data` array; see `documentation-audit.md`.

```json
{
  "inputs": {
    "swinput.data": {
      "value": {
        "ndat1": 1,
        "ndat2": 2,
        "kl": 0
      }
    },
    "sw.data": [
      [
        "1.00d8", "1273.", ".0400", ".0200", ".7653", ".0032",
        ".1201", ".0027", ".0246", ".0006", ".0018", ".0132",
        ".0378", ".0306"
      ],
      [
        "2.00d8", "1173.", ".0200", ".0010", ".7053", ".0032",
        ".1301", ".0027", ".0146", ".0006", ".0118", ".0232",
        ".0378", ".0306"
      ]
    ]
  },
  "outputs": {
    "solwcad_out": {
      "transmissionMode": "value"
    }
  }
}
```

`swinput.data` is object-valued and uses the qualified `value` wrapper. `sw.data` is array-valued and is sent directly.

## Outputs

Output name: `solwcad_out`

Supported media types:

- `application/json`
- `text/plain`

When no user preference is stated, prefer `application/json` because it is directly parseable. Use `text/plain` when the user needs the plain-text representation or when matching a published example.
To request a specific representation, set `outputs.solwcad_out.format.mediaType` to one of the supported media types.

The output is an array of rows. Each row has 15 numeric string values:

1. pressure, MPa
2. temperature, K
3. total or dissolved H2O, wt%
4. total or dissolved CO2, wt%
5. H2O dissolved in melt, wt%
6. CO2 dissolved in melt, ppm
7. CO2 in fluid, wt%
8. CO2 in fluid, mol%
9. amount of fluid phase in magma, wt%
10. amount of fluid phase in magma, vol%
11. melt density, kg/m3
12. gas density, kg/m3
13. two-phase magma density, kg/m3
14. melt viscosity, log(Pa s)
15. two-phase magma viscosity, log(Pa s)

## Preflight Questions To Ask

Ask only for values not already supplied:

- Which computation mode should be used: `kl = 0`, `1`, `2`, or `-1`?
- Which `sw.data` item range should be computed?
- Are H2O and CO2 values fractions or wt%?
- What are pressure, temperature, volatile contents, and oxide composition values?
- For open-system modes, is `iopen` `0` or `1`?
- If `iopen = 1`, what is `fopen`?
- For `kl = 2`, what are `dt` and `tlimit`?
- Do `ndat1` and `ndat2` match the submitted row range?
- Should any oxide composition that does not sum to the expected volatile-free total be reviewed or normalized by a scientist?
