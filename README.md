# UnpackEA Graphics Library — Wii

> Extract, parse, and convert Wii EA Graphics Library (EAGL) 3D models and terrains into modern GLB files.


---

## Overview

**UnpackEA Graphics Library — Wii** is a reverse-engineered toolkit for reading and converting proprietary EA Graphics Library (EAGL) assets from Wii game titles. It decodes binary 3D model and terrain data from EA's closed format and reconstructs them as standards-compliant **GLB (glTF Binary)** files — ready for use in Blender, Unity, Unreal Engine, or any modern 3D pipeline.

This project bridges the gap between legacy console game assets and modern tooling, enabling preservation, research, and creative reuse of Wii-era EA game content.

---

## Features

- **Binary EAGL Parsing** — Deep binary parsing of EA's proprietary Wii graphics format
- **Terrain Extraction** — Full reconstruction of large-scale game terrain meshes
- **Character Model Export** *(In Development)* — Skeletal mesh extraction with geometry and UV mapping
- **GLB Output** — Industry-standard glTF Binary output compatible with all major 3D tools
- **Mesh Fidelity** — High-accuracy vertex, normal, and UV data reconstruction
- **Data-Driven Layout Detection** — Descriptor layouts are defined in an external JSON config, making it easy to support new mesh formats without touching parser code

---

## Gallery
### Character Models
<img width="2112" height="1139" alt="Screenshot 2026-06-19 222532" src="https://github.com/user-attachments/assets/ad835815-c3ff-4c4c-8bd2-f6ef54409592" />

### Terrain & World Geometry
<p align="center">
  <img src="https://github.com/user-attachments/assets/685c1258-28e8-4701-9211-1a41e5e1690e" width="49%" />
  <img src="https://github.com/user-attachments/assets/35a0fdb1-4799-4cb2-8f8e-f9424e142147" width="49%" />
</p>

### 3D Model Extraction

<p align="center">
  <img src="https://github.com/user-attachments/assets/71defffe-eb6c-43ff-8fe5-a15fedbabbac" width="49%" />
  <img src="https://github.com/user-attachments/assets/a849cbdc-e127-4911-b255-df8b6879ba48" width="49%" />
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/7977d7e1-75d6-4683-b476-0b0257402a18" width="49%" />
  <img src="https://github.com/user-attachments/assets/5ba5a0cc-1e86-4be0-a89e-ef2edb31c19f" width="49%" />
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/ea0c8419-4577-46d1-a399-42ae9b2b9741" width="49%" />
  <img src="https://github.com/user-attachments/assets/d2b48cc6-6ffa-4a23-9b26-b99523091775" width="49%" />
</p>

---

## Parser & Terrain Versions

The terrain and geometry parser evolved through several versions, each addressing accuracy issues discovered through deeper reverse-engineering of the EAGL binary format.

---

### Parser v4 — GX Scan Window Fix

**Key fix:** Corrected a critical mesh boundary bug in the GX display list scanner.

Previously, the GX scan window was bounded by `vtx_count * stride + 2048` — an estimate that routinely overran into adjacent meshes' float and byte data. Embedded `0x98` bytes in that neighboring data were misread as GX triangle-strip headers, producing spurious triangles with out-of-range normal indices. This manifested as visible **mesh tearing and corrupted faces** on complex geometry.

The fix bounds the scan window by the next mesh's actual position-array pointer instead. Additionally, descriptors are now **sorted by their pos-array pointer** before GX boundaries are computed, ensuring correct ordering regardless of descriptor discovery order. The `_parse_mesh` function now accepts an explicit `gx_end` argument rather than estimating it from the descriptor's vertex count field.

---

### Parser v6 — Layout C: Track & Terrain Geometry

**New support:** `Layout C` — identified by `byte[0] == 0x02` — covers track and terrain geometry files such as *TestTrackforTodd*.

Layout C differs from Layouts A and B in several important ways:

