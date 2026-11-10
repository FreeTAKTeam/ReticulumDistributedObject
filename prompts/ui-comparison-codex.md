# Codex execution prompt: RDO UI comparison

Implement and evaluate a proof of concept in **FreeTAKTeam/ReticulumDistributedObject** comparing these three desktop clients:

1. **Rune**, using native Rust rendering.
2. **egui + eframe**, using native Rust rendering.
3. **Electron + React + TypeScript**, reusing the existing Dafrion Desktop impact-analysis presentation code.

Produce working code, reproducible commands, functional evidence and measurements on available hardware. Continue through implementation and verification; a plan, empty scaffolding or screenshots of static mockups are insufficient. This is an experiment to choose a UI, not a prior decision that Rust or any candidate must win.

## 1. Objective and scope

Answer these questions with evidence:

- Which candidate best supports a focused knowledge graph, navigation and note editing on the required devices?
- What are its startup, memory, CPU and interaction costs?
- How much of the application cost comes from rendering, and how much comes from the connection to the Rust backend?
- Do measured benefits justify the effort of porting the existing Dafrion presentation?

The intended devices are Windows x86_64, Linux x86_64 and Raspberry Pi OS ARM64. Record the actual hardware and OS used. Do not assume access to all three or infer Pi results from an x86 machine.

Put the experiment under **`experiments/ui-comparison/` in this RDO repository**. Use a contained work branch or worktree according to the repository's instructions, preserving existing user changes. Resolve routine implementation choices autonomously and document material choices.

Keep this work separate from the normative protocol. Do not change `SPECIFICATION.md`, the ontology, accepted proposals or protocol version to accommodate a UI experiment. Treat Dafrion Desktop, Rune, egui and other source repositories as read-only inputs. Do not modify or publish changes to them.

Peer selection and delivery states are local demonstration data. Actual Reticulum networking, LXMF propagation, cryptographic replication, RPP transfers and production Dafrion Core integration are outside this experiment. A demonstration of local persistence is not proof of RDO conformance.

## 2. Read before implementing

Read applicable `AGENTS.md` files and these RDO documents:

- [Reference application design](../reference_Implementation.md).
- [RDO specification](../SPECIFICATION.md), especially ownership, partial replication and transport responsibilities.
- [RDO-0001](../proposals/RDO-0001-selective-lxmf-replication.md).

Use these primary sources:

