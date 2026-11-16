# /setup-engine — Babylon.js (a TypeScript 3D library on Vite)

> Part of `/setup-engine`. `SKILL.md` names when to read this file; the settings it
> resolved and the rules under its later headings still apply here.

Read when the engine is **Babylon.js** (`/setup-engine babylonjs`, a guided-mode
pick, or a `package.json` that depends on `@babylonjs/core`). Babylon.js is a
versioned engine like Godot, so Sections 3 to 8 all run; this file gives the
Babylon.js row of each, and replaces Section 7.5. Never load
`godot-language-config.md`, and never read `references/web.md`: the Web engine
(`engine.name: Web`) is a different engine with its own scaffold.

**In this file:** B1 Version and probe (Section 3) · B2 CLAUDE.md (Section 4) ·
B3 Technical preferences (Section 5) · B4 `project.yaml` (Section 5.5) ·
B5 Knowledge gap and reference docs (Sections 6 and 7) · B6 The Vite project
(Section 7.5) · B7 Read back (Section 8.5) · B8 Summary (Section 12)

## B1. Version and probe (Section 3)

The version is the `@babylonjs/core` npm version, without the product name
(`9.36`, not `Babylon.js 9.36`). Look up the latest with
`npm view @babylonjs/core version` (or the registry,
`https://registry.npmjs.org/@babylonjs/core`), then compare it with the version
the shipped reference set is curated for (`docs/engine-reference/babylonjs/VERSION.md`,
**Engine Version**). When the two differ, say so and offer: pin the curated
version (the reference matches it), or pin the latest and refresh the reference
(Section 7, then `/setup-engine refresh`). Babylon.js ships a minor every week,
so the latest is usually ahead of the curated set.

There is no editor binary — Babylon.js is an npm package. Probe the toolchain
with `node --version` and `npm --version`, then the resolved package:
`npm ls @babylonjs/core --depth=0`, else the `version` field of
`node_modules/@babylonjs/core/package.json`. Before `npm install` has run (the
usual state at setup time) the package is `NOT DETERMINED`, not absent. There is
no `engine.path` to record: the commands go through `npm`.

## B2. CLAUDE.md Technology Stack (Section 4)

The language is fixed — no question is asked:

```markdown
- **Engine**: Babylon.js [version]
- **Language**: TypeScript (strict + noUncheckedIndexedAccess mandatory)
- **Build System**: Vite
- **Asset Pipeline**: glTF/glb import via @babylonjs/loaders, KTX2 textures for production, .env files for IBL
```

Engine reference import: `@docs/engine-reference/babylonjs/VERSION.md`.

## B3. Technical preferences (Section 5)

