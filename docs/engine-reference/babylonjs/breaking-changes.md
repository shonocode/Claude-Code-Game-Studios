# Babylon.js — Breaking Changes Reference

Last verified: 2026-11-16 | Engine: Babylon.js 9.36

This file summarizes the major breaking changes between LLM training data
(~7.x) and the pinned version. For the canonical list, always check
https://doc.babylonjs.com/breaking-changes/.

## 7.x → 8.0 (March 2025)

### Audio Engine: Default Disabled

The legacy audio engine no longer auto-creates with the graphics Engine.

**Before (7.x):** `new Engine(canvas)` automatically constructed an AudioEngine.

**After (8.0+):** AudioEngine must be opted into explicitly:

```typescript
// Old engine (still available, but deprecated):
const engine = new Engine(canvas, true, { audioEngine: true });

// New AudioEngine v2 (recommended):
const audioEngine = await CreateAudioEngineAsync();
```

If your code calls `Engine.audioEngine` without setting `audioEngine: true`, it returns null.

Source: https://forum.babylonjs.com/t/audio-engine-breaking-change-for-babylon-js-8-0/56012

### USDz Import — CORRECTION (2026-09-25)

An earlier version of this file said `SceneLoader` accepts `.usdz` from 8.0 via
the loaders package. **That is not supported by the sources:** the
`@babylonjs/loaders` 9.5.0 npm tarball contains no USD/USDZ loader directory
(only `BVH`, `OBJ`, `SPLAT`, `STL`, `glTF`), the 7.x–8.x CHANGELOG entries are
for the USDZ **exporter** (serializers), and PR #18882 describes the 9.26 OpenUSD
loader as Babylon.js's first USD import. USD import starts at **9.26** — see the
9.5 → 9.28 section below.

### WebXR Depth Sensing

New `WebXRFeatureName.DEPTH_SENSING` feature available in 8.0+. Older code that
ran without depth sensing continues to work; new projects targeting AR with
occlusion should adopt it.

### TypeScript Type Tightening

Several Babylon.js APIs that previously accepted loose types (e.g. `any`)
narrowed in 8.0. Code with `strict: true` and `noUncheckedIndexedAccess: true`
may surface errors that were previously silent. Treat new TS errors as bugs
to fix, not warnings to suppress.

### 8.10.1: Target camera rotation in right-handed scenes (official breaking change)

In right-handed scenes (`scene.useRightHandedSystem = true`), `rotation` /
`rotationQuaternion` of `TargetCamera`-derived cameras (`FreeCamera`,
`ArcRotateCamera`, …) was previously rotated 180° on Y relative to the camera's
view/world matrix. From 8.10.1 it is not, so code that **set** those properties
in right-handed scenes is now wrong. Left-handed scenes are unaffected.

Source: https://doc.babylonjs.com/breaking-changes/ (8.10.1 entry), PR https://github.com/BabylonJS/Babylon.js/pull/16691

## 8.0 → 9.0 (March 2026)

### NodeMaterial v2

NodeMaterial received a v2 redesign with new node types. Existing v1 materials
continue to work, but v2 nodes are not available to v1 graphs and vice versa.

**Migration:** Most v1 materials work unchanged. Materials that used internal
private APIs may break. Re-export through the Node Material Editor (NME) if so.

### Inspector v2

The runtime Inspector (`@babylonjs/inspector`) was rewritten ground-up in 9.0
with a service-oriented React-based architecture.

**Migration:** Public Inspector APIs are mostly compatible. Custom inspector
extensions written against v1 internals must be ported.

### Clustered Lighting (New, opt-in)

A new clustered lighting path is available for scenes with many real-time lights.
**Not enabled by default** — opt-in via scene configuration. Existing forward
lighting code continues to work unchanged.

### Frame Graph (New, opt-in)

A new declarative render pipeline API (`FrameGraph`) lands in 9.0. Existing
post-process pipelines (`DefaultRenderingPipeline`, etc.) continue to work.
FrameGraph is for advanced users wanting fine-grained pipeline control.

### Floating Origin / Large World Rendering (New, opt-in)

New API for keeping the active camera at world origin and offsetting geometry,
solving precision issues at large world coordinates. Opt-in; existing code unchanged.

### OpenPBR (New, opt-in material)

