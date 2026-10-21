# MaterialX Graph Editor — GLSL vs Cycles (lightIntensity 0.5)

Open `index.html` for the side-by-side gallery. Every image links to its
full-size PNG, and per-material logs are included.

## Scope

- All 50 documents under `resources/Materials/Examples`.
- Cycles: 16 samples, OpenImageDenoise enabled (`--denoiseStartSample 8`).
- GLSL: single frame.
- Both backends: `--lightIntensity 0.5`.
- Resolution: 512x512.
- Default shaderball geometry and Graph Editor lighting.

Command template:

```bash
MaterialXGraphEditor \
  --material <file.mtlx> \
  --viewWidth 512 --viewHeight 512 \
  --renderBackend Cycles \
  --lightIntensity 0.5 \
  --samples 16 --denoise true --denoiseStartSample 8 \
  --captureFilename <out.png>
```

For GLSL, omit the Cycles-only options and use `--renderBackend GLSL`.

## Method notes

The sweep was run sequentially. A capture can still race the first OSL shader
build in this build, so any black frame or crash was rechecked individually.
`disney_principled_default` produced one flaky black capture during the sweep;
three individual reruns were correct and the good frame is used here.

## Materials that need attention

| Material | Result |
|---|---|
| `Examples/SimpleHair/simple_hair_default.mtlx` | Cycles frame is black in 3/3 sequential reruns. |
| `Examples/StandardSurface/standard_surface_thin_film.mtlx` | Cycles segfaults consistently (`exit -11`). |
| `Examples/OpenPbr/open_pbr_ketchup.mtlx` | Cycles renders nearly white; red transmission/absorption is lost. |
| `Examples/StandardSurface/standard_surface_jade.mtlx` | Cycles renders pale white with pink edges; subsurface/transmission is lost. |
| `Examples/OpenPbr/open_pbr_honey.mtlx` | Cycles renders clear glass; amber volume absorption is lost. |
| `Examples/OpenPbr/open_pbr_pearl.mtlx` | Cycles renders plain white; the pearlescent coating is lost. |

Lowering `--lightIntensity` to 0.5 reduces the overall exposure and brings the
default materials closer to GLSL, but does not change the feature losses above.

## Files

- `index.html` — comparison gallery with name filter and “show failures only”.
- `results.csv` — status, timing, luminance, object coverage, black-frame flags.
- `images/<id>/{glsl,cycles}.png` — captures.
- `logs/<id>.<backend>.log` — stdout/stderr from each render.
