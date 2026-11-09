# Cable Pusher

**Current version: v1.4 “Fall”** · [Open the app](https://claude.ai/artifact/Jru6YZ9enoc7HhQRfUAoAm)

Cable Pusher is an interactive 2D analyser for hanging cable structures: chains, suspension and stayed cables, guyed masts, cable trusses, nets and tensegrities. You draw cables and struts, anchor them, and load them, and the app shows the hanging shape, tensions, sag and reactions.

It is built from **Frame Pusher** (a 2D frame finite-element app) and **Arch Pusher** (funicular arches), and it uses their interface, styling and annotation tools. The whole app is a single self-contained HTML page that runs in the browser, with no install and no server.

> Results are for exploration and teaching. They are not a substitute for a checked design calculation.

---

## Quick start

1. Pick a structure from **Load example…** or **Gallery**, or press **New** to start from an empty canvas.
2. Draw with the tool bar on the left: **C** cable, **B** strut, **N** node, **1–4** supports.
3. Load the structure with **P** (point load: press and drag), **U** (deck load per plan metre) or **W** (cable self-weight).
4. Read the results on the canvas, in the info panel and in the **Inspector** (**I**).
5. Save the model with **Ctrl+S** (a `.json` file), or make A4 calculation sheets with **⎙ Report**.

---

## Drawn state, Live / Paused, Solve, Step, Play, Reset and Bake (v1.3 and v1.4)

Every node now stores two positions:

| Position | Meaning |
|---|---|
| **Drawn** (`x0, y0`) | Where you drew, moved or typed the node. This is the *initial state*. |
| **Current** (`x, y`) | Where the node is now. After a solve, this is the equilibrium position. |

The drawn positions are only the starting point of the Newton–Raphson iteration. The unstressed lengths L₀ decide the answer. Some structures have more than one equilibrium: a net that can snap through, a tensegrity, or a strut chain that can fold either way. For those, the starting point decides which equilibrium the solver finds.

### The solver bar

The solver bar sits under the canvas toolbar.

| Control | Key | What it does |
|---|---|---|
| **● Live** | S | Works as in earlier versions: the structure is solved again after every change. |
| **❚❚ Paused** | S | Nodes stay where you draw, drag or type them. Each cable is drawn with its exact catenary between its two nodes and coloured by tension. Orange **ΔF** arrows show the out-of-balance force at each free node, and the largest one is labelled. The app remembers your choice. |
| **▶ Solve** | Enter | Iterates to equilibrium from the current state (Paused only). |
| **⏭ Step** | . | Runs **one** Newton–Raphson iteration (Paused only), so you can watch the structure fall from its drawn shape into equilibrium. The Levenberg shift and the warm starts carry over from one step to the next. |
| **⏵ Play** *(v1.4)* | Space | Plays the solver as an animation: the iterations are solved first, then played back as one smooth movement, so the cables fall into place. From a solved state it replays from the drawn state, and from Live it returns to Live at the end. Speed ½×–4×; a click, an edit or Space stops it. |
| **⟲ Reset** | R | Puts every node back at its drawn position and forgets the warm starts. In Live mode it then solves again from there. |
| **⤓ Bake → Bake shape** | — | The solved positions become the drawn state, and L₀ is kept. The state does not change, and solving again gives the same answer. |
| **⤓ Bake → Bake as unstressed** | — | The solved positions become the drawn state, and every L₀ is set to its stretched length L₀ + ∫T/EA ds. The shape is then stress-free, which is the form-finding step of a hanging model. Under weight or load it moves again when solved. |

### How Play animates the iterations (v1.4)

1. **Solve first, then play.** The iterations are run one at a time, exactly as Step does them, and each one is stored as a frame: node positions, every member's curve and tensions, and the out-of-balance forces.
2. **Interpolate along a spline.** Playback moves through the frames by the *distance moved*, not by the iteration count. Between frames it follows a Hermite spline whose tangents come from the neighbouring frames (chord-length Catmull–Rom). The motion therefore stays smooth through each iteration, with no stop-start. Tension colours and ΔF arrows are blended linearly.
3. **Ease in, ease out.** An ease-in / ease-out in time makes the structure gather pace as it drops and settle gently. Newton's first big step takes most of the time, and the tiny converging steps take almost none.
4. **Finish exactly.** At the end the last iteration is solved exactly, so the final state is the true equilibrium.

The iterations themselves are true solver states. The frames between them are only for the eye. A new verification check confirms that the spline passes exactly through every iteration.

The **Drawn state** tick box in the bottom bar shows the drawn state as a dotted ghost under the current shape.

The bar also shows the solver state: *Live · in equilibrium*, *Paused · drawn state · ΔF …*, *Paused · step n · ΔF …* or *Paused · in equilibrium after n steps*.

### Suggested workflows

- **Watch the cables fall:** draw a chain straight between two pins with 30 % slack, then press Play (Space).
- **Watch the solver work:** press Paused, load an example, then press Step repeatedly. The ΔF arrows shrink as the structure settles. Press Reset to start again.
- **Choose the equilibrium:** press Paused, drag the free nodes of a tensegrity or net to a different starting shape, then press Solve.
- **Hanging-model form-finding (Gaudí):** hang a chain under its loads, use *Bake as unstressed*, then tick **Inverted arch** to see the matching funicular compression form.
- **Save a pretensioned state:** solve, use *Bake shape*, then save. The file reopens in that state, and Reset returns to it.

Undo covers Solve, the first Step, Reset and Bake. Save files store both positions. Files written before v1.3 still open, and their drawn positions are taken to be the saved positions.

---

## Method (summary)

- **Cables** are exact elastic catenary elements (Irvine; O’Brien & Francis). One element reproduces a hanging cable with its self-weight and stretch, slack or taut, without subdivision.
- **Deck-loaded cables** are split internally into 16 sub-catenaries. The plan load is lumped at their joints.
- **Bars and struts** are geometrically non-linear two-force members that carry tension or compression.
- **Equilibrium** is found by Newton–Raphson on the free nodal coordinates. The solver uses the exact tangent, a Levenberg shift, and a line search on the energy gradient.
- **The stiffness matrix inspector** (▦) shows the tangent K at the solved shape.
- Units are **kN** and **m**. Positive directions: x to the right, y upwards, tension positive.

**Method** in the top bar gives the full equations and assumptions.

---

## Verification

**✓ Verification tests** runs 104 checks against closed-form solutions: catenary, elastic catenary, funicular, statics, the X-module self-stress, the stiffness-matrix tangent against finite differences, and others. Five checks are new since v1.3:

1. The Play spline passes exactly through every iteration (v1.4).
2. Solving the suspension bridge from its drawn state gives the same H as the live solve.
3. Stepping one iteration at a time converges to the same state as Solve.
4. Bake shape leaves the equilibrium unchanged.
5. Bake as unstressed leaves weightless members with no force, and the load node's out-of-balance force equals the load.

All 104 checks pass in v1.4. All 24 examples solve in Live mode, and they also solve from their drawn state in Paused mode.

---

## Keyboard shortcuts

| Group | Keys |
|---|---|
| Tools | V select · N node · C cable · B strut · X split · 1 pin · 2 roller V · 3 roller H · 4 no support · P point load · U deck load · W self-weight · E length |
| Annotate | T text · G box · J ellipse · L line · K arrow · D dimension |
| Solver | S Live / Paused · Enter solve · . step · Space play / stop · R reset |
| Edit | Ctrl+Z / Ctrl+Y undo / redo · Ctrl+C / Ctrl+V copy / paste · Ctrl+A select all · Del delete · arrow keys nudge (Shift = 5×) |
| View | F fit · I inspector · scroll to zoom · right-drag or middle-drag to pan · Esc cancel |
| File | Ctrl+S save · Ctrl+O open (or drop a file on the app) |

---

## Version history

| Version | Name | Highlights |
|---|---|---|
| **1.4** | **Fall** | ⏵ Play: the solver iterations animated as one smooth movement (spline interpolation between iterations, distance-based timing, ease in and out), speed ½×–4×; 1 new verification check. |
| 1.3 | Bake | Drawn (initial) state for every node. Live / Paused solver; Solve, Step, Reset; Bake shape and Bake as unstressed; out-of-balance ΔF arrows; Drawn state ghost; 4 new verification checks. |
| 1.2 | Matrix | Tangent stiffness matrix inspector: heatmap, element-by-element assembly, δ / F / R strips. |
| 1.1 | Tensegrity | Frame Pusher grid and snapping, hover highlighting, animation, node and member inspector, theme and home buttons, Millennium Dome, cable dome and tensegrity examples. |
| 1.0 | Catenary | First release: elastic catenary solver, drawing tools, loads, examples and gallery, verification tests, save / open, PDF report. |

The full changelog is under **About** in the app.

---

*Cable Pusher was created with Claude (Anthropic) as a sibling of Frame Pusher and Arch Pusher. Home: [ankacha.github.io](https://ankacha.github.io/).*