New `OpenPBRMaterial` joins existing `PBRMaterial` and `StandardMaterial`. Use
when you need OpenPBR-spec compliant materials (cross-engine asset interchange).

### Animation Retargeting (New)

Built-in retargeting API for skeletal animations. New feature; existing
animation code unchanged.

### Geospatial Camera (New)

`GeospatialCamera` for orbiting a spherical planet, separate from existing camera classes.

### WebGPU Production-Ready

WebGPU is no longer "experimental" — 9.0 marks production-ready status.
WebGL2 remains the conservative default; WebGPU is opt-in via `WebGPUEngine`.

## 9.5 → 9.28 (May–September 2026 — POST-CUTOFF, HIGH RISK)

Upstream's only **officially documented** breaking change in this span is the
9.15 decorator migration. Everything else below is additive or a behaviour
change flagged in PR notes. Primary source for versions: the CHANGELOG
(https://github.com/BabylonJS/Babylon.js/blob/master/CHANGELOG.md); API names
were checked against the published `@babylonjs/core` / `@babylonjs/loaders` /
`@babylonjs/gui` 9.28.0 `.d.ts` files.

| Version | Subsystem | Change | Details |
|---------|-----------|--------|---------|
| 9.15 | Core / TypeScript | **TC39 Stage 3 decorators (BREAKING, official)** | Only affects code that applies Babylon decorators (`@serialize`, `@serializeAsVector3`, `@expandToProperty`, `@addAccessorsForMaterialProperty`, `@editableInPropertyPage`) to your own classes. See subsection below. |
| 9.15 | Build | UMD bundles now ES2015 | `babylon.max.js` and companion `babylonjs.*.js` UMD bundles are emitted at ES2015 instead of ES5 (irrelevant for this project's ESM-only policy). |
| 9.8 | Core / imports | Pure barrel (`@babylonjs/core/pure`) + `.pure` modules | Additive; default imports unchanged. `.pure` modules run no side effects — you must call `Register*()` functions. Not present in 9.5 (`pure.js` first ships in `@babylonjs/core` 9.8.0). |
| 9.15 → 9.19 | Core / imports | Side-effect registration regressions from the `.pure` split | 9.15 dropped registrations in some paths (WebGPU PCF/PCSS shadows via `ShadowGenerator`, `.babylon` loading leaving no active camera). Fixes: #18671 (9.16.0), `.pure.ts` transitive regressions for SceneLoader / depth-stencil / WGSL depth shaders (9.17.1), "Restore transitive side-effect registrations" #18751 (9.19.0). **Do not pin 9.15.x–9.18.x.** |
| 9.8 | Cameras | `InputMapper` input-mapping system | `ArcRotateCamera` (`ArcRotateCameraMovement`) and `GeospatialCamera` (`GeospatialCameraMovement`) route input through `camera.movement.input` (an `InputMapper`). Legacy flags (`useCtrlForPanning`, `panningMouseButton`, `useAltToZoom`) still work via automatic bridging. PR states no breaking changes. |
| 9.12.1 | Cameras | Framerate-independent `FreeCamera` / `FlyCamera` movement | New `camera.movement` (`TargetCameraMovement`) with configurable `InputMapper`. Default feel preserved at the reference framerate; inertial glide is now framerate-independent. **No opt-out flag**; `camera.inertia = 0` disables inertia. `cameraDirection` / `cameraRotation` still work. |
| 9.14 | Cameras | Inertia glide cutoff decoupled from `camera.speed` | Behaviour change: at non-default `ArcRotateCamera.speed` the inertia tail is no longer truncated early; `TargetCamera` rotation glide now eases out instead of snapping (`_rotationEpsilon` no longer ends rotation glide). At `speed = 1` ArcRotate behaviour is identical. |
| 9.12.1+ | Loaders | FBX loader in `@babylonjs/loaders` | `FBXFileLoader` (plugin name `"fbx"`, extension `.fbx`). First present in the 9.12.1 npm tarball (no CHANGELOG entry found for the initial add); major coverage expansion in 9.26.1 (#18907). |
| 9.26 | Loaders | OpenUSD WebAssembly loader | `USDFileLoader` (plugin name `"usd"`; `.usd`, `.usda`, `.usdc`, `.usdz`). Runs a prebuilt OpenUSD WASM module in a Web Worker; **defaults to downloading worker/WASM from the Babylon.js CDN** (override with `workerUrl` / `glueUrl` / `wasmUrl`). Does not crawl external layers/textures — pass them via `files`. |
| 9.16 | Loaders | `KHR_gaussian_splatting` glTF extension | Additive. |
| 9.27 | Loaders / Serializers | `KHR_interactivity` export | Additive ("ratified KHR_interactivity Phase 1 support" landed in 9.26.2). |
| 9.16.2 | Loaders | glTF 2.0 import auto-registers the SceneLoader plugin | Fix (#18679): importing glTF/2.0 now registers the loader plugin. |
| 9.28 | Animation | Root motion | `RootMotionClip(animationGroup, options?)` and `RootMotionController(characterNode, clips?)` in `@babylonjs/core/Animations/rootMotion`. Additive. |
| 9.7 / 9.17.1 | Physics | `PhysicsCharacterController` additions | `maxStepHeight` (9.7) and `footOffset`; runtime shape changes via `setShapeOptions(options, preserveFootPosition?)` / `shapeOptions` ("Physics Controller shape options", 9.17.1). 9.19.1 fixed inverse-inertia-tensor rows in the controller. |
| 9.19.1 | Physics | Havok floating-origin world regions opt-out | `HavokPluginParameters.disableWorldRegions` (default false) and `floatingOriginWorldRadius` (default 100000) — 3rd arg of `new HavokPlugin(useDelta, havok, parameters)`. |
| 9.8 / 9.11 / 9.26 | Audio (AudioV2) | Distance-only spatial mode, waveform data, iOS ringer opt-out | `spatialPanningEnabled: false` option / `sound.spatial.panningEnabled` (distance attenuation without L/R panning; cones not applied). Analyzer `getByteTimeDomainData()` / `getFloatTimeDomainData()`. `disableIOSRingerSwitchWorkaround` engine option (default false). |
| 9.26 | GUI | `FlexPanel` + em/rem units | New `FlexPanel` control (`flexDirection`, `flexWrap`, `justifyContent`, `alignItems`, `alignContent`, `gap`); `ValueAndUnit` accepts `"2em"` / `"1rem"`. Additive. |
| 9.14 | Engine | `canvasTabIndex` engine option | Additive. |
| 9.14 | Textures | `HtmlTexture` (HTML-in-Canvas, WICG) | `new HtmlTexture(name, element, options)`. Depends on the experimental WICG HTML-in-Canvas proposal (PR ships a polyfill installer). Browser support: unverified. |
| 9.21.1 | Rendering | `FSR1RenderingPipeline` | AMD FSR 1 (EASU + RCAS) upscaling; WebGPU, WebGL2 and Native. `new FSR1RenderingPipeline(name, scene, cameras?)`, `scaleFactor`, `sharpnessStops`, `isSupported`. |
| 9.7–9.26 | Rendering | Misc. additive | Texture repetition breaking for Standard/PBR/OpenPBR (9.7); white balance (`temperature` / `tint`, `whiteBalanceEnabled`) in image processing (9.24); `DitheredTileFadeMaterialPlugin` (9.26.1); `MeshBlendingPostProcess` (9.26.2); IBL shadows on `ShadowOnlyMaterial` (9.26); normalized occlusion-query visibility API (9.27.1). |
| 9.16 → 9.24 | WebXR | WebGPU XR + many features | See `modules/webxr.md` "9.6–9.28 Changes". WebGL2 XR behaviour unchanged. |
| 9.7 / 9.9.1 | Tooling | Smart Assets, Inspector v2 Project Authoring, `.babylonproj` | Editor/Inspector workflow; see `packages/inspector.md`. |

### 9.15: TC39 Stage 3 decorators — migration

Code that merely *uses* Babylon classes is unaffected. If your code subclasses a
Babylon type and applies Babylon decorators:

1. Compile with `experimentalDecorators: false` (the modern TS default — usually
   just remove the flag). Leaving it `true` makes Babylon's decorators receive the
   wrong arguments and **silently fail** (properties not serialized / not shown in editors).
2. Fields decorated with the auto-accessor decorators `@expandToProperty` or
   `@addAccessorsForMaterialProperty` must use the `accessor` keyword:

   ```typescript
   // Before (≤ 9.14)
   @expandToProperty("_reorderLightsInScene")
   public renderPriority: number = 0;

   // After (9.15+)
   @expandToProperty("_reorderLightsInScene")
   public accessor renderPriority: number = 0;
   ```

   `@serialize()` / `@editableInPropertyPage(...)` on plain fields need no change.
3. Target `ES2015` or later (required by `accessor`). This project's `ES2022` target already satisfies it.
4. Replace reads of the internal `_propStore` with `GetEditableProperties(target)`.

Note: the official breaking-changes page files this under **9.15.0**; the
CHANGELOG lists the flip PR (#18647) under the 9.16.0 heading. Treat 9.15–9.16 as
the transition window.

Source: https://doc.babylonjs.com/breaking-changes/ (9.15.0 entry), PRs
https://github.com/BabylonJS/Babylon.js/pull/18631, https://github.com/BabylonJS/Babylon.js/pull/18647

### Sources for this span

- CHANGELOG: https://github.com/BabylonJS/Babylon.js/blob/master/CHANGELOG.md
- Pure barrel: https://github.com/BabylonJS/Babylon.js/pull/18441, https://doc.babylonjs.com/setup/frameworkPackages/es6Support/treeShaking
- 9.15 regression fix: https://github.com/BabylonJS/Babylon.js/pull/18671
- Camera input mapping: https://github.com/BabylonJS/Babylon.js/pull/18379, https://github.com/BabylonJS/Babylon.js/pull/18573, https://github.com/BabylonJS/Babylon.js/pull/18589
- OpenUSD loader: https://github.com/BabylonJS/Babylon.js/pull/18882 — FBX: https://github.com/BabylonJS/Babylon.js/pull/18907
- Root motion: https://github.com/BabylonJS/Babylon.js/pull/18908 — FSR 1: https://github.com/BabylonJS/Babylon.js/pull/18723
- FlexPanel/em/rem: https://github.com/BabylonJS/Babylon.js/pull/18881 — AudioV2 distance-only: https://github.com/BabylonJS/Babylon.js/pull/18462
- WebGPU XR: https://github.com/BabylonJS/Babylon.js/pull/18650

## 9.28 → 9.36 (October–November 2026 — POST-CUTOFF)

The CHANGELOG has **no `[Breaking Change]` entries** for 9.29.0 through 9.36.0.
Everything below is additive or a fix. Versions come from the CHANGELOG
(https://github.com/BabylonJS/Babylon.js/blob/master/CHANGELOG.md); API names
were checked against the published `@babylonjs/core` / `@babylonjs/loaders` /
`@babylonjs/inspector` 9.36.0 `.d.ts` files and `package.json`.

| Version | Subsystem | Change | Details |
|---------|-----------|--------|---------|
| 9.30 | WebGPU | Canvas tone mapping / color space | `WebGPUEngineOptions.canvasToneMapping?: GPUCanvasToneMapping` and `canvasColorSpace?: PredefinedColorSpace` (#18967). Additive. |
| 9.31.1 | Engine | `useReverseDepthBuffer` creation option | `useReverseDepthBuffer?: boolean` on engine options (#19010). Typings: recommended for WebGPU, little gain on WebGL; pair with `stencil: false` to get `depth32float`. |
| 9.31.1 | Shadows | "Support cascaded shadow maps" (#19001) | `CascadedShadowGenerator(mapSize, light, usefulFloatFirst?, camera?, useRedTextureType?)` and `static IsSupported` exist in 9.36.0 typings, but the class predates 9.31 — the PR's exact scope is **unverified** (likely Native). 9.32.1 adds a per-generator cascade uniform buffer for WGSL receivers (#19037). |
| 9.31.1 | Gaussian splatting | Gaussian Point Splatting (WebGPU) | #18962. Additive. |
| 9.33.0 | WebGPU | GPU frame time via timestamp writes | #19066, read after the frame's submit. |
| 9.33.0 | Loaders | `KHR_materials_scatter` import/export | #18810, `@babylonjs/loaders` `glTF/2.0/Extensions/KHR_materials_scatter`. |
| 9.33.1 | Textures | KTX2 encoder (Basis Universal) | `KhronosTextureContainer2Encoder` in `@babylonjs/core/Misc/khronosTextureContainer2Encoder` (#19011). |
| 9.33.3 | Materials | Bent-normal reflection occlusion | `PBRMaterial.bentNormalTexture` (#19069). |
| 9.33.3 | WebGPU | `maxFramesInFlight` | `WebGPUEngine.maxFramesInFlight: number`, default `0` = no limit, opt-in (#19093). |
| 9.33.3 | Loaders | SPZ-compressed glTF Gaussian splats | `KHR_gaussian_splatting_compression_spz` (#19106). |
| 9.31 → 9.35 | Inspector | Inspector Lite | `@babylonjs/inspector/lite` export; needs the optional peer `@babylonjs/lite` (`^1.38.1`). Gained animation, GPU picking, gizmos, stats, physics (9.34) and audio (9.35) panels. Inspector v2 (default export) unchanged. |
| 9.36 | Meshes | `SubMesh.setIndirectDrawBuffer(buffer, byteOffset, renderPassId)` | GPU-written draw arguments per render pass, WebGPU (#19097). |
| 9.36 | Textures | `Texture.replaceImage(source: Texture): boolean` | Take the image of another, already loaded texture (#19099). |
| 9.36 | Textures | `InstallHtmlInCanvasPolyfill` deprecated | See `deprecated-apis.md` "9.28 → 9.36" (#19132). |
| 9.29 | WebXR | Default XR entry errors documented for WebGPU recovery | #18940, flagged Bug Fix. |

Fixes worth knowing when upgrading from 9.28: many WebGPU correctness fixes
(context-loss effect rebuild 9.35, cached draws invalidated on vertex buffer
change 9.33.3, `WebGPUEngine.dispose()` after failed init 9.32.1, geometry
rebuild on buffer restore 9.32.1), FBX import memory / animation fixes (9.32.1,
9.33.3), and glTF Draco normalized override without workers (9.36).

## What Generally Did NOT Change

- ESM scoped imports (`@babylonjs/core/...`) work the same way 7→8→9
- Scene/Mesh/TransformNode core APIs are stable
- glTF/glb loading is unchanged (USDz is additive)
- Observer pattern (`Observable`/`Observer`) is unchanged
- Disposal semantics (`dispose()`) are unchanged
- 9.5 → 9.36: default (non-`.pure`) imports keep running their side effects exactly as before

## Migration Checklist for Code Last Touched on 7.x

1. Audit all `Engine.audioEngine` references — set `audioEngine: true` or migrate to AudioEngine v2
2. Re-run TypeScript build with `strict: true` + `noUncheckedIndexedAccess: true` and address new errors
3. Test custom Inspector extensions against Inspector v2 (rebuild if broken)
4. Re-check NodeMaterial graphs that used internal APIs (export via NME if needed)
5. Verify any `cannon`/`ammo` physics imports — Havok is now the default recommendation
6. Run a full visual smoke test — rendering output is unchanged for default paths but worth confirming

## Migration Checklist for Code Last Touched on 9.5

1. Bump **all** `@babylonjs/*` packages together to 9.36.x (`npm ls @babylonjs/core` shows one version)
2. Grep for Babylon decorators on your own classes (`@serialize`, `@expandToProperty`, `@addAccessorsForMaterialProperty`, `@editableInPropertyPage`) — apply the 9.15 migration above; confirm `experimentalDecorators` is not `true`
3. If adopting `@babylonjs/core/pure`, call the matching `Register*()` helpers (at minimum `RegisterStandardEngineExtensions()`), then smoke-test shadows, `.babylon` loading and loaders
4. Re-check camera feel (inertia, high-refresh displays, non-default `speed`) — Free/Fly/ArcRotate movement changed in 9.12.1 / 9.14
5. If using `@babylonjs/loaders` USD: self-host the USD worker/WASM (`workerUrl`/`glueUrl`/`wasmUrl`) instead of the CDN default
6. Run a full visual + XR smoke test on target devices

## Migration Checklist for Code Last Touched on 9.28

1. Bump **all** `@babylonjs/*` packages together to 9.36.x (no official breaking changes in 9.29–9.36)
2. If you call `InstallHtmlInCanvasPolyfill` / `UninstallHtmlInCanvasPolyfill`, switch to importing an up-to-date polyfill directly before creating the engine (9.36 deprecation)
3. Smoke-test on WebGPU and WebGL2 — the span is dominated by WebGPU fixes, which can change output that relied on the old behaviour
