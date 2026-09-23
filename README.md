# Recoil — Static Recompilation

A static recompilation of **Recoil** (1999, developed by Zipper Interactive, published by Virgin Interactive),
targeting modern Windows with native x86 execution.

Recoil is a vehicular-combat game on the Zipper Interactive **GameZ/GOS engine** —
the same engine as **MechWarrior 3** (1999) and **Crimson Skies** (2000). This is the
third project in that engine family; tooling and engine structures carry across all
three.

## Why this game

- **Shared GOS engine.** Function-discovery approach, runtime bridges, and engine
  internals are common with MechWarrior 3 and Crimson Skies.
- **Clean binary.** `Recoil.exe` is **not** SafeDisc-protected — analyzes directly
  (no runtime memory dump needed).
- **VC6 / MFC.** Visual C++ 6.0 + MFC42, so IDA FLIRT names a chunk of the library
  surface automatically.

## Project status

| Phase | Status | Description |
|-------|--------|-------------|
| **Phase 0** | **Complete** | Disc extraction (BIN/CUE→ISO, InstallShield cab), triage, **IDA-seeded discovery** |
| **Phase 1** | **Complete** | x86→C code generation — **3,490 functions, 0 lift errors**, 321K lines |
| **Phase 2** | **Complete** | Compilation — **all 8 translation units compile** (MSVC x86, 0 errors) |
| Phase 3 | Pending | Executable link: Win32/MFC42 runtime, register/memory model, import bridges |
| Phase 3+ | Pending | Win32/DirectX HAL, GOS engine abstraction, asset loading, gameplay |

## Binary

| Property | Value |
|----------|-------|
| **Target** | `Recoil.exe` (1.21 MB) |
| **Compiler** | Visual C++ 6.0 + MFC42 |
| **Architecture** | x86-32, PE32, image base `0x00400000` |
| **Code (.text)** | `0x00401000` – `0x004C3000` (~795 KB) |
| **DRM** | none (clean) |

## Phase 0: IDA-seeded discovery

Function discovery was bootstrapped with **IDA Pro 9.1**, which follows **vtables** —
so virtual methods reached only through a vtable pointer are captured from the start
(the call-graph discovery used on the sister project Crimson Skies missed ~770 of those).

| Artifact | Value |
|----------|-------|
| `config/functions.json` | **3,490** functions |
| `config/ida_names.txt` | **252** FLIRT-identified names |
| `analysis/vtables.json` | **166** vtables → **517** virtual methods (all in functions.json) |

## Phase 1: x86 → C code generation

`run_pipeline.py` lifts every function via the shared `pcrecomp` lifter, seeding
entries from the IDA list:

| Result | Value |
|--------|-------|
| Functions lifted | **3,490** |
| Lift errors | **0** |
| Output | 321,121 lines of C, 16.7 MB |

The generated `src/recomp/gen/` is **not committed**. Lifted C is a derivative work
of `Recoil.exe`, so it is generated locally from your own copy of the game. The
function list (`config/functions.json`, addresses only) is what makes it
reproducible. Same lifter / `recomp_types.h` as the Crimson Skies project, which
already compiles cleanly.

## Build

Requires Python 3.11 with `capstone`, CMake 3.20+ and MSVC (32-bit target).

```
copy <your install>\Recoil.exe analysis\Recoil.exe   # gitignored, never committed
py -3.11 run_pipeline.py                              # -> src/recomp/gen/recomp_*.c
cmake -B build -A Win32
cmake --build build
```

Today that builds the lifted code as a static library. There is no executable
yet; linking one is Phase 3.

## Toolkit

The lifter and PE analysis come from [pcrecomp](https://github.com/sp00nznet/pcrecomp),
the family's shared recompilation toolkit. `tools/pcrecomp/` is a **vendored
snapshot** taken when this project started (May 2026). It is not kept in sync, and
upstream pcrecomp has moved well past it. Treat upstream as the reference.

## Disc note
The source is a BIN/CUE (Mode2/2352). Converted to ISO with `mdf2iso.py`; the game
files live in an **InstallShield v5** `data1.cab` (header embedded), extracted with
`pcrecomp/tools/assets/isextract.py`. Game data is **not** in this repo — supply your
own legally-obtained copy.

## License

MIT for the code in this repository. See [LICENSE](LICENSE), including its Scope
section: the grant does not cover Recoil itself or the lifted C.
