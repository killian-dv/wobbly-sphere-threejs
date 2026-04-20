# Wobble Shader Material — Three.js Journey

Quick recap of the **Wobble Shader Material** lesson from [Three.js Journey](https://threejs-journey.com/) by Bruno Simon.

## What this project covers

This project shows how to create an animated wobble effect on a mesh by combining custom GLSL deformation with physically based rendering. Instead of replacing Three.js materials entirely, the setup uses `three-custom-shader-material` to inject custom vertex and fragment logic into `MeshPhysicalMaterial`, keeping standard lighting behavior while adding procedural motion and color variation.

- **Procedural vertex deformation** driven by 4D simplex noise (`position + time`) to animate geometry in real time.
- A **two-stage noise approach** (warp noise + wobble noise) to produce richer, less uniform motion.
- **Custom normal reconstruction** in the vertex shader so lighting stays coherent after geometry deformation.
- A **custom depth material** using the same vertex deformation so shadow casting matches the animated shape.
- A fragment shading step that **mixes two artistic colors** from wobble intensity and adjusts roughness for shiny highlights.
- **lil-gui controls** to tune frequencies, strengths, and colors interactively.

## What I built

- Set up `CustomShaderMaterial` with `THREE.MeshPhysicalMaterial` as base to keep PBR features while adding custom shader code.
- Loaded `suzanne.glb` with `GLTFLoader` + `DRACOLoader`, then applied the wobble material to the mesh.
- Loaded an HDRI environment (`urban_alley_01_1k.hdr`) with `RGBELoader` for reflections and realistic scene lighting.
- Added wobble uniforms for motion behavior:
  - `uTime`
  - `uPositionFrequency`, `uTimeFrequency`, `uStrength`
  - `uWarpPositionFrequency`, `uWarpTimeFrequency`, `uWarpStrength`
  - `uColorA`, `uColorB`
- In the vertex shader:
  - computed wobble from simplex noise
  - displaced vertices along normals
  - sampled neighboring positions along tangent/bitangent
  - rebuilt `csm_Normal` from cross products for correct lighting
- In the fragment shader:
  - used wobble intensity to blend between two colors
  - modulated roughness for a shinier look on selected areas
- Created a matching custom depth material (`MeshDepthMaterial` base + same vertex shader) to preserve accurate animated shadows.

## What I learned

### 1) How to extend standard Three.js materials safely

- `three-custom-shader-material` lets me inject custom GLSL while preserving the full PBR pipeline.
- This is often more practical than writing a complete `ShaderMaterial` from scratch.
- I can keep familiar physical controls (`metalness`, `roughness`, `ior`, `thickness`, etc.) and still add custom effects.

### 2) Why 4D noise works well for animation

- Sampling simplex noise with `vec4(position, time)` creates smooth temporal evolution.
- Adding a warp layer before the main noise increases visual complexity.
- Frequency and strength controls make the effect art-directable without changing shader structure.

### 3) Why normals must be recomputed after deformation

- Moving vertices in the vertex shader invalidates original normals.
- Rebuilding normals from local deformed neighbors keeps lighting stable and believable.
- Tangent + bitangent sampling is a useful pattern for procedural surface deformation.

### 4) How to keep shadows consistent with animated geometry

- If only the visible material deforms, shadows become incorrect.
- A custom depth material with the same vertex displacement keeps shadow maps aligned with the final mesh.
- This is essential for convincing real-time shading in deformed objects.

### 5) How shader outputs can drive look development

- The wobble signal is useful beyond displacement: it can also drive color and roughness.
- Mapping noise to material properties creates richer surfaces with minimal extra logic.
- Keeping these parameters exposed in GUI speeds up iteration.

### 6) Practical real-time workflow improvements

- `lil-gui` makes it easy to tune shader parameters live and understand their visual impact.
- HDR environments quickly improve perception of volume and surface quality.
- Orbit controls + live uniforms provide a strong sandbox for learning advanced shading.

## Run the project

```bash
npm install
npm run dev
```

## Credits

Part of the **Three.js Journey** course by Bruno Simon.

