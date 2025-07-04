# LOONYTOONY
OPEN SOURCE AI TUNING PLATFORM
# Project: Open-Source AI Tuning Platform for 2015 Mazda 3 2.0 Skyactiv-G (PE-VPS, Mitsubishi ECU) AN EVENTUALLY EVER MAKE AND MODEL

## Mission
Write clean, well-documented Python tooling that **extracts, labels, and exports all calibration maps** (AFR, ignition, VVT, torque/load limits, etc.) from raw ECU binaries and .vtune/versatuner containers so they can be fed into an AI model for real-time tuning suggestions.

## Inputs
1. `firmware.bin` – full PE-VPS ROM read (Mitsubishi MH8106F)  
2. `tune.vtune` – VersaTuner package (zipped XML + binary segments)  
3. Known axis values & screenshots (folder: `/reference_screens/`)  
4. J2534 log snippets for reflash routines (optional)

## High-Level Tasks
- **Static analysis**  
  - Use `ghidra` (invoke via `ghidra_bridge`) to locate map tables, axis arrays, checksum routines, and map pointers.  
  - Identify table metadata blocks (Mitsubishi table header pattern `xx xx 3F F0 xx xx`).  
- **Binary parsing helpers**  
  - Build a `Map` class that stores: name, start offset, dimensions, endianess, scaling, units.  
  - Provide `read_map()` and `write_map()` that return/accept `pandas.DataFrame` and auto-export to CSV.  
- **Checksum patching**  
  - Detect CRC blocks, recompute CRC-16/32, and patch on save so modified ROMs stay flashable.  
- **.vtune unpacker**  
  - Parse the ZIP, extract each `table.xml`, decode base64 payloads, convert to the same `Map` object.  
- **CLI** (`python -m ai_tuner …`)  
  - `list-maps` → prints all detected maps + axes + units.  
  - `export-all` → writes every map as `csv` into `/maps_original/YYYY-MM-DD/`.  
  - `diff --old A.csv --new B.csv` → outputs delta CSV ready for VersaTuner import.  
- **ML hooks**  
  - Optional: add `dataset_builder.py` that merges map CSVs + log CSVs into a supervised dataset for an LSTM/Transformer model.

## Coding Conventions
```python
# Use Python 3.11+
from __future__ import annotations
import struct, pathlib, pandas as pd, numpy as np
