# Tests overview (for contributors)

This directory holds Pydantic’s automated tests. Most work is **pytest** unit/integration tests at the top level; special suites live in subfolders.

Quick start (from the repo root):

```bash
make test          # main pytest suite
make testcov       # tests with coverage
make test-mypy     # mypy plugin suite (tests/mypy)
make               # lint + tests (see Makefile / CONTRIBUTING.md)
```

Useful filters:

```bash
uv run pytest tests/test_fields.py -q
uv run pytest tests -k "alias" --ignore=tests/test_docs.py
uv run pytest --benchmark-enable tests/benchmarks
```

Shared fixtures and helpers: [`conftest.py`](./conftest.py) (e.g. `create_module`, CLI flags `--test-mypy` / `--update-mypy`), [`utils.py`](./utils.py).

`pydantic_core` under this tree is a symlink/link into the core package tests area used by some setups; core’s own suite also lives under `pydantic-core/tests/`.

---

## Folder structure

```
tests/
├── conftest.py, utils.py     # shared pytest fixtures & helpers
├── test_*.py                 # main feature tests (one area per file, mostly)
├── benchmarks/               # performance benchmarks (pytest-benchmark)
├── mypy/                     # mypy *plugin* behavior (golden outputs)
├── typechecking/             # static typing of public APIs (mypy/pyright/pyrefly)
├── types/                    # extra focused type-behavior tests
├── plugin/                   # sample plugin + plugin integration test
├── test_pydantic_settings.sh # third-party: pydantic-settings
└── test_pydantic_extra_types.sh  # third-party: pydantic-extra-types
```

| Area | What it is | When to touch it |
|------|------------|------------------|
| Top-level `test_*.py` | Runtime behavior of models, validators, serializers, types, config, etc. | Most PRs that change library behavior |
| `benchmarks/` | Timing / throughput of validation, schema gen, imports, FastAPI-style startup | Perf-sensitive changes |
| `mypy/` | Runs mypy on small modules with various configs; compares to checked-in outputs | Mypy plugin or typing diagnostics |
| `typechecking/` | Files type-checked in CI (not executed as unit tests) | Public type hints / overloads |
| `types/` | Narrower type cases (unions, paths, fractions, nested models) | Specific type edge cases |
| `plugin/` | Example entry-point plugin + loader/integration | Plugin system |
| `*.sh` scripts | Install/run tests against related packages | Cross-package regressions |

Details for the typing suites: [`mypy/README.md`](./mypy/README.md), [`typechecking/README.md`](./typechecking/README.md).

---

## Top-level test files (what each covers)

Grouped by theme. Filenames map 1:1 to the feature area unless noted.

### Core models & construction

| File | Covers |
|------|--------|
| `test_main.py` | Core `BaseModel` behavior (validation, methods, inheritance, common APIs) |
| `test_construction.py` | `model_construct`, building instances without full validation |
| `test_create_model.py` | Dynamic `create_model(...)` |
| `test_fields.py` | `Field()`, defaults, constraints, metadata on fields |
| `test_private_attributes.py` | `PrivateAttr` / private fields |
| `test_computed_fields.py` | `@computed_field` (properties on models) |
| `test_root_model.py` | `RootModel` |
| `test_model_signature.py` | Generated `__init__` / signatures |
| `test_model_validator.py` | `@model_validator` |
| `test_abc.py` | Models mixed with `abc` abstract base classes |
| `test_meta.py` | Metaclass / model class machinery edges |
| `test_pickle.py` | Pickling models |
| `test_rich_repr.py` | Rich `__repr__` / pretty printing integration |
| `test_structural_pattern_matching.py` | `match` / class patterns on models |

### Config, aliases, strictness, partial data

| File | Covers |
|------|--------|
| `test_config.py` | `ConfigDict` / model config options |
| `test_aliases.py` | Field aliases, `AliasPath` / `AliasChoices`, validation & serialization aliases |
| `test_strict.py` | Strict mode validation |
| `test_allow_partial.py` | Partial validation (e.g. incomplete dicts / typed dicts) |
| `test_titles.py` | Schema / model titles |
| `test_missing_sentinel.py` | Experimental / missing-value sentinel behavior |

### Validators, serializers, decorators

| File | Covers |
|------|--------|
| `test_validators.py` | Field/model validators on models |
| `test_validators_dataclass.py` | Validators on Pydantic dataclasses |
| `test_assert_in_validators.py` | Assertion failures surfaced as validation errors |
| `test_serialize.py` | Serialization (`model_dump`, modes, custom serializers) |
| `test_serialize_as_any.py` | `SerializeAsAny` / “serialize as any” behavior |
| `test_decorators.py` | Internal decorator inspection (validator/serializer signatures) |
| `test_validate_call.py` | `@validate_call` on functions/methods |
| `test_callable.py` | `Callable` type annotations on fields |

### Types & annotations

| File | Covers |
|------|--------|
| `test_types.py` | Broad built-in / constrained / special Pydantic types |
| `test_types_typeddict.py` | `TypedDict` support |
| `test_types_namedtuple.py` | `NamedTuple` support |
| `test_types_self.py` | `typing.Self` |
| `test_types_zoneinfo.py` | `zoneinfo` / timezone types |
| `test_types_payment_card_number.py` | Payment card number type |
| `test_annotated.py` | `Annotated[...]` metadata (constraints, custom metadata) |
| `test_type_alias_type.py` | `TypeAliasType` / PEP 695-style aliases |
| `test_type_hints.py` | Type-hint resolution / utilities used by Pydantic |
| `test_typing.py` | Typing helpers and edge cases |
| `test_type_adapter.py` | `TypeAdapter` (validate/serialize arbitrary types) |
| `test_forward_ref.py` | Forward references / string annotations |
| `test_deferred_annotations.py` | PEP 649/749 deferred annotations (Python 3.14+) |
| `test_generics.py` | Generic models / type variables |
| `test_discriminated_union.py` | Discriminated (tagged) unions |
| `test_color.py` | `Color` type |
| `test_datetime.py` | Date/time parsing and validation |
| `test_networks.py` | Network types (URLs, emails, etc.) |
| `test_networks_ipaddress.py` | IP address types |
| `types/test_*.py` | Extra focused cases: unions, paths, fractions, model-as-type |

