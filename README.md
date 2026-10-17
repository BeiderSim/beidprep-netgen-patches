# beidprep-netgen-patches

Patches to the **Netgen** mesh generator (<https://github.com/NGSolve/netgen>), used by **BeidPrep** by
Konstantin Arhiptsov (Beider Simulations).

## Upstream version

The patches apply to **Netgen v6.2.2608** — git tag `v6.2.2608`, commit
`96e5682f6ea43ba77ba3bb2ae4bb4bd1791f506e` (1 October 2026) — and are built against **OpenCASCADE 7.7.2**
(conda-forge `occt=7.7.2`).

## The patches

| File | What it changes |
|---|---|
| `patches/0001-From_PyOCC-accept-TopoDS-subtypes.patch` | `netgen.occ.From_PyOCC` also accepts pythonocc's `TopoDS_Solid`, `TopoDS_Shell`, … wrappers, not only the base `TopoDS_Shape` wrapper (the same object underneath). |
| `patches/0002-GenericImprove-walk-face-elements-only.patch` | The 2D surface optimisation walks only the current face's elements instead of every element of the body for each face (time grew with the square of the face count). Same result, less time. |
| `patches/0003-skip-curvature-sizing-when-off.patch` | With curvature-based sizing switched off, the OpenCASCADE triangulation that only feeds that sizing is skipped (it used a thread pool that can deadlock inside a forked worker process). |
| `patches/0004-keep-caller-meshed-faces.patch` | Faces that already hold surface elements when surface meshing starts are kept as they are and are not re-meshed or optimised. |
| `patches/0005-per-face-time-limit.patch` | New meshing parameter `facetimeout` (seconds per face attempt): a face that exceeds it is given up cleanly (no half-meshed face) and the rest of the body is still meshed. |
| `patches/0006-illegal-triangle-table-once.patch` | The table of illegal (duplicate) triangles read by the 2D optimisation is built once after all faces instead of after every face (the same table, so the same mesh; less time). |

## Licence

Netgen is distributed under the **GNU Lesser General Public License, version 2.1** (see `LICENSE`, Netgen's own
licence text). These patches are modifications of Netgen and are **released under the same licence, the GNU LGPL
version 2.1**.

## How to apply

```bash
git clone --branch v6.2.2608 --depth 1 --recurse-submodules --shallow-submodules \
    https://github.com/NGSolve/netgen netgen-src
cd netgen-src
for p in ../patches/*.patch; do git apply --ignore-whitespace --whitespace=nowarn "$p"; done
```

On Windows (PowerShell), clone with `git -c core.autocrlf=false clone --config core.autocrlf=false ...` so
Netgen's LF line endings are kept, then:

```powershell
Get-ChildItem ..\patches\*.patch | Sort-Object Name | ForEach-Object { git apply --ignore-whitespace --whitespace=nowarn $_.FullName }
```

## How to build

A Python environment with OpenCASCADE 7.7.2 and its CMake files (for example conda-forge
`occt=7.7.2 pythonocc-core=7.7.2 cmake ninja`), then, from an empty build folder:

```bash
cmake -G Ninja ../netgen-src \
    -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=<install folder> \
    -DUSE_SUPERBUILD=ON -DUSE_GUI=OFF -DUSE_PYTHON=ON -DUSE_OCC=ON \
    -DUSE_NATIVE_ARCH=OFF -DBUILD_STUB_FILES=OFF \
    -DCMAKE_PREFIX_PATH=<env> -DOpenCascade_DIR=<env>/lib/cmake/opencascade \
    -DPython3_EXECUTABLE=<env>/bin/python -DPython3_ROOT_DIR=<env> -DPython3_FIND_REGISTRY=NEVER
ninja
ninja install
```

On Windows: run from the "x64 Native Tools Command Prompt for VS 2022", use `<env>\Library\lib\cmake\opencascade`
and `<env>\python.exe`, and also set the environment variable `Python3_ROOT_DIR=<env>` before running CMake, so
that every part of the build links the environment's own Python and not another Python found on the machine.
Measured: about 5 minutes on a 16-thread laptop.
