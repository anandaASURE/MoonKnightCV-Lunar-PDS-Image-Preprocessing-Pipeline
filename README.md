<img width="112" height="103" alt="image" src="https://github.com/user-attachments/assets/d90dfa40-d78b-4acd-b0e3-ce7df8bbc919" />

# MoonKnightCV: Lunar Shadow Detection & Preprocessing Pipeline

**Automated planetary data engineering and computer vision pipeline for NASA PDS lunar imagery — IMG-to-PNG conversion, overlapping tiling, shadow-mask generation, and spatial metadata propagation.**

> Project focus: *Shadow-Based Sun Direction Estimation* — building clean, tile-level ground-truth data from raw lunar orbital imagery.

---

## Overview

MoonKnightCV automates the end-to-end data preparation workflow for large-scale NASA planetary imagery, transforming raw PDS strips into a clean, ML-ready dataset of tiles, shadow masks, and structured metadata — all driven from a single notebook.

### Core Capabilities

- **Automatic IMG → PNG Conversion** — Converts NASA/ISRO planetary `.img` rasters to standard PNG using a three-tier fallback strategy: GDAL (if installed) → XML-metadata-guided binary reading → auto-detected binary reading (tries common lunar image dimensions, 8-bit and 16-bit grayscale).
- **PDS XML Metadata Parsing** — Recursively parses nested NASA PDS3/PDS4 XML metadata into structured Python dictionaries, with round-trip export back to XML or JSON.
- **Overlapping Stride Tiling Engine** — Crops large raster strips into uniform 256×256 pixel tiles using a 128-pixel stride (50% overlap), so continuous terrain features, crater walls, and long shadows aren't truncated across tile boundaries.
- **Classical CV Shadow Detection** — Global percentile normalization followed by Otsu thresholding and morphological cleanup (`scikit-image` / `scipy.ndimage`) to produce binary shadow ground-truth masks, plus per-tile shadow statistics (coverage, region count, intensity).
- **Spatial Metadata Propagation** — Each tile's metadata preserves its parent image's metadata and adds bounding box, offset, normalization parameters, and shadow-detection statistics.
- **Organized Dataset Generation** — Outputs are automatically routed into dedicated folders for converted PNGs, tiles, masks, metadata, and visualizations.
- **Built-in Inspection Tools** — Functions to list generated metadata files, inspect a single tile's metadata, and compare parent vs. child metadata side by side.

---

## Pipeline Architecture

```
NASA PDS Raw Imagery (.img + .xml)
                 │
                 ▼
      IMG → PNG Conversion
     (GDAL / XML-guided / raw binary)
                 │
                 ▼
     PDS XML Metadata Parsing
                 │
                 ▼
     Overlapping 256×256 Tiling
        (128px stride, 50% overlap)
                 │
                 ▼
   Global Normalization (2nd–98th pct)
                 │
                 ▼
  Otsu Thresholding + Morphology
                 │
                 ▼
       Binary Shadow Masks
                 │
                 ▼
  Spatial + Shadow Metadata Propagation
                 │
                 ▼
    ML-Ready Lunar Tile Dataset
```

---


## Output Visualization Example
<img width="1479" height="511" alt="m103947777lc_tile_0001_viz" src="https://github.com/user-attachments/assets/3f57a8f5-fe83-4ebc-a758-8e061f61e6a9" />

## Repository Structure

```
MoonKnightCV/
│
├── input_images/                 # Place your raw .img / .png / .jpg lunar images here
├── input_metadata/                # Corresponding PDS XML metadata files
│
├── output_png/                    # Auto-generated: PNGs converted from .img files
├── output_tiles/                  # Auto-generated: 256×256 image tiles
├── output_masks/                  # Auto-generated: binary shadow masks
├── output_metadata/                # Auto-generated: per-tile JSON/XML metadata
├── output_visualizations/          # Auto-generated: inspection/visualization outputs
│
├── MAIN__lunar_shadow_preprocessing_pipeline_with_metadata.ipynb   # Main pipeline notebook
├── lunar_shadow_theory - Copy.pdf   # Background theory reference (tile size, normalization, etc.)
│
├── .gitignore
├── LICENSE                        # MIT License
├── README.md
└── requirements.txt
```

> **Note:** `input_images/` and `output_png/` aren't tracked in the repo by default (they're either user-populated or auto-created by the notebook on first run) — the notebook creates any missing output directories automatically.