**Naming (TypeScript):**
- Classes: PascalCase (e.g., `PlayerController`)
- Functions/methods: camelCase (e.g., `updatePosition`, `takeDamage`)
- Variables: camelCase (e.g., `moveSpeed`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_HEALTH`)
- Files: kebab-case or camelCase (e.g., `player-controller.ts`) — pick one and be consistent project-wide
- Type imports: `import type { ... }` separated from runtime imports for tree-shaking
- Observer callbacks: prefixed `onSomething` (matches Babylon.js `Observable<T>` conventions)
- Async functions: no special prefix (TypeScript types make async-ness visible)

**Testing:** Vitest, plus Playwright for browser/E2E. It must match
`commands.test` (`npm test`, B4) and `.claude/rules/test-standards.md`.

**Engine Specialists:**

```markdown
## Engine Specialists
- **Primary**: babylonjs-specialist
- **Language/Code Specialist**: babylonjs-specialist (TypeScript review — primary covers it; TS is fixed)
- **Shader Specialist**: babylonjs-shader-specialist (NodeMaterial v2, GLSL/WGSL, post-processing, particle shaders)
- **UI Specialist**: babylonjs-gui-specialist (Babylon GUI: ADT, 2D Controls, 3D GUI for XR, Node GUI)
- **Additional Specialists**: babylonjs-webxr-specialist (WebXR sessions, hand tracking, depth sensing, anchors, Quest/Vision Pro)
- **Routing Notes**: Invoke primary for scene graph architecture, asset loading, render loop, Vite/tsconfig setup, and physics integration. Invoke shader specialist for any material/shader/post-process work. Invoke UI specialist for in-scene UI (HTML overlay UI is `ui-programmer` domain). Invoke WebXR specialist for any immersive/VR/AR feature. Babylon.js is TypeScript-only — no language specialist split.

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| Game code (.ts files) | babylonjs-specialist |
| Shader files (.glsl, .wgsl, inline shaders in .ts) | babylonjs-shader-specialist |
| NodeMaterial JSON exports (.nme.json) | babylonjs-shader-specialist |
| In-scene UI code (Babylon GUI, ADT, 3D GUI) | babylonjs-gui-specialist |
| Node GUI exports (.gui.json) | babylonjs-gui-specialist |
| WebXR / immersive feature code | babylonjs-webxr-specialist |
| Build / config files (vite.config.ts, tsconfig.json, package.json) | babylonjs-specialist |
| HTML overlay UI (DOM/CSS) | ui-programmer (engine-agnostic) |
| General architecture review | babylonjs-specialist |
```

## B4. `project.yaml` (Section 5.5)

```yaml
engine:
  name: "BabylonJS"
  version: "<version from B1>"
  language: "TypeScript"
  rendering: "WebGL2"
  physics: "Havok"
```

> **Babylon.js is written `BabylonJS` — no dot, exactly this casing.**
> `engine.name` is an enum (`yaml-helper.sh` rejects anything else), and
> skills derive two paths from it by lowercasing: the reference directory
> `docs/engine-reference/<engine>/` and the primary agent `<engine>-specialist`.
> `BabylonJS` lowercases to `babylonjs`, which is both the directory and the
> agent prefix; `Babylon.js` would produce `babylon.js-specialist`, an agent that
> does not exist. The prose name `Babylon.js` stays correct everywhere else
> (CLAUDE.md, `technical-preferences.md`).

**Rendering and physics have one row, keyed `any`** — do not run Godot's
shape resolution, and ask nothing. **Why `WebGL2` and not `WebGPU`:** Babylon's
default `Engine` class is WebGL2; WebGPU is opt-in through `WebGPUEngine`
(`docs/engine-reference/babylonjs/modules/rendering.md`). If the user has
*already said* they want WebGPU, write `WebGPU (WebGL2 fallback)` instead —
`project-coherence.sh` then checks that `src/` references `WebGPUEngine`.
`Havok` requires the `@babylonjs/havok` package (WASM) in `package.json`, which
the same script checks; note to the user that Havok needs WebAssembly SIMD
(`docs/engine-reference/babylonjs/packages/havok.md`). A 2D Babylon.js project
that uses no physics may write `none`.

**`specialists`:**

| code | shader | ui | additional |
|------|--------|----|------------|
| babylonjs-specialist | babylonjs-shader-specialist | babylonjs-gui-specialist | [babylonjs-webxr-specialist] |

**`naming`:**

| Field | Babylon.js |
|-------|------------|
| `classes` | PascalCase |
| `variables` | camelCase |
| `constants` | SCREAMING_SNAKE |
| `signals` | **camelCase `on<Event>Observable`** |
| `files` | kebab-case |
| `scenes` | kebab-case |

`signals` is Babylon's `Observable<T>` convention (`onPointerObservable`,
`onBeforeRenderObservable`) — Babylon has no signals. `files` is `kebab-case`
because B3 says "kebab-case or camelCase — pick one"; if the user picks
camelCase, write that instead. A Babylon scene is TypeScript code (or a loaded
`.glb`), not an editor scene file, so `scenes` takes the `files` value.

**`commands`** — write as-is; there is nothing to ask. The web build has a
single target (Vite emits `dist/`), so `build` carries no platform placeholder.
None of the four values contains a quote, so all take plain double-quoted
scalars. Each `npm` command names a `package.json` script (`build`, `test`,
`dev`), which B6 and `/test-setup` create — `project-coherence.sh` reports any
that are missing. `run` starts the Vite dev server, not a window:
`.claude/docs/run-and-observe.md` covers how the run-and-observe step opens it
in a browser at 1280x720 and captures it. `smoke` is a whole-project type-check
— it finishes in seconds and fails on the errors that would stop the page
loading. These are also the BabylonJS row of `.claude/engines/engines.tsv`.

```yaml
commands:
  build: "npm run build"
  test: "npm test"
  run: "npm run dev"
  smoke: "npx tsc --noEmit"
```

Omit `engine.path`: Babylon.js has no editor.

## B5. Knowledge gap and reference docs (Sections 6 and 7)

**Knowledge gap:** training data likely covers Babylon.js up to ~8.x / early 9.x
(9.0 shipped March 2026; later 9.x minors are post-cutoff, and a minor ships
every week). Any pin past 9.5 is HIGH RISK.

**Reference docs:** the template ships `docs/engine-reference/babylonjs/`
already populated. Follow Section 7's "does a reference set already exist"
branches: same version → nothing to do; the directory pins an older version →
update in place from the official CHANGELOG
(`https://github.com/BabylonJS/Babylon.js/blob/master/CHANGELOG.md`) and the
breaking-changes page, never from recall. **Still add the `Installed at pin
time` row from B1 if the file has none** — `project-coherence.sh` reports its
absence as `[DIFFERS]`.

## B6. The Vite project (Section 7.5)

Replaces Section 7.5 for Babylon.js. **Never overwrite an existing
`package.json`, `tsconfig.json`, `vite.config.ts` or `index.html`.** If
`package.json` is already there, say so, confirm it has `dev`, `build` and
`test`-able scripts matching B4's `commands` (add missing scripts only with
approval), and skip to Section 8.

