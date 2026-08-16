# Three.js Water Simulation Shootout

Three LLMs were given the exact same prompt and asked to produce a **single self-contained HTML file**. This folder holds their three independent outputs.

Original prompt by [@WescheNex1q](https://x.com/WescheNex1q).

## The prompt

> Create a Three.js water simulation.
>
> Requirements:
> - Plane mesh with 512x512 subdivisions
> - Wave propagation simulation
> - Mouse click creates ripples
> - Multiple interacting waves
> - Real-time lighting
> - Everything in a single HTML file
> - No external libraries besides Three.js

## The outputs

| File | Model | Wave simulation approach | Lighting |
|------|-------|--------------------------|----------|
| [`ds4f.html`](./ds4f.html) | DeepSeek v4 Flash | Analytic (no sim grid): sum of travelling sines + up to 64 Gaussian ripple rings, all evaluated in a custom **vertex shader** with analytic normals | Custom GLSL: moving sun, Fresnel shading, Blinn–Phong specular, canvas-gradient sky + fog |
| [`glm53.html`](./glm53.html) | GLM-5.3 | **GPU height-field** wave equation (Elias scheme) on 3 ping-pong render targets with fixed 120 Hz timestep, damping and absorbing borders | Custom GLSL: procedural dusk sky dome, Fresnel reflection of sky, sun disc/halo, sun specular |
| [`qwen38-27b.html`](./qwen38-27b.html) | Qwen3.8-27B | **CPU finite-difference** wave equation (`h' = 2h − h_old + K·∇²h`, damped + clamped) over 513×513 floats; per-vertex normal rebuild every frame | Three.js PBR: `MeshStandardMaterial` with PMREM env map, ACES tone mapping, shadow-casting moving sun + buoy prop |

## How to run

Open any file in a browser — no build step, no install:

```bash
python3 -m http.server 8000
# then → http://localhost:8000/ds4f.html
```

Files that use the `type="importmap"` npm-style loader may fail when opened via `file://` — serve them over HTTP.

## Comparison

| | DeepSeek v4 Flash | GLM-5.3 | Qwen3.8-27B |
|---|---|---|---|
| Three.js version | 0.170.0 (unpkg) | 0.160.0 (unpkg) | 0.160.0 (jsDelivr) |
| Mesh | 512×512 segments, 513×513 verts | 512×512 segments, sim tex 512² | 512×512 segments, 263,169 verts / 524,288 tris |
| Wave physics | Analytic (shader-only) | GPU wave equation (render targets) | CPU wave equation (typed arrays) |
| Interactivity | click/drag ripple, drag orbit, wheel zoom | click ripple, drag wake, right-drag orbit, zoom, `R` rain, `Space` burst | click/drag ripples, `1` sun sweep, `2` burst, `3` wind, `R` calm |
| Extras | HUD (fps / ripples / verts), demo ripples on load | HUD, procedural sky, demo sequence | PBR + shadows, bobbing buoy, auto-drips |
| WebGL requirement | WebGL1/2 | **WebGL2** (float targets) | WebGL2 (uses PMREM) |
| Diagnostics | `window.__WATER` probe, `__WATER_INFO()` | `window.__water` probe | `window.__probe` probe |

**Takeaways**

- **DeepSeek v4 Flash** produced the smallest, shader-cheapest version: waves are fully analytic, so there is no simulation cost and no texture plumbing — but "wave propagation" is procedural, not a real physics field.
- **GLM-5.3** implemented a genuine GPU water simulation with ping-pong render targets and a fixed timestep for consistent wave speed, plus the richest environment (dusk sky dome, sun, halo).
- **Qwen3.8-27B** wrote the heaviest-engineered solution: a real CPU simulation with clamping/damping, PBR materials, environment maps, tone mapping and shadow-casting lights, at the price of per-frame CPU/GPU sync of 263k vertices.

All three satisfy every line of the prompt, in three very different architectures.
