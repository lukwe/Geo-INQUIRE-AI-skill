# CONDUIT Process Reference

Process ID: `conduit`

Title: CONDUIT

Version: `2.2.0`

Execution: sync and async

Purpose: compute one-dimensional, steady, isothermal, multiphase and multicomponent flow of magma in volcanic conduits or fissures.

Live description:

`https://voice.pi.ingv.it/geoinquire/processes/conduit?f=json`

Implementation source:

`https://github.com/francescoingv/ingv-pygeoapi-process-plugins/blob/main/ingv_plugin_pygeoapi/process/conduit.py`

## Inputs

CONDUIT has seven required top-level input sections:

- `melt_composition`
- `volatiles`
- `crystals`
- `fragmentation`
- `pressure_temperature`
- `geometry`
- `searching_mode`

### `melt_composition`

All values are weight fractions, not wt%.

Required fields:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `sio2` | SiO2 weight fraction | `0 < value < 1` |
| `tio2` | TiO2 weight fraction | `0 < value < 1` |
| `al2o3` | Al2O3 weight fraction | `0 < value < 1` |
| `fe2o3` | Fe2O3 weight fraction | `0 < value < 1` |
| `feo` | FeO weight fraction | `0 < value < 1` |
| `mno` | MnO weight fraction | `0 < value < 1` |
| `mgo` | MgO weight fraction | `0 < value < 1` |
| `cao` | CaO weight fraction | `0 < value < 1` |
| `na2o` | Na2O weight fraction | `0 < value < 1` |
| `k2o` | K2O weight fraction | `0 < value < 1` |

If the user gives percentages, ask whether to convert to fractions. Do not silently convert.

The documented example oxide fractions sum to `0.9985`, plausibly because of rounding. Do not silently normalize a composition to `1.0`; report the total and ask the user or expert whether normalization is appropriate.

### `volatiles`

Required fields:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `h2o` | Total H2O weight fraction | `0 < value < 1` |
| `co2` | Total CO2 weight fraction | `0 < value < 1` |

### `crystals`

Required fields:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `c` | Crystal volume fraction relative to degassed magma | `0 < value <= 0.7` |
| `den` | Average crystal density, kg/m3 | `value > 0` |

### `fragmentation`

Required fields:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `fe` | Fragmentation efficiency | `0.2 <= value <= 1.0` |
| `pd` | Pumice degassing parameter | `0.0 <= value <= 1.0` |
| `dp` | Average pumice diameter at fragmentation, m | `value > 0` |
| `ds` | Average shard diameter at fragmentation, m | `value > 0` |
| `dc` | Average crystal diameter at fragmentation, m | `value > 0` |

### `pressure_temperature`

Required fields:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `p` | Magma chamber pressure, Pa | `value > 101325` |
| `t` | Magma temperature, K | `value > 273.15` |

### `geometry`

Required fields:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `g` | Geometry | enum: `conduit`, `fissure` |
| `l` | Conduit length, m | `value > 0` |

### `searching_mode`

The schema uses `oneOf`; select one branch.

#### Search for mass flow rate from diameter

Required:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `d` | Conduit diameter or fissure width, m | `value > 0` |

Optional:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `fg` | Initial guess for mass flow rate | `value > 0` |

#### Search for diameter from mass flow rate

Required:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `f` | Mass flow rate or mass flow per unit surface | `value > 0` |

Optional:

| Field | Meaning | Constraint |
| --- | --- | --- |
| `dg` | Initial guess for diameter or fissure width, m | `value > 0` |

## Outputs

| Output | Media type | Meaning |
| --- | --- | --- |
| `gas` | chart JSON | Gas volume fraction profile along conduit |
| `velocity` | chart JSON | Liquid and gas velocity profiles |
| `pressure` | chart JSON | Pressure profile |
| `outfile` | text/csv | Table of dependent variables and physical properties |
| `exit` | text/plain | Exit density, velocity, pressure, and flow rates |

Use the OGC output-object form. The service accepts output-name arrays as a local extension, but the skill must not generate them. At least one output must use `transmissionMode: "value"`; all-reference responses are not implemented by the shared plugin.

## Verified Request Example

This payload passes the live process schemas and the published OGC execute envelope. Values come from the service example and are examples, not defaults.

```json
{
  "inputs": {
    "melt_composition": {
      "value": {
        "sio2": 0.7669,
        "tio2": 0.0012,
        "al2o3": 0.1322,
        "fe2o3": 0.0039,
        "feo": 0.0038,
        "mno": 0.0007,
        "mgo": 0.0006,
        "cao": 0.008,
        "na2o": 0.03,
        "k2o": 0.0512
      }
    },
    "volatiles": {
      "value": {
        "h2o": 0.05,
        "co2": 0.02
      }
    },
    "crystals": {
      "value": {
        "c": 0.1,
        "den": 2800
      }
    },
    "fragmentation": {
      "value": {
        "fe": 0.2,
        "pd": 0.9,
        "dp": 0.0002,
        "ds": 0.0002,
        "dc": 0.0002
      }
    },
    "pressure_temperature": {
      "value": {
        "p": 100000000,
        "t": 1050
      }
    },
    "geometry": {
      "value": {
        "g": "conduit",
        "l": 4000
      }
    },
    "searching_mode": {
      "value": {
        "f": 88393000
      }
    }
  },
  "outputs": {
    "gas": {
      "transmissionMode": "value"
    },
    "velocity": {
      "transmissionMode": "value"
    }
  }
}
```

All seven top-level inputs are objects and therefore use the qualified `value` wrapper.

### `outfile` Columns

1. conduit length, m
2. gas in pumice, volume %
3. gas before fragmentation, volume %
4. gas velocity, m/s
5. liquid velocity, m/s
6. pressure, MPa
7. dissolved H2O, mass %
8. dissolved CO2, mass %
9. gas H2O, mass %
10. gas CO2, mass %
11. gas before fragmentation, mass %
12. crystals, volume %
13. crystals, mass %
14. liquid+crystals+gas viscosity, Pa s
15. liquid+crystals viscosity, Pa s
16. liquid viscosity, Pa s
17. liquid+crystals+gas density, kg/m3
18. liquid+crystals density, kg/m3
19. liquid density, kg/m3
20. gas density, kg/m3
21. rate of strain, s^-1

## Preflight Questions To Ask

Ask only for values not already supplied:

- Is the geometry a cylindrical conduit or planar fissure?
- Is the user providing conduit diameter/fissure width, or mass flow rate?
- Are melt oxide values fractions or wt%?
- What are total H2O and CO2 fractions?
- What pressure and temperature should be used?
- What crystal fraction and crystal density should be used?
- What fragmentation parameters should be used?
- Which outputs does the user want: charts, CSV table, exit conditions, or all?
- Does the supplied volatile-free oxide composition need expert-approved normalization, or should it be preserved exactly as entered?