A Vite + TypeScript project is a handful of plain-text files, so unlike Unity
and Unreal it *can* be written here — but **no version number is typed from
recall.** The files carry no `dependencies`; `npm install` writes them, resolving
the pinned version itself. Show the proposed files, then ask: *"May I create
`package.json`, `tsconfig.json`, `vite.config.ts`, `index.html` and
`src/main.ts`, then run `npm install`?"* On approval, write:

`package.json` (`name` from the brief's working title, else the repo directory
name, lowercased and hyphenated):

```json
{
  "name": "<project-name>",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "preview": "vite preview"
  }
}
```

`tsconfig.json` — the flags are the ones
`docs/engine-reference/babylonjs/current-best-practices.md` §Project Setup
mandates (`strict`, `noUncheckedIndexedAccess`) or recommends:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "noEmit": true
  },
  "include": ["src", "tests"]
}
```

`vite.config.ts`:

```typescript
import { defineConfig } from "vite";

export default defineConfig({
  server: { port: 5173, strictPort: true },
});
```

`index.html` — one full-window canvas and the module entry:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><project name></title>
    <style>html, body { margin: 0; height: 100%; overflow: hidden; } #renderCanvas { width: 100%; height: 100%; display: block; touch-action: none; }</style>
  </head>
  <body>
    <canvas id="renderCanvas"></canvas>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

`src/main.ts` — the Engine/Scene pattern from `current-best-practices.md`
§Scene Construction, nothing more. On `WebGPU (WebGL2 fallback)` use that file's
`WebGPUEngine.IsSupportedAsync` branch (`modules/rendering.md`) instead of a bare
`Engine`:

```typescript
import { Engine } from "@babylonjs/core/Engines/engine";
import { Scene } from "@babylonjs/core/scene";
import { FreeCamera } from "@babylonjs/core/Cameras/freeCamera";
import { HemisphericLight } from "@babylonjs/core/Lights/hemisphericLight";
import { Vector3 } from "@babylonjs/core/Maths/math.vector";

const canvas = document.getElementById("renderCanvas") as HTMLCanvasElement;
const engine = new Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true });
const scene = new Scene(engine);
const camera = new FreeCamera("camera", new Vector3(0, 5, -10), scene);
camera.setTarget(Vector3.Zero());
camera.attachControl(canvas, true);
new HemisphericLight("light", new Vector3(0, 1, 0), scene);

engine.runRenderLoop(() => scene.render());
window.addEventListener("resize", () => engine.resize());
```

`preserveDrawingBuffer: true` is kept deliberately: it is what lets the
run-and-observe step read the canvas back (`.claude/docs/run-and-observe.md`).

Then install, letting npm resolve and record every version:

```bash
npm install --save-dev vite typescript
npm install @babylonjs/core@~<engine.version>.0
npm install @babylonjs/havok          # only when engine.physics is Havok
```

`~<version>.0` keeps installs on the pinned minor (patches only), matching the
reference set's minor-version policy.

**Leave the scene's content, a scene-select URL parameter, and Havok
initialisation to `/dev-story`.** How Havok's `.wasm` is served under Vite is
not covered by `docs/engine-reference/babylonjs/packages/havok.md`; record it as
`NOT SOURCEABLE` rather than adding bundler config from recall.

**Then verify, and report what the verification actually was.** If `npm` was
found in B1, run `npm run build` and report the result. If `npm` is not
available, or `npm install` failed, write **`Babylon.js project NOT VERIFIED —
<reason>`**.

No `test` script is written here — `/test-setup` adds it with Vitest. Until
then Section 8.5 reports `commands.test` (`npm test`) as `[DIFFERS]`; that is
expected, and the fix is `/test-setup`, not a placeholder script.

## B7. Read back (Section 8.5)

`project-coherence.sh` compares `project.yaml` against
`docs/engine-reference/babylonjs/VERSION.md`, against `package.json` and `src/`
(the `@babylonjs/core` and `@babylonjs/havok` dependencies, `WebGPUEngine` when
rendering is WebGPU), against the `@babylonjs/core` npm resolved into
`node_modules/`, and checks that every `npm` command's script is defined in
`package.json`. A `[NOT CHECKED]` for no `node_modules/` means `npm install` has
not run; it is not a pass.

## B8. Summary (Section 12)

`Language: TypeScript`. Section 9 (Version Awareness in the agent files)
applies to the four `babylonjs-*` agents.
