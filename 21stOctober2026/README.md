# MaterialX Graph Editor — GLSL vs Cycles A/B sweep

Open `index.html` to eyeball all comparisons. The page links each image to its
full-size PNG and includes per-material logs.

## Scope

- All 50 documents under `resources/Materials/Examples`.
- Cycles: 16 samples, OpenImageDenoise enabled (`--denoiseStartSample 8`).
- GLSL: single frame.
- Resolution: 512x512.
- Default shaderball geometry and default Graph Editor lighting.

Command template:

```bash
MaterialXGraphEditor \
  --material <file.mtlx> \
  --viewWidth 512 --viewHeight 512 \
  --renderBackend Cycles \
  --samples 16 --denoise true --denoiseStartSample 8 \
  --captureFilename <out.png>
```

For GLSL, omit the Cycles-only options and use `--renderBackend GLSL`.

## Method notes

The first sweep ran three renders in parallel. This races the first OSL shader
build: some captures were written before the material became active, producing
black frames or occasional crashes. Materials that looked black or crashed were
rerun sequentially. Individual cards note when a sequential rerun was used.

## Materials that need attention

| Material | Result |
|---|---|
| `Examples/SimpleHair/simple_hair_default.mtlx` | Unstable: sequential runs produced either a fully black frame or a segfault (`exit 139`). |
| `Examples/StandardSurface/standard_surface_thin_film.mtlx` | Cycles segfaults consistently (`exit 139`). |
| `Examples/OpenPbr/open_pbr_ketchup.mtlx` | Cycles renders nearly white; red transmission/absorption is lost. |
| `Examples/StandardSurface/standard_surface_jade.mtlx` | Cycles renders pale white with pink edges; subsurface/transmission is lost. |
| `Examples/OpenPbr/open_pbr_honey.mtlx` | Cycles renders clear glass; amber volume absorption is lost. |
| `Examples/OpenPbr/open_pbr_pearl.mtlx` | Cycles renders plain white; the pearlescent coating is lost. |

The parallel sweep's black gold/chrome/marble/USD-glass frames and the Disney
car-paint crash were capture races; those materials render correctly when run
individually.

## Files

- `index.html` — side-by-side gallery, filterable by name, with a “show
  failures/blank only” toggle.
- `results.csv` — per-backend status, timings, luminance, object coverage, and
  black-frame flags.
- `images/<id>/{glsl,cycles}.png` — captures.
- `logs/<id>.<backend>.log` — stdout/stderr from each render.
- `work/<id>/<backend>/` — per-render working directory, including generated OSL.