**Output naming convention:**
- Tiles: `{parent_name}_tile_{index}.png`
- Masks: `{parent_name}_tile_{index}_mask.png`
- Metadata: `{parent_name}_tile_{index}.json` (or `.xml`)

---

## Tech Stack & Dependencies

| Category | Tools / Libraries |
|---|---|
| Language | Python 3.9+ (tested on 3.12) |
| Image I/O | Pillow (`PIL`), NumPy |
| Computer Vision | `scikit-image` (filters, morphology, exposure), `scipy.ndimage` |
| Planetary Image Support | GDAL/OSGEO *(optional — falls back to raw binary reading if unavailable)* |
| Metadata Utilities | `xml.etree.ElementTree`, `xml.dom.minidom`, `json`, `pathlib` |
| Visualization | Matplotlib |

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/MoonKnightCV-Lunar-PDS-Image-Preprocessing-Pipeline.git
cd MoonKnightCV-Lunar-PDS-Image-Preprocessing-Pipeline/MoonKnightCV
```

### 2. Set Up Virtual Environment & Install Dependencies

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

> GDAL is optional. If it isn't installed, the pipeline automatically falls back to XML-guided or auto-detected raw binary reading for `.img` files — you'll see a `GDAL not available` notice, which is expected and non-fatal.

### 3. Run the Pipeline

```bash
jupyter lab MAIN__lunar_shadow_preprocessing_pipeline_with_metadata.ipynb
```

---

## Notebook Walkthrough

The notebook is organized into 10 sequential steps:

1. **Import Libraries** — Loads image processing, XML, and CV dependencies; detects GDAL availability.
2. **Configuration Parameters** — Sets tile size (256), overlap (50%), directory paths, and shadow-detection thresholds.
3. **XML Metadata Utilities** — Parse/save functions for converting between XML and JSON metadata representations.
4. **IMG File Conversion** — Converts planetary `.img` files to PNG via GDAL, XML-guided binary reading, or auto-detected binary reading.
5. **Image Tiling with Metadata Generation** — Crops images into overlapping 256×256 tiles and generates matching per-tile metadata.
6. **Batch Processing** — Runs the full pipeline across every image found in `input_images/`.
7. **Run the Pipeline** — Executes `process_all_images()` end-to-end.
8. **Process Single Image (Example)** — Demonstrates running the pipeline on one image for debugging/inspection.
9. **Inspect Metadata** — Utilities to list, inspect, and compare generated metadata files.
10. **Example Usage** — Sample calls demonstrating the inspection functions above.

### Per-Tile Metadata Contents

Each generated tile metadata file includes:
- **Tile Info** — ID, parent image name, tile and mask filenames.
- **Spatial Info** — Tile position (x, y), size, and parent image dimensions.
- **Preprocessing Info** — Normalization method, percentile clip values, mean intensity.
- **Shadow Detection Info** — Method used, shadow statistics, and detected region count.
- **Timestamp** — When the tile was processed.

---

## Usage Summary

1. Place source images in `input_images/` (PNG, JPG, or `.img` format).
2. Place matching XML metadata files in `input_metadata/`.
3. Run all cells, or call `process_all_images()` directly.
   - `.img` files are automatically converted to PNG first.
   - All PNGs are then tiled, masked, and given propagated metadata.
4. Inspect results in `output_tiles/`, `output_masks/`, `output_metadata/`, and `output_visualizations/`.
5. Use the built-in inspection functions (`list_all_metadata_files()`, `inspect_metadata_file()`, `compare_parent_child_metadata()`) to verify output quality.

---

## Reference Data

NASA PDS lunar raster files (`.img`) and their matching metadata labels (`.xml`) can be obtained from:
- [NASA ODE](https://ode.rsl.wustl.edu/)
- [LROC Image Search Portal](https://www.lroc.asu.edu/)

For background on the design choices (tile size selection, normalization strategy, etc.), see `lunar_shadow_theory - Copy.pdf` in the repository root.

---

## Target Applications

- Building clean ground-truth training datasets for deep learning segmentation models (U-Net, Mask R-CNN, YOLOv8-Seg).
- Lunar landing hazard detection and solar illumination mapping.
- Shadow-based sun direction estimation for planetary geology workflows.
- Automated feature extraction for geospatial data pipelines.

---

## License

Distributed under the MIT License. See [`LICENSE`](./LICENSE) for details.
