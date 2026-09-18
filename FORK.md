# Fork notes

This is the [`dnouri/duckdb-dbn`](https://github.com/dnouri/duckdb-dbn) fork of
[`tbeason/duckdb-dbn`](https://github.com/tbeason/duckdb-dbn), based on upstream
`55280ca`. The maintained branch is `improvements`.

## Changes

- Schema-specific readers reject files whose declared DBN schema belongs to a
  different reader family instead of silently returning zero rows.
- Market-data, polymorphic, symbol-mapping, and system readers accept an ordered
  `VARCHAR[]` of paths or globs as well as one `VARCHAR` path.
- The README documents the in-file mapping requirement for `symbols := true`.

The behavior commits are deliberately separate so they can be reviewed or
cherry-picked independently.

## Build and validation

The fork pins DuckDB `v1.5.5` (`d8cdaa33fd`) and the matching
`extension-ci-tools` branch (`72e76e99cd`). A direct Linux build avoids vcpkg;
the extension has no vcpkg dependencies:

```sh
cmake -G Ninja -S duckdb -B build/release \
  -DCMAKE_BUILD_TYPE=Release \
  -DEXTENSION_STATIC_BUILD=1 \
  -DDUCKDB_EXTENSION_CONFIGS="$PWD/extension_config.cmake" \
  -DBUILD_UNITTESTS=ON \
  -DENABLE_UNITTEST_CPP_TESTS=FALSE \
  -DUNITTEST_ROOT_DIRECTORY="$PWD"
ninja -C build/release -j2 dbn_loadable_extension shell unittest
./build/release/test/unittest --test-dir . 'test/*'
```

Validation on 2026-09-18 passed all 35 SQLLogicTest files (672 assertions).
The loadable extension also loaded into the stock `duckdb==1.5.5` Python wheel.
On a 146,585,133-byte owned MBO day, both `databento-python==0.86.0` and
`read_dbn_mbo` counted exactly 9,418,365 records; the DuckDB process peaked at
61,728 KiB RSS. A one-minute filtered window returned 1,328 rows with timestamps
inside the requested interval.

## Licensing

The fork retains upstream's MIT `LICENSE`. The wire structs vendored from
`databento-cpp` remain under Apache-2.0 with their source notices and full
license at `src/include/databento/LICENSE`.
