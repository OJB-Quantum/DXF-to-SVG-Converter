# DXF-to-SVG-Converter

A tool that runs in the browser using Google Colab and a GPU to identify, flatten, and convert a DXF or OASIS layout into an SVG output. Control knobs are included for easy parameter adjustment. The Colab code is open source, written by Onri Jay Benally in 2026.

---

You can use a free GPU in Google Colab as needed.

## Click to use the DXF-to-SVG converter in Google Colab: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/OJB-Quantum/DXF-to-SVG-Converter/blob/main/DXF_to_SVG_Converter.ipynb)

---

## Overview

When exporting layout geometries from Electronic Design Automation (EDA) and Computer-Aided Design (CAD) environments, geometric data is often nested in complex hierarchies, obscured by display policies, or accidentally saved into "orphaned" block libraries rather than the primary visible canvas. 

This script provides a robust, fail-safe pipeline to parse `.dxf` and `.oas` files, hunt down the geometric data wherever it resides, flatten it to a single 2D plane, and render it into a high-fidelity Scalable Vector Graphics (SVG) file suitable for presentations, papers, and web viewing.

## Key Features

* **3-Tier Deep Geometry Scanner:** Prevents the "blank output" problem common with standard converters. If the primary Model Space is empty, it automatically scans all Paper Space layouts, and finally probes the internal Block Libraries for orphaned EDA components.
* **Automated Hierarchy Flattening:** Iteratively explodes nested `INSERT` blocks (up to 15 levels deep) to expose raw geometry.
* **Dual Format Support:** Processes CAD drawing files (`.dxf`) via `ezdxf` and semiconductor masking files (`.oas` / `.oasis`) via `gdstk`.
* **Visual Control Knobs:** Easily toggle between transparent or colored backgrounds, and choose whether to preserve native CAD layer colors or force high-contrast monochrome rendering.
* **GPU Acceleration:** Includes an optional CUDA warmup via `cupy` to rapidly dispatch array operations within the Colab environment.

---

## The Control Knobs

At the top of the script, you will find a dedicated `# CONTROL KNOBS` section. These parameters are exposed for immediate adjustment without needing to alter the core pipeline logic.

### Display & Environment
* `DEFAULT_SVG_WIDTH_PX` (int): Sets the baseline resolution width of the output SVG (Default: `3000`).
* `ENABLE_GPU_WARMUP` (bool): Toggles CuPy CUDA context initialization (Default: `True`).
* `DPI_SETTING` (int): Enforces a strict Dots Per Inch resolution for backend plotting (Default: `250`).

### Visual Render Controls
* `USE_TRANSPARENT_BACKGROUND` (bool): If `True`, the SVG background is stripped out for seamless document integration.
* `ALTERNATIVE_BACKGROUND_COLOR` (str): The HEX code applied to the canvas if transparency is disabled (Default: `"#FFFFFF"`).
* `KEEP_ORIGINAL_COLORS` (bool): If `True`, native DXF/OASIS layer colors are preserved. If `False`, everything is forced to a single color for clarity.
* `FORCED_GEOMETRY_COLOR` (str): The HEX code applied to all geometry if original colors are not kept (Default: `"#000000"`).

### Flattening Target Parameters
* `TARGET_DXF_LAYER` (str): The destination layer name for flattened DXF geometry (Default: `"0"`).
* `TARGET_OASIS_LAYER` (int): The destination layer index for flattened OASIS geometry (Default: `0`).

---

## How It Works (Under the Hood)

1.  **Disk-Backed Parsing:** Uploaded files are temporarily written to the Colab disk to bypass strict ASCII/UTF-8 string decoding errors inherent to legacy CAD text encodings.
2.  **Adaptive Targeting:** The engine evaluates the spatial database. If the `modelspace()` returns 0 entities, it traverses layout viewports. If viewports are empty, it searches for the largest isolated component in the block reference inventory.
3.  **Entity Explosion:** A `while` loop aggressively queries and explodes blocks until only fundamental primitives (lines, polygons, paths, text) remain.
4.  **Policy Assignment:** User-defined background policies (`BackgroundPolicy`) and color mapping overrides (`ColorPolicy`) are injected into the render context.
5.  **Extents Validation:** If the native bounding box is mathematically blank, the script injects an invisible layout boundary vector to force the SVG scaling engine to execute correctly.
6.  **Inline Render & Delivery:** An HTML block displays the output directly in the Jupyter/Colab cell for immediate verification, followed by an automated browser payload triggering the `.svg` download.

---

## Dependencies

The script will automatically install these packages in the first Colab cell:
* `ezdxf`
* `gdstk`
* `cupy-cuda12x`
* `matplotlib`
