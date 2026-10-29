# Singularity / Galdrix

Singularity is the visual coding client. Galdrix is the sandboxed WebAssembly
runtime underneath it.

This repository is the project entry point and documentation. The source is
kept in three independent Rust workspaces, each in its own repository. This
repository contains only this README and the project license, and **no git
submodules**: clone the three repositories as sibling directories as described
below.

## Source repositories

| Directory | Repository | Contents |
|---|---|---|
| `galdrix/` | [GaldrixProject/GaldrixRuntime](https://github.com/GaldrixProject/GaldrixRuntime) | `galdrix-idl`, `galdrix-metadata`, `construct`, `galdrix-repository`, `galdrix-runtime` and the `galdrix` executable |
| `visual/` | [GaldrixProject/GaldrixVisual](https://github.com/GaldrixProject/GaldrixVisual) | `galdrix-visual-protocol`, `galdrix-visual-daemon`, `galdrix-visual-media`, the visual client SDK and its widgets |
| `singularity/` | [GaldrixProject/Singularity](https://github.com/GaldrixProject/Singularity) | `singularity-client`, the visual coding client |

All three repositories use `fasttrack` as their default branch. `trunk` keeps
the initial history. The repositories are private to the GaldrixProject
organization.

## Placement required to build

`singularity/client` depends on the visual SDK through path dependencies
(`../../visual/client`, `../../visual/widgets/player`,
`../../visual/widgets/markdown`, `../../visual/protocol`), and the visual crates
depend on each other through `../protocol`, `../daemon` and `../media`. The
directory names below are therefore required; the three repositories must be
siblings:

```
Galdrix/
  galdrix/
  visual/
  singularity/
```

Clone them into that layout:

```sh
mkdir -p Galdrix
cd Galdrix
git clone https://github.com/GaldrixProject/GaldrixRuntime.git galdrix
git clone https://github.com/GaldrixProject/GaldrixVisual.git visual
git clone https://github.com/GaldrixProject/Singularity.git singularity
```

Each workspace is an independent repository with its own `Cargo.toml` and
`Cargo.lock`; update them independently with `git -C galdrix pull` and so on.
Do not add them as submodules.

## Building

```sh
(cd galdrix && cargo build)
(cd visual && cargo build)
(cd singularity && cargo build)
```

Tests:

```sh
(cd galdrix && cargo test)
(cd visual && cargo test)
(cd singularity && cargo test)
```

Desktop manifests:

```sh
(cd visual && cargo build --release -p galdrix-visual-daemon -p galdrix-visual-media)
(cd singularity && cargo build --release -p singularity-client)
```

Example development launch:

```sh
(cd visual && cargo run -p galdrix-visual-daemon -- --socket /tmp/galdrix-visual.sock)
(cd singularity && cargo run -p singularity-client -- /tmp/galdrix-visual.sock)
```

On Linux, install `libpulse-dev` and the FFmpeg development libraries
(`libavformat-dev`, `libavcodec-dev`, `libavutil-dev`, `libswscale-dev`,
`libswresample-dev`) before building. SDL3 is built statically from the vendored
source; the headers let its PulseAudio driver use the current default
PulseAudio/PipeWire sink. The media helper links the FFmpeg libraries directly
and never invokes the `ffmpeg`/`ffprobe` executables. Desktop deployment uses
the distribution's FFmpeg runtime libraries; the launcher checks for missing
shared objects and reports the package requirement. The same helper also
performs still-image decoding, so it links the FFmpeg runtime in every mode;
image-only deployments still need the FFmpeg shared libraries.

## Knowledge base

Each repository owns its documentation under `doc/kb/` and indexes it from
`doc/kb/README.md`:

| Repository | Knowledge base |
|---|---|
| `galdrix/` | Runtime, heap, memory, IDL and Wasmtime notes |
| `visual/` | Protocol, daemon, composition, GPU process, text stack and window API notes |
| `singularity/` | Client UI, control catalog and code review notes |

## Runtime data

Sessions live under `~/.local/state/singularity/`, and settings under
`~/.config/singularity/settings.json`. Legacy `galdrix-agent` data has been
migrated and removed.

## License

MIT. See [LICENSE](LICENSE).
