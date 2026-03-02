# Vera

MPCORB small body orbital analysis tool written in [Vera](https://veralang.dev), a contract-driven programming language with refinement types and algebraic effects.

## Overview

Vera parses the Minor Planet Center's [MPCORB.DAT](https://minorplanetcenter.net/iau/MPCORB.html) fixed-width format (~1.3M records of minor planet orbital elements) and produces validated, typed records. Refinement types enforce physical constraints at the type level — angles stay within bounds, eccentricities remain sub-unity, and distances stay positive.

## Structure

```
vera/
├── mpcorb.vera            Main module: MinorPlanet ADT, ParseResult, parse_line
└── mpcorb/
    ├── packed.vera         Packed designation & epoch decoders
    └── fields.vera         Individual field parsers with refinement types
```

### `mpcorb.vera`

Top-level entry point. Defines the `MinorPlanet` record carrying all 17 orbital element fields, a `ParseResult` ADT for error handling, and `parse_line` which orchestrates the field parsers on a single 202-character MPCORB line.

### `mpcorb/packed.vera`

Decoders for the MPC's compact encodings:

- **`unpack_designation`** — 7-char packed designation to readable string. Handles numbered asteroids (plain, letter-prefixed 100k–619k, tilde-encoded ≥620k via base-62) and provisional designations.
- **`unpack_epoch`** — 5-char packed date to (year, month, day) tuple. Decodes century letters (I/J/K), month chars (1–9, A–C), and day chars (1–9, A–V).
- **`letter_to_number`** — A=10..Z=35, a=36..z=61 character encoding.

### `mpcorb/fields.vera`

Fifteen pure functions, one per MPCORB column group. Each slices the input line at the correct offset and returns a refined type:

| Field | Columns | Type |
|---|---|---|
| Designation | 1–7 | `String` |
| Absolute magnitude H | 9–13 | `Option<Float64>` |
| Slope parameter G | 15–19 | `Option<Float64>` |
| Epoch | 21–25 | `Tuple<Nat, Nat, Nat>` |
| Mean anomaly M | 27–35 | `Degrees` (0 ≤ x < 360) |
| Arg. perihelion ω | 38–46 | `Degrees` |
| Long. asc. node Ω | 49–57 | `Degrees` |
| Inclination i | 60–68 | `Inclination` (0 ≤ x ≤ 180) |
| Eccentricity e | 71–79 | `Eccentricity` (0 ≤ x < 1) |
| Mean daily motion n | 81–91 | `PositiveFloat` |
| Semimajor axis a | 93–103 | `PositiveFloat` |
| Num. observations | 118–122 | `Nat` |
| Num. oppositions | 124–126 | `Nat` |
| RMS residual | 138–141 | `Option<Float64>` |
| Readable name | 167–194 | `String` |

## Refinement Types

```vera
type Degrees      = { @Float64 | @Float64.0 >= 0.0 && @Float64.0 < 360.0 };
type Inclination   = { @Float64 | @Float64.0 >= 0.0 && @Float64.0 <= 180.0 };
type Eccentricity  = { @Float64 | @Float64.0 >= 0.0 && @Float64.0 < 1.0 };
type PositiveFloat = { @Float64 | @Float64.0 > 0.0 };
type Magnitude     = { @Float64 | @Float64.0 >= -2.0 && @Float64.0 <= 35.0 };
```

These constraints are checked statically by Vera's verifier (via Z3), ensuring that physically impossible values cannot be constructed.

## Usage

```bash
vera check vera/mpcorb.vera       # parse and type-check
vera verify vera/mpcorb.vera      # verify contracts via Z3
vera test vera/mpcorb.vera        # contract-driven testing
```

## References

- [MPCORB.DAT format specification](https://minorplanetcenter.net/iau/info/MPOrbitFormat.html)
- [MPC packed designation format](https://minorplanetcenter.net/iau/info/PackedDes.html)
- [Vera language](https://veralang.dev)

## License

MIT
