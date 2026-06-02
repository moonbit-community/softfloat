# SoftFloat Vector Fixtures

This directory contains the committed golden-vector fixtures used by the
MoonBit tests. The repository no longer regenerates these fixtures locally;
`tools/vectors/` is the source of truth.

## Commands

Rebuild the generated MoonBit batch test packages from the existing fixture
files:

```bash
tools/test_gen.mjs packages
moon fmt
```

Validate the committed fixture file structure:

```bash
tools/check_vectors.sh
```

`check_smoke.sh` is kept as a compatibility alias:

```bash
tools/check_smoke.sh
```

## Fixtures

Fixtures live under `tools/vectors/`. Each non-comment record is line-oriented:

```text
specialization operation rounding tininess exact inputs... output flags
```

Fields:

- `specialization`: `x86_8086_sse` or `riscv`.
- `operation`: MoonBit snake-case operation name, such as `f64_add` or
  `f32_to_ui32_min_mag`.
- `rounding`: `near_even`, `min_mag`, `min`, `max`, `near_max_mag`, or `odd`.
- `tininess`: `before` or `after`.
- `exact`: `0` or `1`; present for every record.
- `inputs`: fixed-width bit patterns or two's-complement integer inputs.
- `output`: fixed-width bit pattern, integer value, or `0`/`1` boolean.
- `flags`: two hex digits for exception bits.

`vectors/smoke.txt` contains a small sanity set for both specializations.
`vectors/core.txt` contains one representative case for every v1
function-style operation and both specializations. `golden_core_wbtest.mbt`
checks the MoonBit implementation against those committed records.

The larger fixture set includes:

- `edge_*`: boundary values for every included format and mode combination.
- `random_*`: fixed-seed random arithmetic records.
- `exhaustive_bf16_*`: all 65,536 BF16 inputs for selected unary operations.
- `exhaustive_f16_*`: all 65,536 F16 inputs for selected unary operations and
  round-to-int modes.

`golden_sweep_digest_wbtest.mbt` recreates the deterministic input rows in
MoonBit, folds each result and exception flag into a per-file digest, and
checks those digests with `@debug.assert_eq` against the committed records.

`tools/test_gen.mjs packages` converts the committed fixtures into MoonBit test
packages under `sweep_test/batch<N>/`. Core and smoke files are emitted as
direct record assertions; sweep files are emitted as 200-record digest range
tests. Generated batch packages use at most 10 MoonBit test files per package
and at most 100 `test` blocks per file.
