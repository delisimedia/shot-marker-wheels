# shot-marker-wheels

Prebuilt Python wheels for [Shot Marker](https://github.com/delisimedia/Shot-Marker)'s
AI runtime, hosted as GitHub Release assets.

These are wheels that PyPI doesn't ship for Windows — primarily
[`flash-attn`](https://github.com/Dao-AILab/flash-attention) compiled
for specific GPU architectures + matching torch ABI. Shot Marker's
installer auto-fetches them at install time on hardware that benefits
(currently Blackwell consumer GPUs / RTX 50-series).

## Why

PyPI publishes zero Windows wheels for `flash-attn` (any version).
Building from source on every end user's machine requires CUDA
Toolkit + Visual Studio Build Tools + ~30 minutes of compile time —
not realistic for an installer. We pre-compile once against the exact
torch / Python / CUDA combo bundled in Shot Marker's AI runtime, host
the resulting `.whl` here as a Release asset, and have Shot Marker's
installer download + SHA-256-verify it during setup.

End users on non-Blackwell GPUs or different torch versions skip the
wheel install gracefully and use PyTorch's built-in SDPA mem_efficient
backend, which works fine on every supported GPU.

## What's here

Each release is one wheel for one (`flash-attn version`, `torch
version`, `CUDA version`, `Python version`, `GPU arch`) combination.
Tags follow the pattern:

```
wheels-flash-attn-<version>-cu<cuda>torch<torch>
```

For example:
- `wheels-flash-attn-2.8.3-cu128torch211` — flash-attn 2.8.3 built for
  CUDA 12.8, torch 2.11, Python 3.12, sm_120 (Blackwell consumer).

The wheel filename inside the release follows pip's standard wheel
naming convention so `pip install <filename>` Just Works:

```
flash_attn-<ver>-cp<python>-cp<python>-win_amd64.whl
```

## Provenance

Every wheel is built directly from the upstream source tarball
([Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention)
or whichever upstream the wheel comes from), with the only non-default
flags being `TORCH_CUDA_ARCH_LIST` (to target the right GPU arches)
and `MAX_JOBS` (to cap parallel compile RAM peak).

Each release description carries:
- Upstream version
- Build environment (Python / torch / CUDA versions)
- SHA-256 of the wheel file
- File size

The SHA-256 is pinned in Shot Marker's `installer.py` so a tampered
or substituted release asset is rejected at install time rather than
silently installed.

## Rebuilding

The build scripts live alongside Shot Marker's source under
`bundled_wheels/`. See REBUILD.md there for the rebuild workflow.

After rebuilding:

1. Compute the new SHA-256.
2. Cut a new release on this repo with the new wheel attached.
3. Update `_FLASH_ATTN_WHEEL_URL` + `_FLASH_ATTN_WHEEL_SHA256` +
   `_FLASH_ATTN_WHEEL_SIZE_BYTES` + `_FLASH_ATTN_TORCH_PREFIX` in
   Shot Marker's `installer.py` to point at the new release.

## License

The wheels themselves are compiled binaries of upstream Apache-2.0
licensed source (flash-attention is Apache-2.0). The metadata and
tooling in this repo is MIT.
