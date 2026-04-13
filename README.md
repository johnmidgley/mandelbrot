# Mandelbrot Explorer

A real-time, GPU-accelerated Mandelbrot set explorer that runs entirely in the browser as a single HTML file. All fractal computation happens in WebGL 2 fragment shaders, rendering every pixel in parallel on the GPU.

## Features

- **Real-time GPU rendering** — WebGL 2 fragment shaders compute the fractal at 60fps
- **Double-single precision** — Emulates ~48-bit float precision on the GPU by splitting each coordinate into two 32-bit floats, enabling zoom depths up to ~10^14 (vs ~10^7 with standard float32)
- **Multiple fractal types** — Mandelbrot, Julia set, Burning Ship, Tricorn
- **Arbitrary power** — Change the exponent in z^n + c from 1 to 8 for Multibrot variations
- **7 color schemes** — Smooth Rainbow, Fire, Ocean, Electric, Psychedelic, Grayscale, Neon Glow
- **Smooth coloring** — Normalized iteration count eliminates banding
- **Color animation** — Cycle colors in real-time
- **Adjustable parameters** — Max iterations, escape radius, color cycle speed/offset, zoom speed
- **Interior coloring** — Black, white, or orbit-trap-based coloring for the set interior
- **Navigation** — Scroll to zoom (toward cursor), drag to pan, click to center
- **Julia set picking** — Shift+click any point to use it as the Julia constant
- **Touch support** — Pinch to zoom, drag to pan on mobile/tablet
- **Retina display** — Renders at native devicePixelRatio
- **Export** — Save the current view as PNG

## Usage

Open `index.html` in any modern browser. No build step, no dependencies.

```
open index.html
```

### Controls

| Input | Action |
|---|---|
| Scroll wheel | Zoom in/out toward cursor |
| Click + drag | Pan |
| Click | Center on point |
| Shift + Click | Set Julia constant from that point |
| Pinch (touch) | Zoom |

All parameters are adjustable via the control panel (toggle with the "Controls" button).

## How it works

The entire fractal is computed in a GLSL fragment shader. A fullscreen quad is drawn, and the fragment shader runs the escape-time iteration for each pixel independently across thousands of GPU cores.

### Double-single precision

Standard GPU floats are 32-bit, limiting zoom to about 10^7 before pixels quantize to the same coordinate. This explorer uses **double-single arithmetic**: each coordinate is represented as two floats `(hi, lo)` where `hi + lo` equals the full-precision value. JavaScript (which uses float64) splits the center coordinates into these pairs, and the shader reconstructs precise per-pixel positions using compensated addition and multiplication. This extends usable zoom to approximately 10^14.

## Potential improvements

### Perturbation theory (unlimited zoom depth)

The current double-single approach hits its limit around 10^14 zoom. For truly unlimited depth, **perturbation theory** is the standard technique:

1. Compute a single **reference orbit** at the center of the view using arbitrary-precision arithmetic on the CPU (e.g., using a BigDecimal library)
2. For each pixel, compute only the tiny **perturbation** (delta) from the reference orbit
3. The deltas remain small enough for standard float32 on the GPU

This keeps GPU rendering fast while allowing mathematically unlimited zoom. All modern deep-zoom software (Kalles Fraktaler, Mandel Machine) uses this approach.

### Series approximation

An extension of perturbation theory: approximate the perturbation with a polynomial Taylor series to **skip thousands of iterations** for pixels near the reference point. This makes extreme depths (10^100+) feasible in reasonable time.

### Adaptive iteration count

Automatically increase max iterations as zoom depth increases. A heuristic like `maxIter ≈ log2(zoom) × 50` keeps detail visible without manual adjustment.

### Distance estimation

Use the derivative of the iteration to estimate distance to the set boundary. This enables:
- **Anti-aliasing** — smooth edges without supersampling
- **Adaptive detail** — skip iteration for pixels far from the boundary
- **3D rendering** — use distance as a height field

### Orbit traps

Instead of coloring by iteration count, color by how close the orbit comes to a geometric shape (circle, cross, line). Produces striking artistic effects with minimal performance cost.

### Web Workers for CPU fallback

For browsers without WebGL or for arbitrary-precision computation, distribute work across CPU cores using Web Workers. Each worker handles a tile of the image.
