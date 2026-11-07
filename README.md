# entities-releases

Binary artifacts distributed with [Entities](https://github.com/polysok/entities),
a 100%-local desktop app that runs pipelines of specialized tools (OCR, PDF
reading, email processing…). This repository only hosts large files that do not
belong in the application's git history: the files are attached to releases
and downloaded by the app at first launch.

## Why a separate repository

The Python interpreter the app runs tools with is `python.wasm`, built as a
WebAssembly module by the [wasi-python](https://pypi.org/project/wasi-python/)
project. Upstream only ships it inside a 112 MB wheel, whose standard library
weighs 65 MB and includes packages (numpy, pandas, scipy…) that are compiled
into the binary but cannot be imported through the WebAssembly standard
library zip the app mounts. Entities therefore republishes the binary as-is
and a **standard-library-only** zip, so that a first launch downloads
103 MB + 5.4 MB instead of the full wheel.

## Releases and contents

| Tag | Artifact | Size | Purpose |
|---|---|---|---|
| `interpreter-0.1.4` | `python.wasm` | ~103 MB | CPython 3.15 compiled for WASI (wasi-python 0.1.4), unchanged |
| `interpreter-0.1.4` | `python315.zip` | ~5.4 MB | Python standard library only, reduced from the upstream zip |
| `interpreter-0.1.4` | `LICENSE`, `LICENSES.zip` | — | Apache-2.0 license and the licenses of the third-party code compiled into `python.wasm` (see Licensing) |

### Checksums (SHA-256)

```
503293500a642197f6335499cf8094c02e397671261ac2f422b3f6d759ffdf28  python.wasm
bfaa4e0cc21a23fe5349544ad178c912fc962cc24fd592c62a79e2c4136245af  python315.zip
```

The app verifies these checksums after downloading and refuses to install a
file that does not match.

### Reproducibility

`python.wasm` is the unmodified file from the `wasi-python` 0.1.4 wheel
(`pip install wasi-python==0.1.4`). The reduced `python315.zip` is rebuilt
with Entities' `scripts/slim-stdlib.py`: the module list is asked to
`python.wasm` itself (`sys.stdlib_module_names`) and the output is
reproducible — the same inputs give the checksum above.

## Licensing

`python.wasm` is redistributed unchanged from wasi-python
([Apache-2.0](https://github.com/bretwo/wasi-python)); its `LICENSE` file is
attached to the release unchanged. Because wasi-python compiles third-party
libraries (numpy, cairo, bzip2…) into that binary, their licenses are kept
too: `LICENSES.zip` holds the `LICENSES/` folder shipped in the upstream
wheel, one text file per library.
