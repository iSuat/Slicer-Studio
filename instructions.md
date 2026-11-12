# OpenSlicer Studio — AI-Assisted Development Instructions (`instructions.md`)

**English Version** | [Türkçe Sürüm](instructions.tr.md)

This document is prepared to prevent context loss, minimize credit/token consumption, and maintain existing architectural rules when developing the **OpenSlicer Studio** project with AI assistants in future sessions.

The `./Backup` directory is exclusively for code backups; do not read or scan this folder.

---

## 1. Project File Structure and Rules
* **Main Application File:** [index.html](file:///d:/Projeler/Open%20Slicer/index.html) (`6,843` lines)
  * The project follows a **Single-File Application** (single HTML + CSS + JS) architecture. No external bundlers (Webpack, Vite) are used; runs directly in modern browsers and is fully compatible with GitHub Pages.
  * Dependencies are loaded via CDN:
    * `Three.js r128` (`three.min.js`, `OrbitControls.js`, `TransformControls.js`, `STLLoader.js`, `OBJLoader.js`)
    * `JSZip 3.10.1` (for `.3mf` parsing and `.zip` batch export)
    * `ClipperLib 6.4.2` (`clipper.js` — for 2D Boolean union, difference, and parallel offset operations)
* **License:**
  * The project is officially licensed under the **GNU General Public License v3.0 (GPLv3)** ([LICENSE](file:///d:/Projeler/Open%20Slicer/LICENSE)).
* **Strict Versioning Rule:**
  * The version badge displayed in the top bar is set to `beta 1.5` (`.app-version-badge`).
  * **Important:** Version numbers must NOT be incremented automatically during routine tasks. The version is only updated **when the user explicitly requests to publish a new release**, setting it to the user's chosen version string.
* **Support & Donation Buttons:**
  * A `Buy Me a Coffee` button (`https://www.buymeacoffee.com/iSuat`) is placed in the top bar between `☀️ Light Mode` and `? Help`.
  * A GitHub Sponsors button will be placed alongside once approval is granted.
* **CAD Integration & Add-In Files:**
  * [OpenSlicerBridge.py](file:///d:/Projeler/Open%20Slicer/Fusion360_Plugin/OpenSlicerBridge/OpenSlicerBridge.py): Autodesk Fusion 360 Add-in script (exports STL to selected directory with dynamic `[Project]-[Body].stl` naming on 1-click or save).
  * [OpenSlicerBridge.manifest](file:///d:/Projeler/Open%20Slicer/Fusion360_Plugin/OpenSlicerBridge/OpenSlicerBridge.manifest): Fusion 360 add-in manifest.
  * [Install_OpenSlicerBridge.bat](file:///d:/Projeler/Open%20Slicer/Fusion360_Plugin/Install_OpenSlicerBridge.bat): 1-click installer copying add-in to `%APPDATA%\Autodesk\Autodesk Fusion 360\API\AddIns\`.
* **Backup Standard (Mandatory Rule):**
  * Before performing major edits to the main file (`index.html`), always copy the current working version as `SlicerStudio-backup-vX.html` to `./Backup` (Currently, the latest is [SlicerStudio-backup-v14.html](file:///d:/Projeler/Open%20Slicer/Backup/SlicerStudio-backup-v14.html); the next backup should be `Backup/SlicerStudio-backup-v15.html`).
  * **Rule:** Unless explicitly instructed otherwise by the user, always take a backup before any major modification.
* **Syntax Verification (Node.js):**
  * After any code modification, always verify the embedded JavaScript syntax before browser testing:
    ```powershell
    node -e "const fs = require('fs'); const html = fs.readFileSync('index.html', 'utf8'); const scripts = [...html.matchAll(/<script\b[^>]*>([\s\S]*?)<\/script>/gi)]; const mainJs = scripts[scripts.length - 1][1]; new Function(mainJs); console.log('JS Syntax OK! Lines:', html.split('\n').length);"
    ```

---

## 2. Section Map (`index.html`)
When searching inside the file, target these specific section headers instead of reading the entire codebase:

1. **CSS Styles (`Lines 6–510`):**
   * Dark / Light theme variables (`:root`, `body.light-theme`), Left Accordion Sidebar (`#sidebar`), Center 3D Scene (`#viewport3d`), Right Panel (`#viewport2d`), Fullscreen Sheet Lightbox (`#sheetLightbox`), Bottom-Right Assembly Step HUD (`#assemblyStepHud`), `.app-version-badge`, and `.bmc-link`.
2. **HTML Layout (`Lines 515–1160`):**
   * Top Header (`#topbar`): Brand logo, `.app-version-badge` (`beta 1.5`), Undo/Redo buttons, Grid Toggle, Light/Dark Toggle, Buy Me a Coffee Link (`.bmc-link`), and Help button.
   * Left Sidebar Boxes:
     * `Box 1`: Import, Presets & Universal Live Watch (`#presetSelect`, `#fileInput`, `#btnLiveWatchFile`, `#lblLiveWatchStatus`)
     * `Box 2`: Manufacturing Settings (`#unitSelect`, `#sheetW`, `#sheetH`, `#matThick`, `#skipCount`, `#sheetMargin`, `#slotOffset`, `#chkToolDia`, `#toolDia`)
     * `Box 3`: Construction Technique (`#modeSelect` [`stacked` / `waffle` / `radial`], `#waffleBox`, `#radialBox` [`#radialCount`, `#radialRings`, `#radialCoreRadius`, `#radialCenterX`, `#radialCenterY`])
     * `Box 4`: Modify Form / Hollow (`#chkHollow`, `#hollowWall` — *hidden in Waffle & Radial modes*)
     * `Box 5`: Dowels / Alignment Pins (`#chkDowels`, `#dowelShape`, `#dowelDia`, `#btnAutoDowel` — *hidden in Waffle & Radial modes*)
     * `Box 6`: Slice Direction (`#sliceAngX`, `#sliceAngY`, `#btnToggleSliceGimbal`)
     * `Box 7`: Assembly Steps & Preview (`#assemblySlider`, `#btnStepPrev`, `#btnStepNext`, `#btnStepPlay`, `#chkShowExtrude`)
     * `Box 8`: Export Plans (`#layoutSelect`, `#exportFormatSelect`, `#chkSvgCut`, `#chkSvgGuide`, `#chkSvgLabel`, `#labelFontSize`, `#btnExportZip`, `#btnExportSingleMaster`)
   * Center Viewport (`#viewport3d`): Top Toolbar (`Move/Rotate/Scale/Lay-Flat/Measure`), ViewCube Canvas (`#viewCubeCanvas`), Nav Buttons (`#navBtnZoom`, `#navBtnPan`, `#navBtnFit`).
   * Right Panel (`#viewport2d`): Model Issues (`#issuesCard`) and Cut Layout (`#cutLayoutContainer`).
3. **JS Sections 1–2 (`Lines 1160–1580`): UI Settings, Presets & 3D Scene / Large-Scale CNC:**
   * `updateDynamicGridAndFloorDimensions()`: Dynamically scales grid cells (50/100/200/500mm), camera `near`/`far` planes (`near = sceneSpan / 2800`, `far = sceneSpan * 14`), and floor dimension labels for 5m–10m+ CNC models.
   * Magnifying glass zoom damping (`orbit.zoomSpeed = 0.18`).
4. **JS Sections 3–4 (`Lines 1585–2160`): OrcaSlicer Protractor Gimbal & Flicker-Free ViewCube:**
   * `rebuildCustomOrcaSliceGimbal()`: Outer tick ring remains static; cursor inside snaps to 5°, outside provides 0.1° free rotation.
   * `drawViewCube()`: 6 faces ($[-0.505, 0.505]$), 12 chamfer edges, and 8 corners ($[0.51, 0.81]$) with disjoint bounding coordinates and `polygonOffset`, completely resolving gray-tone Z-fighting/flickering.
5. **JS Sections 5–7 (`Lines 2165–2730`): Dowels, Model Loading & OrcaSlicer Lay-on-Face:**
   * `buildLayFlatCandidatePlates()` & `layMeshFlatOnNormal()`: 3D Convex Hull (QuickHull) candidate face detection and flat alignment.
   * Model Inspection Eye (`👁️`): Toggles slice visibility off to inspect raw model geometry as an opaque mesh (`ghostMesh`).
6. **JS Section 8 (`Lines 2735–2825`): Universal Live File Watcher (File System Access API):**
   * Uses `window.showOpenFilePicker` to poll on-disk STL/OBJ file modifications every 850ms, reloading the mesh while preserving camera position, transforms, slice mode, and manufacturing parameters.
7. **JS Sections 9–10 (`Lines 2830–4780`): ClipperLib Boolean, Slice Engine (`runSlicer`), Waffle & Radial Slices:**
   * `sliceMeshAtZ(geom, val, axis)`: Slices along 'z', 'x', 'y', and polar 'radial' (`{ type: 'radial', cx, cy, cos, sin }`).
   * **Radial Slices (`RS`):** Vertical radial fins (`R-1..R-n`) bounded by Clipper half-plane $[r_{\text{core}}, R_{\text{max}}]$ and horizontal rings (`Z-1..Z-m`). `getAutoRadialCoreRadius` dynamically prevents center fin collisions. Features angle-invariant halving joints, Dog-Bone/T-Bone corner relief, and Notch Factor chamfers.
8. **JS Sections 11–13 (`Lines 4785–5765`): Dowels, Issues, 2D Nesting, DXF/SVG/EPS Export & Lightbox:**
   * `updateSliceColors()`: Corrected visibility condition (`grp.visible = !excludedSliceIds.has(id)`) ensuring unhidden parts immediately reappear in 3D.
   * Multi-layer AutoCAD R12 (`AC1009`) pure DXF (`POLYLINE` + `VERTEX`), proportional Single-Line CNC Font (`SINGLE_LINE_FONT`).
   * Fixed real-world mm font sizes with part-specific auto-fit clamping.
9. **JS Sections 14–16 (`Lines 5770–6843`): Assembly Steps, 3D Interaction & Event Loop:**
   * Continuous slider animation, instant magenta collision highlights, and radial inward sliding animation along $(\cos\theta_i, \sin\theta_i, 0)$ for radial fins.

---

## 3. Coordinate Systems and Geometric Rules
1. **3D Scene Coordinates (Z-Up):**
   * `X`: Right/Left, `Y`: Forward/Back (Depth), `Z`: Up (Height). Build plate lies on $Z = 0$.
2. **Slicing Planes and 2D Projection (`sliceMeshAtZ(geom, val, axis)`):**
   * **`Z` Axis (Stacked Slices):** Cut plane $Z = z_0$. 2D coords: $(x = X,\; y = Y)$.
   * **`X` Axis (Waffle X Ribs):** Cut plane $X = x_0$. 2D coords: $(x = Y,\; y = Z)$.
   * **`Y` Axis (Waffle Y Ribs):** Cut plane $Y = y_0$. 2D coords: $(x = X,\; y = Z)$.
   * **`Radial` Axis (Radial Fins):** Normal $(\sin\theta_i, -\cos\theta_i, 0)$. 2D coords: $(x = \text{radial\_dist},\; y = Z)$.
3. **Joint & Slot Direction Rules:**
   * **Waffle:** `Y` ribs are base lattice (top-down slots); `X` ribs slide down from above (bottom-up slots).
   * **Radial Slices:** Circular rings (`Z-1..Z-m`) receive radial slots opening from outer edge toward core; radial fins (`R-1..R-n`) receive horizontal slots opening from inner core outward.

---

## 4. Completed Features Summary (v1.5 Beta)

1. **`P2+` Radial Slices (`RS` - Polar 3D Slicing):**
   * Added `Radial Slices` mode.
   * `R-1..R-n` vertical radial fins + `Z-1..Z-m` horizontal rings with interlocking slots.
   * `getAutoRadialCoreRadius` prevents center fin collisions trigonometrically.
2. **Large-Scale CNC (5m–10m+ / 5000–10000mm+) Support:**
   * Proportional dynamic grid (50–500mm), adaptive camera near/far clipping, and scaled floor dimension lines.
3. **ViewCube Flicker (Z-Fighting) Fix:**
   * Fully partitioned geometric coordinate boundaries between faces, edges, and corners.
4. **Navigation Magnifier Zoom Damping:**
   * Smooth zoom rate clamped to `0.18` when navigation zoom button is toggled.
5. **Cut Layout Unhide Visibility Bug Fix:**
   * Restoring hidden parts from the Cut Layout immediately displays them in the 3D scene.
6. **Universal Live File Watcher:**
   * Zero-install local disk polling (850ms) via File System Access API for external CAD model exports.
7. **Autodesk Fusion 360 Add-In (`OpenSlicerBridge`):**
   * 1-click automated STL exporter with dynamic `[Project]-[Body].stl` naming.
8. **Top Bar `beta 1.5` Badge & Buy Me a Coffee Support:**
   * Placed version badge and Buy Me a Coffee button into the header bar.
   * Officially added GNU GPLv3 license.
9. **Label Size Fix & Box 8 Font Size Control:**
   * Solved huge sheet-proportional text bug; switched to real manufacturing mm font sizes with auto-fit clamping and radial slot alignment.
10. **GitHub Pages & index.html Migration:**
    * Renamed main file to `index.html` for direct web hosting and GitHub Pages compatibility.

---

## 5. Upcoming Roadmap

### ✅ Phase 1: Rapid Fabrication Fixes, Desktop UI & Undo/Redo (Completed)
1. **Desktop App Styling, App Icon & Status Bar:** `slicer-studio.png` icon, 50px top bar, GitHub link, and live bottom status bar (`#statusBar`).
2. **45-Step Undo / Redo Engine (`Ctrl+Z` / `Ctrl+Y`):** Full history across transforms, parameters, dowels, and part exclusions.
3. **Z-Cut (`Sink Below Bed`):** Sinking models below $Z = 0$ with watertight flat base truncation (reset by `Drop to Bed` and `Lay Flat`).
4. **Radial: Lock Pattern to Bed Center (`🔒 Lock Pattern to Bed Center`):** Anchors radial slicing axis to bed origin `(0, 0)`.
5. **Blind Dowel Depth Control (`Top Cap Layers` / `Bottom Cap Layers`):** User-controlled undrilled cap layer counts at top and bottom ends.

### ✅ Phase 2: Lamp, Lighting, Square/Rect Dowel (`R-Bone`), External i18n & Bugfix Suite (Completed)
1. **Square / Rectangular Dowels (`Square / Rect Dowel`):**
   * `Shape` selector placed above dimension inputs in Box 5.
   * Shows `Diameter` when `Round` is selected; hides `Diameter` and reveals **`Width (X)`** and **`Depth (Y)`** when `Square / Rect` is selected to support both square and rectangular battens/profiles.
2. **`R-Bone` (4-Way Rotational Pinwheel) Corner Relief & `Relief Type` Relocation:**
   * Moved `Relief Type` (`#waffleReliefType`) into **2. Manufacturing Settings (`#box2`)** (appears below Advanced when open, or below `Skip Layers` when closed; options: `Square (None)`, `T-Bone`, `Dog Bone`).
   * **User Note:** Tested `R-Bone` on normal interlocking slots and determined it is impractical for general slotting; removed from `#waffleReliefType`. A more complex relief strategy for normal slots will be designed later.
   * `R-Bone` mode is automatically applied **only** to **square/rectangular dowel holes and dowel support washer holes** IF Relief Type is enabled (`T-Bone` or `Dog Bone`). If Relief Type is `Square (None)`, dowels remain sharp/square.
   * Prominent UI warning in Box 5 and Model Issues if cutter diameter exceeds the narrow dowel dimension (`toolDia >= min(dowelW, dowelD)`).
3. **Stacked: Dowel Support Washers (Spacers in 3D Viewport + 2D Sheets):**
   * Automatically generates drilled spacer washers (`W-1, W-2...`) in both the **3D Viewport** (at their exact gap Z positions along each dowel) and **2D Cut Layout** when `Skip Layers > 0` and `Dowels` are active.
4. **Stacked: Symmetric Center-Out Slicing when Tilted (`Tilt X = 90°` or Piece #1 Off-Bed):**
   * When Slice Direction is tilted (`Tilt X = 90°`, etc.) or Piece #1 does not sit flat on the bed, slicing steps symmetrically outward from the center of the slicing axis (`zMid`) toward both ends so symmetric models (like armchairs/sofas) begin with identical end pieces.
5. **Radial: Batch Inward Offset (`Horiz. Ring Offset`) & Per-Part Center Holes (`Center Hole Ø`) in Cut Layout:**
   * Inward offset (`offset(-inwardAmount)`) for horizontal rings (`Z-1..Z-m`) in radial mode for vertical-fin lamp fabrication.
   * Moved Center Hole cutting to the right-hand **Cut Layout (`#partActionBar`)** panel so each horizontal part (`Z-1`, `Z-2`...) can have its own custom hole diameter (`partCenterHoles[id]`), with active part pills.
6. **Radial Slice Selection Bugfix (`R-*` and `Z-*`):**
   * Fixed hidden dowel work planes intercepting 3D clicks in non-stacked modes, relaxed 3D click/drag thresholds, enabled `THREE.DoubleSide` on slice meshes, and used `fill-rule="evenodd"` compound SVG paths so ring holes never block clicks on nested parts.
7. **External Language Files (`locales/en.js`, `locales/tr.js`), Single-Language Help, Ghost Lines Fix & Box 8 `Guides`:**
   * External locale files loaded from `locales/`; single-language Help modal dynamically rendered in the active language.
   * Removed misaligned `EdgesGeometry` wireframe from `ghostMesh` and disabled `Guides (Blue)` in Box 8.

### 📐 Phase 3: 3D & Vector Export Optimizations (In Progress)
1. **✅ 3D Sliced Assembly Export (`.3mf` / `.stl`):**
   * Multi-body 3D manufacturing formats added to Box 8 (`📦 3D (.3MF)` and `⬇ 3D (.STL)`) to export all sliced parts into 3D printer slicers (Bambu Studio, OrcaSlicer, PrusaSlicer, Cura).
   * `.3mf`: Multi-body ZIP archive with each slice (`Z-1`, `X-1`, `R-1`, `W-1`...) as an individual named `<object>`. In Bambu/OrcaSlicer, users can click "Split to Objects" and "Auto-Arrange" to flat-pack every piece onto build plates in 1 click.
   * `.stl`: Assembled world-coordinate binary STL representing the entire sliced physical model.
2. **Vector Arc / Bézier Fitting (Optional):**
   * Converts segmented polyline facets in DXF and SVG exports into smooth `ARC`/`BULGE` and Bézier curves.

### 🛠️ Phase 4: Viewport Gizmos, High-Poly Performance, Hover Tooltips & Smart Geometry (In Progress)
1. **✅ Move Gizmo Cone/Line Synchronization & Back-Left Bed Corner Triad:**
   * Permanently eliminated Three.js `TransformControls` line/cone detachment bug across all camera angles by locking absolute scales (`Math.abs`) and visibility flags for all axis line and cone meshes.
   * Relocated 3D XYZ bed corner triad to the **back-left** corner (`minX`, `maxY`) of the model footprint with the green Y arrow pointing forward (`(0, -1, 0)`) into the build plate.
2. **✅ Model Centering (`🎯 Center (C)`), Prominent Center Axes, Origin Dot & Axis-Locked Subgrid:**
   * Added `🎯 Center (C)` toolbar button and `C` hotkey to center the model on the bed `(0, 0)`.
   * Highlighted center X=0 and Y=0 grid lines with a dedicated CAD origin marker ring and dot at `(0, 0)`.
   * Subgrid is locked to the bed origin `(0, 0)`; expands cell-by-cell in full main grid increments without partial cell cutoffs, with subgrid line spacing set to exactly half of the visible main grid cell (`mainCellSize / 2`).
3. **✅ Enlarged 3D Navigation Cube (Autodesk ViewCube):**
   * Enlarge navigation ViewCube from 112px to 140px (280×280 HiDPI canvas) for comfortable face, edge, and corner clicking.
4. **✅ Panel Interaction Auto-Select Reset & Lay Flat Cleanup:**
   * Clicking on any input, select, or button in the left sidebar or right panel immediately resets the active 3D tool back to `select` mode.
   * Instant cleanup of candidate plates and helper meshes on tool switch or new model import; 6mm smooth fillet corners applied to candidate plates.
5. **✅ Gizmo & Axis Test Environment (`index2.html`):**
   * Independent test sandbox [index2.html](file:///d:/Projeler/Open%20Slicer/index2.html) featuring directional markers (Front, Back, Left, Right, Up) and an asymmetric directional 3D test body to isolate and evaluate `TransformControls` axis flipping across all camera angles.
6. **Lay Flat Outer Tangent Planes (Red Line / Convex Envelope - Backlogged):**
   * Detecting outer tangent contact planes bridging the peaks of curved/stepped meshes (red line) instead of inner facets (blue lines).
7. **Center Square Drop-Zone on Launch (Fast Light Load - Backlogged):**
   * Lightweight startup view with a prominent square drag-and-drop model target in the center of the viewport, expanding on drag and transitioning to full studio layout once loaded.
8. **Hide Box 3 Parameters Until Technique Selected (Backlogged):**
   * Keep Box 3 sub-controls clean/hidden on startup until a construction technique is actively chosen.
9. **ViewCube Themes (SVG / PNG Skins - Backlogged):**
   * Support for seasonal and custom theme skins (Holiday, Dark Industrial, Classic CAD) on ViewCube faces.
10. **Contextual Sidebar Hover Tooltip Popover:**
    * Displays an explanatory tooltip card anchored to the right edge of the left sidebar.
11. **High-Polygon / Large Model Performance Optimization:**
    * Spatial Z/X/Y triangle bucketing / BVH indexing.
12. **Intelligent Dowel Placement (Medial Axis / Wall Mid-Line):**
    * Algorithmic dowel distribution along the medial axis.
13. **Hardware Negative Boolean Library:**
    * Built-in 3D negative boolean templates.
14. **Deduplicate Identical Parts in Cut Layout (Backlogged):**
    * Detect identical slices and place only 1 copy on the cutting sheet with a quantity multiplier (`×N`).

### ⏳ Phase 5: Future Architecture (On Hold)
1. **Folded Panels / Sheet & Cardboard Unfolding (`FP`):**
   * 2D sheet metal and cardboard unrolling with fold lines, bending tabs, and material thickness allowances (Pepakura-style parametric fabrication). Primary focus for future advanced fabrication.
2. **Full-Scope Project Save / Load (`.3mf` / `.json`):** Preserving model, slicing parameters, and slice visibility states.
3. **Advanced True-Polygon Nesting & Web Worker:** True-shape packing with background thread execution.
4. **Curve Slices (`CS`):** Slicing along arbitrary user-defined guide curves.

---

## 6. Communication & Operational Principles
* Always communicate in **Turkish** unless requested otherwise.
* Use clickable markdown links (`[file.ext](file:///...)`) for files and code references.
* When the user requests a discussion before coding, refrain from modifying files until the design is approved.
* Only increment the application version when explicitly requested by the user.
* **Backup Rule:** Unless explicitly instructed otherwise by the user, always take a backup into `./Backup` before performing major modifications to `index.html`.