| Candidate or reference | Source and version policy |
| --- | --- |
| Dafrion presentation baseline | [Dafrion Desktop commit `5942491b8733bb0a5d2cb34a99852cf679c1302f`](https://github.com/FreeTAKTeam/dafrion-desktop/tree/5942491b8733bb0a5d2cb34a99852cf679c1302f). Extract from this inspected commit. Document any necessary departure instead of silently using a different source. |
| Rune | [Repository](https://github.com/anomalyco/rune) and [core documentation](https://github.com/anomalyco/rune/blob/dev/crates/rune/README.md). Resolve a full commit, read its matching documentation and pin it. Start from its native examples; avoid optional development tooling in measured runs. |
| egui/eframe | [Repository](https://github.com/emilk/egui), [Scene](https://docs.rs/egui/latest/egui/containers/struct.Scene.html), [TextEdit](https://docs.rs/egui/latest/egui/widgets/text_edit/struct.TextEdit.html) and [renderer selection](https://docs.rs/eframe/latest/eframe/enum.Renderer.html). Select and pin a released version; use documentation matching that version. |
| Electron measurement guidance | [Process model](https://www.electronjs.org/docs/latest/tutorial/process-model) and [performance guidance](https://www.electronjs.org/docs/latest/tutorial/performance). |

Record exact resolved dependency versions, Rust/toolchain versions, lockfiles and graphics settings. Manifest version ranges are not resolved versions. Use the pinned Dafrion dependency choices where practical for the Electron extraction, and explain compatibility changes.

## 3. Deliverable structure

Use a small local Cargo workspace and a separate Electron package within the experiment. The following paths are suggested; minor changes are acceptable when documented:

| Path below `experiments/ui-comparison/` | Purpose |
| --- | --- |
| `apps/rune/` | Rune executable. |
| `apps/egui/` | egui/eframe executable. |
| `apps/electron/` | Minimal production Electron host and reused React presentation. |
| `crates/poc-core/` | Shared Rust data access, navigation, note persistence and peer-selection logic for the integrated pass. |
| `crates/poc-service/` | Thin local wrapper over that same Rust core for the Electron integrated pass. |
| `fixtures/` | Seeded graph data, prepared scenes and expected states. |
| `scenarios/` | One repeatable action sequence consumed by all candidates. |
| `tools/` | Small build/run/measurement helpers. |
| `results/` | Run metadata, small raw results and representative screenshots. |
| `README.md`, `REPORT.md` | Exact operating instructions and measured findings. |
| `provenance.json`, `third-party/` | Source commits, extraction details and applicable license/NOTICE files. |

Avoid a new benchmark service, dashboard, plugin system or production database. Keep build caches, dependency directories, executables and large recordings out of Git. Retain small raw result files needed to check the report. Use deterministic synthetic data; do not depend on a live enterprise twin or private production content.

## 4. Extract and reuse the Dafrion presentation

The inspected graph uses **React/HTML cards, SVG connections and CSS transitions**. It is not a React Flow or ECharts component.

The main source is [DafrionImpactVisualization.tsx](https://github.com/FreeTAKTeam/dafrion-desktop/blob/5942491b8733bb0a5d2cb34a99852cf679c1302f/packages/desktop/apps/electron/src/renderer/components/product/dafrion/DafrionImpactVisualization.tsx).

Under `packages/desktop/apps/electron/src/renderer/components/product/dafrion/`, inspect and extract only what is needed:

- `DafrionImpactVisualization.tsx`: preserve or adapt `impactStarModel`, `impactStarLayout`, `impactSpokeEndpoints`, rendering, selection, panning and zoom behavior.
- `DafrionLaneBoard.tsx`: card/header/message presentation, accents and size constants; the complete lane-board view is unnecessary.
- `DafrionToolbar.tsx`: buttons and zoom controls.
- `DafrionDashboardData.ts`: the small `DafrionViewState` shape only.
- `DafrionImpactVisualization.test.tsx`: useful fixtures and behavioral expectations.

Also inspect `src/shared/dafrion-core.ts` for scene/hit/connection types, and `src/renderer/dafrion-atmosphere.css` and `src/renderer/index.css` for the required graph styles, fonts, theme tokens and Tailwind configuration. These paths are relative to `packages/desktop/apps/electron/`.

The graph accepts a scene through props; it does not need a running Core server. The harness owns focus, selection and history, and responds to `onFocusChange` by supplying the next scene. Do not import the full dashboard, agent runtime or Core client merely to display fixtures.

Decouple two existing dependencies deliberately: the card's deletion/menu machinery, and the graph's static `DafrionBrainAuthoring` import. Use small optional callback/render interfaces or omit those integrations from the extracted presenter. Merely omitting an optional prop does not remove its static dependency tree.

Retain source provenance for both the React extraction and Rust translations of layout behavior. Record upstream repository, commit, paths, extracted symbols, destination files and modifications. Preserve applicable upstream copyright, license and NOTICE attribution and mark modified files. Inspect the upstream [desktop LICENSE](https://github.com/FreeTAKTeam/dafrion-desktop/blob/5942491b8733bb0a5d2cb34a99852cf679c1302f/packages/desktop/LICENSE) and [NOTICE](https://github.com/FreeTAKTeam/dafrion-desktop/blob/5942491b8733bb0a5d2cb34a99852cf679c1302f/packages/desktop/NOTICE). An SDK mentioned in the upstream full-product NOTICE is not a dependency to add to this extraction.

The current layout uses concentric elliptical rings, and the viewport centers through scroll positioning. Use that as the initial common visual baseline. TheBrain-style parent/child regions and a more elaborate traversal animation can be later work; do not implement them in just one candidate and compare unlike workloads.

## 5. Establish equivalent behavior before measuring

All three clients must implement:

1. Open a focused node and show its immediate neighborhood, relationship directions, labels and evidence counts.
2. Select a neighbor, update the inspector and recenter on its neighborhood.
3. Pan, zoom, center, navigate back/forward and search by label.
4. Inspect and edit a multiline note, retaining changes when changing focus and after application restart.
5. Select recipients from three labeled saved peers and retain that selection locally.
6. Display an unavailable referenced node and a visibly incomplete attachment reference without blocking access to available content.

Provide the same useful note behavior: selection, clipboard, keyboard focus, undo where supported, accented text and multiline input. Record any mismatch as a parity defect. Plain text or Markdown source is enough; a rich-text editor is outside this experiment.

Designate editable demonstration records as locally owned. A non-owned example is read-only or visibly records a pending request; saving a replica does not grant authority. Fixture projections may group independent RDO-style records for display, but must not redefine authoritative Property/Relationship storage or canonical Destination identity. Legacy Dafrion scene keys remain adapter keys, not a new RDO addressing scheme.

Use matching viewport dimensions, font files and sizes where practical, node dimensions, colors, connection styles, zoom range and animation duration. The initial Dafrion card transition is 300 ms. Make unavoidable rendering differences explicit. Include zoomed-label readability in the functional review.

Match edge-pulse/decorative animation settings across clients. Provide a motion-off mode for idle measurement and an equivalent motion-on workload. No candidate should continuously repaint during a supposedly idle test simply because its loop was configured differently.

## 6. Fixtures and repeatable scenarios

Create three deterministic workloads:

- **Typical:** a modest neighborhood shaped like the current Dafrion impact view.
- **Dense:** a selected node with substantially more relationships, including multiple relationship records to one neighbor and both incoming and outgoing links.
- **Large store:** many local records while displaying only a small focused neighborhood, to distinguish data-access cost from drawing cost.

Choose and document exact sizes and a fixed seed. Include long labels, accented labels, multiline notes, unavailable references and unrelated records excluded from the displayed neighborhood. Report stored records, neighborhood nodes/relationships, submitted drawing elements and on-screen counts separately where available.

Prepare the ordered neighborhoods and coordinates before the rendering pass. Ported Rust geometry should match the baseline's expected positions within a documented tolerance. JavaScript `localeCompare` can produce a different ordering from Rust comparisons; fix the order in the fixtures or use an explicitly identical rule. Preserve IDs, directions and aggregation counts.

Use one scenario definition with candidate adapters. It should launch the app, load the same scene, visit a fixed sequence of focus IDs, pan and zoom, navigate back, search, edit/revisit a note, change a peer selection and settle idle for 30–60 seconds. Verify the resulting application state. Include a separate restart check for persistence; give each candidate an isolated state directory reset to the same starting data.

Internal action replay is acceptable for repeatable performance work. Label it as programmatic replay and also exercise actual pointer/keyboard interactions; internal callbacks alone do not establish that the UI controls work. Keep telemetry buffered or lightweight so logging does not dominate the measured work.

## 7. Implement two measurement passes

### Pass A: prepared rendering and local interaction

Load the same prepared scenes directly into each client. Keep data resident before timing interactive actions. Exclude service calls, graph queries and layout computation from the interaction interval. Report startup/fixture loading separately.

Use the same positions and animation parameters in all three. Where necessary, add an optional prepared-scene input to the extracted React presenter while retaining its real drawing and input behavior. Do not replace it with an unrelated web graph implementation. The Rust clients draw equivalent content using their own UI APIs.

The comparison is a minimal Electron application against minimal native applications, each implementing the same feature set. A separate observation of the full Dafrion application may be useful but must not enter this rendering ranking.

### Pass B: integrated application

Use one shared Rust implementation for lookup/search, neighborhood preparation, local note storage and peer selections. Both native clients call it in-process without blocking the UI thread. Electron reaches the same implementation through a thin local service or IPC wrapper.

Choose one straightforward local transport, document it and keep the service restricted to the local machine. Keep the service thin; do not introduce a second backend implementation in TypeScript. Persist only the small demonstration state needed for this experiment, using one storage strategy for all integrated clients.

Return the focused neighborhood and required content, rather than repeatedly transferring the complete graph. Time backend computation, encoding/transfer and frontend update separately when possible. Record response sizes and call counts. If a boundary cannot be measured independently, report the combined interval with its actual meaning instead of inventing a decomposition.

Include the service in Electron's complete-application memory and startup totals. Its extra process and serialization are real integration costs, but they are not evidence that SVG drawing alone is slower. Include the native core's memory in the corresponding native totals.

## 8. Platform checks and evidence

### Check graphics early

Before investing in polished ports, launch a basic native window, draw a small scene and edit text with each candidate on available target hardware. Test the actual Pi early if it is accessible. Record GPU adapter, graphics API, driver, display system, scaling and hardware acceleration versus software fallback.

For egui, use the selected normal renderer first. Its `glow` backend is a separately labeled compatibility trial if needed. Known [Pi/wgpu reports](https://github.com/emilk/egui/issues/8311) justify checking device limits; they are not proof that every current build fails. Inspect the selected release rather than assuming a past report still applies.

Rune is a pinned pre-release candidate. Investigate build/runtime failures and report the reproducible cause; do not silently substitute another UI. The [Rune Windows memory report](https://github.com/anomalyco/rune/issues/430) is a reason to measure the chosen build and driver, not a benchmark result to copy into this experiment.

### Measurement rules

- Use optimized Rust release builds and production Electron assets. Disable development servers, hot reload and open developer tools during recorded runs.
- Compare candidates on the same physical machine and OS, using the same viewport, display scaling and scenario. Keep cross-platform results in separate groups.
- Record first-run/cache state honestly. A process restart with warm disk/shader caches is not a cold-cache measurement.
- Start with three runs per candidate/pass/workload on available hardware, rotating candidate order. Investigate a concrete noisy or contradictory result before increasing repetitions.
- Measure the whole application process tree, including Electron renderer/GPU processes and the shared-core service. Report unavailable counters explicitly.
- On Linux, prefer process-tree proportional set size where available to avoid repeatedly counting shared pages. Label RSS, PSS, working set and private bytes accurately; do not treat them as interchangeable or rank unlike measures across operating systems.
- Report GPU allocations separately where measurable. On a Pi, account for shared system/GPU memory without adding the same memory twice.
- Record startup to a usable graph; memory after settling and at peak activity; active and settled-idle CPU; first response to an action; frame consistency during navigation; and functional defects.
- Fix animation duration. Distinguish first response, frame delivery and animation completion. A shorter animation is not a faster renderer.
- Use presentation timing where supported. Label event-to-submit, callback or CPU-frame timings as approximations; neither a browser animation callback nor a native GPU submission proves the pixels were displayed then.
- Record exact units, sampling intervals, run IDs, source commits, dependency versions, machine details and measurement methods with the raw results. Screenshots and headless tests support functional evidence, not target-device performance claims.

If the environment has only software graphics, collect and label those results; they cannot establish the hardware-accelerated winner. If Windows or Pi hardware is unavailable, continue the implementation and available-host tests, provide exact target run instructions and mark those rows **not measured**. Compilation, cross-compilation and emulation are not physical-device measurements.

## 9. Execution sequence

1. Inspect repository instructions, dependency access and available runtime/hardware. Record a short plan in the experiment README.
2. Freeze the fixture/scenario format and expected navigation states. Pin source versions and complete early graphics checks where possible.
3. Extract the minimal Electron presentation and establish it as the behavioral baseline. Preserve attribution and document extraction changes.
4. Implement Rune and egui clients. Frontend tasks may run in parallel after the shared inputs are fixed; keep changes to shared contracts coordinated.
5. Verify equivalent graph, editing and peer-selection behavior, then run Pass A.
6. Implement the common Rust core and its thin Electron service adapter. Verify persistence and responsiveness, then run Pass B.
7. Collect available-host measurements, check material discrepancies, and produce the report and remaining target instructions.

Use focused checks for real risks: neighborhood aggregation/direction, deterministic geometry, selection/focus/back behavior, note and peer-selection persistence, and the same integrated operation results. Reuse relevant upstream tests where practical. Include a genuine UI interaction check for each client, using its supported test facilities or desktop automation. Do not substitute a passing data test for a working window.

When a dependency, tool or device is unavailable, record the exact failed command and cause, try a reasonable supported alternative, and complete independent work. A genuine blocker may leave one candidate incomplete; the report must identify that limitation rather than claim all three were delivered.

## 10. Completion criteria and final report

Provide a small documented runner with build, launch, scenario and report operations. It should accept candidate, pass, fixture and output-directory choices. Use a simple script or small tool already suited to the workspace; do not build a permanent benchmarking platform.

The README must contain copyable commands to install prerequisites, build each candidate, run both passes, reset demonstration state, reproduce the scenario and collect results on Windows, Linux and Raspberry Pi OS. Identify commands actually executed and commands prepared for untested targets.

`REPORT.md` must contain:

- A status table distinguishing **implementation**, **functional verification** and **measurement coverage** for each candidate and target.
- Exact source/version/hardware information and links to the corresponding raw result files.
- Separate tables for prepared rendering and the integrated application, with units and methods.
- Visual/interaction findings, including zoomed labels, editing and keyboard behavior.
- A concise account of reused code, porting work, toolkit-specific workarounds and unresolved defects.
- A recommendation based on platform support, functional parity, usable performance and maintenance effort, with observed facts separated from interpretation.
- Exact commands for remaining hardware runs and the evidence still needed for a final platform decision.

Do not invent performance budgets or a weighted score that predetermines the winner. If results are effectively tied, say so and explain the value of code reuse. If a native candidate is better on one device and worse on another, preserve that distinction. A final platform recommendation requires evidence for the required platforms; a partial recommendation must name its measured scope.

Before handing back the work, verify that recorded commands match the delivered files and that the report describes the actual tested builds. Summarize what runs, what was tested, what was measured, genuine blockers and the next exact commands. Do not present simulated delivery states as real replication, or unmeasured hardware as completed validation.