### Dataclasses, JSON, schema, tools

| File | Covers |
|------|--------|
| `test_dataclasses.py` | Pydantic dataclasses vs stdlib dataclasses |
| `test_json.py` | JSON encode/decode helpers and model JSON APIs |
| `test_json_schema.py` | JSON Schema generation |
| `test_docs_extraction.py` | Pulling field descriptions / docs into schema |
| `test_parse.py` | Deprecated / legacy parse helpers (where still tested) |
| `test_tools.py` | Utility “tools” APIs (e.g. schema helpers historically under `tools`) |
| `test_pipeline.py` | Experimental validation pipeline / transforms |
| `test_experimental_arguments_schema.py` | Experimental arguments / call schema APIs |

### Errors, warnings, deprecations, v1 compatibility

| File | Covers |
|------|--------|
| `test_errors.py` | `ValidationError`, user errors, error URLs/codes |
| `test_warnings.py` | Warning types and emission |
| `test_deprecated.py` | Deprecated APIs still supported with warnings |
| `test_deprecated_fields.py` | Deprecated field markers / `deprecated` typing |
| `test_deprecated_validate_arguments.py` | Legacy `@validate_arguments` |
| `test_v1.py` | `pydantic.v1` compatibility layer |
| `test_migration.py` | v1→v2 migration helpers (`_migration`) |
| `test_edge_cases.py` | Odd combinations and regression traps |

### Plugins, exports, docs-as-tests, version

| File | Covers |
|------|--------|
| `test_plugins.py` | Plugin hooks during validation/schema build |
| `test_plugin_loader.py` | Discovering/loading plugins |
| `plugin/test_plugin.py` | End-to-end with `plugin/example_plugin.py` |
| `test_exports.py` | Public exports / `__all__` consistency |
| `test_dunder_all.py` | Package `__all__` re-exports exist and are explicit |
| `test_docs.py` | Executes/checks documentation examples (can update with `--update-examples`) |
| `test_version.py` | Version helpers / version_info |
| `test_internal.py` | Selected internal implementation details (use sparingly when changing `_internal`) |

### Shell scripts (related packages)

| File | Covers |
|------|--------|
| `test_pydantic_settings.sh` | Run **pydantic-settings** tests against this Pydantic |
| `test_pydantic_extra_types.sh` | Run **pydantic-extra-types** tests against this Pydantic |

Invoked via Makefile targets such as `test-pydantic-settings` / `test-pydantic-extra-types` (see root `Makefile`).

---

## Subfolders in more detail

### `benchmarks/`

Pytest benchmarks (often gated with `--benchmark-enable`). Examples of intent:

- Model validation / serialization throughput
- Schema generation (including recursive models)
- Discriminated unions, regex, `isinstance` checks
- Import time and FastAPI-like app startup (simple vs generics)
- “North star” scenario data (`generate_north_star_data.py`, `test_north_star.py`)
- Shared fixtures in `shared.py`

Run: `uv run pytest --benchmark-enable tests/benchmarks` (or `make` target that enables benchmarks).

### `mypy/`

Golden-file suite for the **Pydantic mypy plugin**:

- `modules/` — input Python snippets
- `configs/` — mypy / pyproject configs (plugin on/off, strictness variants)
- `outputs/` — expected mypy diagnostics merged into source
- `test_mypy.py` — driver (`--test-mypy`, `--update-mypy` / `make test-mypy-update`)

See [`mypy/README.md`](./mypy/README.md) for adding cases.

### `typechecking/`

Not a normal pytest collection of runtime asserts. Files are **type-checked** in CI with mypy, Pyright, and Pyrefly using `typechecking/pyproject.toml`. Use `assert_type` and `# type: ignore` / `# pyright: ignore` as documented in [`typechecking/README.md`](./typechecking/README.md). Separate from the mypy *plugin* suite.

### `types/`

Smaller pytest modules for specific type behaviors (`test_union.py`, `test_path.py`, `test_fraction.py`, `test_model.py`) kept out of the very large `test_types.py` where useful.

### `plugin/`

Minimal installable example plugin (`example_plugin.py` + `pyproject.toml`) exercised by `test_plugin.py`.

---

## Tips for new contributors

1. **Find the right file by feature name** — e.g. serializers → `test_serialize.py`, aliases → `test_aliases.py`. Prefer extending an existing file over adding a new top-level module unless the area is large or orthogonal.
2. **Prefer focused tests** that fail clearly when the bug returns; use parametrize for input matrices.
3. **Docs examples** are tested in `test_docs.py`; if you change docs code blocks, run that module (and `--update-examples` when intentionally refreshing outputs — see `CONTRIBUTING.md`).
4. **Typing changes** may need both runtime tests *and* updates under `typechecking/` and/or `mypy/`.
5. **Core vs Pydantic** — low-level validation often belongs in `pydantic-core`; this `tests/` tree is mainly the Python package surface.
6. Full contributor workflow (env, lint, docs): root [`CONTRIBUTING.md`](../CONTRIBUTING.md) and [`Makefile`](../Makefile).

This document is intentionally concise; treat it as a map, not a full API reference. When in doubt, open the matching `test_*.py` and search for a similar case.