- The descriptor contains a **non-aligned reloc at `desc+0x09`** that points directly to the GX display list, rather than deriving the GX start from the UV array end.
- The `ptr_pos`, `ptr_norm`, and `ptr_uv` slots sit at the same offsets as Layout A (`+0x34` / `+0x3c` / `+0x44`), but **array counts are derived from pointer gaps** rather than being stored in the descriptor — the descriptor's count fields hold unrelated data in this layout.
- Normal stride is **4 bytes** (`s8×3` + 1 pad byte) instead of the 6-byte stride used in Layouts A/B.
- The GX end boundary is still determined by the next descriptor's `ptr_pos`.

**Detection** requires `byte[0] == 0x02`, a non-aligned reloc at `desc+0x09` pointing to a plausible GX address (greater than `ptr_uv`), and the standard `+0x34` / `+0x3c` / `+0x44` reloc triple.

---

### Parser v7 — Data-Driven Layout Detection

**Architecture overhaul:** Layout definitions and detection logic were extracted from the parser into `eagl_layouts.json`.

Each layout now carries a list of `signature_checks` — supporting check types: `magic`, `reloc_exists`, `reloc_order`, `count_range`, and `reloc_absent`. Detection no longer performs a hard magic-byte dictionary lookup. Instead, every defined layout is **scored against every candidate descriptor offset**, and the highest-scoring layout wins if it clears a minimum score threshold (`MIN_SCORE`) and leads the runner-up by at least a defined margin (`MARGIN`).

This scoring-based approach makes it straightforward to add support for new layouts by editing JSON alone, with no changes to parser code.

---

## In Development — Character Export

Character model extraction is actively being developed. Each iteration improves mesh accuracy, UV reconstruction, and geometry fidelity.

---

### Version 9 *(Current)*
<p align="center">
  <img src="https://github.com/user-attachments/assets/7c32c93f-dca1-4447-80a9-31ce72376ced" width="80%" />
</p>
<img width="2112" height="1139" alt="Screenshot 2026-06-19 222532" src="https://github.com/user-attachments/assets/ad835815-c3ff-4c4c-8bd2-f6ef54409592" />

Continued refinement of character geometry extraction. Builds on the UV and normal reconstruction improvements from v8, with further accuracy gains in mesh topology and edge cases in the GX display list reader for character-specific descriptors.

<details>
<summary>Previous Versions</summary>

### Version 8

<p align="center">
  <img src="https://github.com/user-attachments/assets/99add145-d321-42a0-bb0f-025e8cae6fbf" width="49%" />
  <img src="https://github.com/user-attachments/assets/352a5630-2043-42df-99ca-e2fdfe68da1c" width="49%" />
</p>

Improved UV mapping reconstruction and normal data handling for character meshes. Addressed seam artifacts and per-vertex attribute misalignment carried over from v7.

---

### Version 7

<p align="center">
  <img src="https://github.com/user-attachments/assets/d6015233-1fd4-4586-97f6-2b437e131764" width="80%" />
</p>

First version to focus specifically on character model extraction, leveraging the data-driven layout detection system introduced in the parser rewrite. Initial geometry reconstruction with basic UV support.

---

### Version 6

<p align="center">
  <img src="https://github.com/user-attachments/assets/0696b5ae-ec83-47bb-abde-2701ef331e9d" width="80%" />
</p>

Final terrain-focused parser version before the character model work began. Introduced Layout C support for track/terrain geometry and the `eagl_layouts.json` scoring system.

</details>

---

## Getting Started

> Documentation and usage instructions coming soon.

```bash
# Clone the repository
git clone https://github.com/sirshoqckings/UnpackEA-Graphics-Lirbray-Wii.git
cd UnpackEA-Graphics-Lirbray-Wii
```

---

## Roadmap

- [x] Terrain mesh extraction
- [x] Static 3D model extraction
- [x] GLB export pipeline
- [x] Layout C support (track/terrain geometry)
- [x] Data-driven layout detection (`eagl_layouts.json`)
- [ ] Character model extraction *(in progress)*
- [ ] Skeletal / bone data support
- [ ] Texture extraction and embedding
- [ ] Full CLI interface

---

## Contributing

Contributions, bug reports, and reverse-engineering findings are welcome. Feel free to open an issue or submit a pull request.

---

## License

This project is intended for preservation and research purposes. All game assets belong to their respective copyright holders.
