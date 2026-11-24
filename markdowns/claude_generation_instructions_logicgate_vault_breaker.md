# Generation Instructions: *LogicGate: Vault Breaker* (Grade 8–9 Browser Mini-Game)

Use this single-document specification and prompt to generate **LogicGate: Vault Breaker** in Claude.ai (or similar assistants). It targets **Grade 8–9 learners** and keeps everything self-contained in a single `.html` file (vanilla HTML, CSS, and JavaScript—zero external dependencies) while organizing the internal JavaScript into clean, modular sections so future levels, editors, and artwork upgrades are easy to maintain.

---

## 1. Core Educational & Gameplay Goals
* **Audience:** Grade 8–9 middle school students learning Boolean logic (`AND`, `OR`, `NOT`, `XOR`, `NAND`, `NOR`), binary states (`0` / `1`), truth tables, and gate budgets.
* **Standalone Delivery:** Runs directly in any modern browser from a single `.html` file.
* **Core Gameplay Loop:**
  1. Load a vault puzzle level with labeled **Input sensors** (`0` / `1`), **Output lock(s)** (`0` = Locked, `1` = Unlocked), and a **Gate Limit**.
  2. Click inputs to toggle `0` / `1`, place logic gates on the canvas, and connect wires between nodes.
  3. Inspect the **Live Truth Table** (clicking rows to test variable states or running an automated truth-table sweep).
  4. Click **Test Vault Lock** to verify the circuit within the allowed gate count, or open **Show Solution** for guided steps.

---

## 2. Modular Architecture Requirements (Inside Single-File HTML)
Even though the output is a single `.html` file, structure the `<script>` cleanly into distinct, well-commented modules with clear state and helper boundaries:

1. **`Assets & Media Registry` (`ASSET_CONFIG`):**
   * Centralizes optional background image URLs/data-URIs and object/node sprite overrides (`input`, `output`, `AND`, `OR`, `NOT`, `XOR`, `NAND`, `NOR`, and canvas background).
   * Falls back gracefully to crisp vector/canvas rendering whenever an image is `null` or fails to load, ensuring zero broken visuals offline.
2. **`Levels & Schema Module` (`LEVELS`, `GATE_TYPES`):**
   * Stores puzzle definitions (`id`, `title`, `briefing`, `maxGates`, `backgroundKey`, `nodes`, `wires`, `solutionSteps`).
   * Supports adding or editing puzzles without touching simulation or rendering internals.
3. **`Level Editor Module`:**
   * Provides a toggleable **Level Editor** mode/drawer so teachers or advanced students can create/edit puzzles: add/move/remove input or output nodes, rename labels, set initial `0`/`1` values, adjust `maxGates`, update the briefing, and **Export / Import Level JSON** (copy or download JSON).
4. **`Visual & Asset Replacement Module`:**
   * Allows replacing the **canvas background image** and **device/gate object images** at runtime (via image URL input or local file upload using `FileReader` / object URLs) as well as resetting back to default vector graphics.
5. **`Logic & Truth Table Engine`:**
   * Evaluates cascaded gates (`AND`, `OR`, `NOT`, `XOR`, `NAND`, `NOR`), generates all $2^n$ binary input rows for the truth table, highlights active rows, and supports automated truth-table sweeps.
6. **`Canvas Renderer & Interaction Controller`:**
   * Handles node selection, wire creation, input toggling, editor placement/dragging, animated signal pulses on active (`1`) wires, and modal dialogs (**Show Solution**, **Level Editor**, **Art & Background**).

---

## 3. Ready-to-Paste Prompt for Claude.ai

Copy and paste everything inside the block below into Claude.ai to generate the expanded application:

```text
Build a single-file, zero-dependency HTML/CSS/JS educational game called "LogicGate: Vault Breaker" for Grade 8–9 learners, based on the reference prototype below.

Keep everything in ONE standalone .html file, but organize the JavaScript into clear, well-commented modules so it is easy to maintain and expand:
1. Asset & Media Registry (ASSET_CONFIG): Support optional background image and object/gate/node image replacements with automatic fallback to clean canvas/vector drawing if no custom image is set or if an image fails to load. Include a UI modal/panel ("Art & Background") where users can upload local image files or enter image URLs for the floor/vault background and for Input, Output, and Gate objects, plus a "Reset Default Art" button.
2. Levels & Schema Module: Keep the existing 5 puzzles (AND, NOT+OR, XOR+AND, NAND, NOR+AND) and add a Level Editor ("Level Editor" button/drawer) allowing users to edit the current level's title, briefing, maxGates, and nodes (add Input/Output nodes, toggle default 0/1, move/delete nodes) and export/copy the updated JSON level object.
3. Logic & Truth Table Engine: Preserve live truth table generation across all 2^n input combinations, click-to-apply rows, animated "Simulate Truth Table" sweep, gate-limit checking, "Show Solution" steps, and vault lock testing.
4. Canvas Renderer & UI: Preserve the dark slate educational UI, toolbar, truth table panel, and console log, while ensuring custom background/object images draw cleanly when provided.

Reference Prototype (v2):
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LogicGate: Vault Breaker v2 — Truth Table &amp; Puzzles</title>
<style>
  :root {
    --bg: #0f172a;
    --panel: #1e293b;
    --border: #334155;
    --text: #f8fafc;
    --muted: #94a3b8;
    --accent: #38bdf8;
    --green: #22c55e;
    --red: #ef4444;
    --amber: #f59e0b;
  }
  * { box-sizing: border-box; font-family: 'Segoe UI', system-ui, sans-serif; }
  body { margin: 0; background: var(--bg); color: var(--text); display: flex; flex-direction: column; height: 100vh; overflow: hidden; }

  header {
    display: flex; justify-content: space-between; align-items: center; gap: 12px; flex-wrap: wrap;
    padding: 9px 18px; background: var(--panel); border-bottom: 1px solid var(--border);
  }
  .brand-level { display: flex; align-items: center; gap: 10px; }
  select.level-select {
    background: var(--bg); color: var(--text); border: 1px solid var(--accent);
    padding: 6px 10px; border-radius: 6px; font-weight: 600; font-size: 0.88rem;
  }
  .hud-stats { display: flex; gap: 10px; font-weight: 600; font-size: 0.85rem; flex-wrap: wrap; }
  .stat-box { background: var(--bg); padding: 5px 10px; border-radius: 6px; border: 1px solid var(--border); }
  .ok { color: var(--green); }
  .bad { color: var(--red); }

  main { display: grid; grid-template-columns: 1fr 410px; flex: 1; overflow: hidden; }

  .board-area { display: flex; flex-direction: column; padding: 10px 12px; gap: 8px; overflow: hidden; }
  .toolbar { display: flex; gap: 5px; flex-wrap: wrap; align-items: center; }
  button {
    background: var(--panel); color: var(--text); border: 1px solid var(--border);
    padding: 6px 9px; border-radius: 6px; cursor: pointer; font-weight: 600; font-size: 0.79rem;
    transition: all 0.14s;
  }
  button:hover { border-color: var(--accent); }
  button.active { background: var(--accent); color: #0f172a; border-color: var(--accent); }
  button.tt-btn { background: #0369a1; border-color: #38bdf8; color: #ffffff; }
  button.tt-btn:hover { background: #075985; }
  button.sol-btn { background: #312e81; border-color: #818cf8; color: #e0e7ff; }
  button.sol-btn:hover { background: #4338ca; }
  button.run-btn { background: var(--green); color: #052e16; border: none; margin-left: auto; font-size: 0.84rem; }
  button.run-btn:hover { filter: brightness(1.08); }
  button.reset-btn { background: #475569; border: none; }

  .hint-bar {
    background: #0b1120; border: 1px solid var(--border); border-radius: 8px;
    padding: 6px 12px; font-size: 0.8rem; color: #cbd5e1;
  }

  .canvas-wrap {
    flex: 1; background: #090d16; border: 1px solid var(--border); border-radius: 8px;
    display: flex; justify-content: center; align-items: center; position: relative;
  }
  canvas { background: #0f172a; border-radius: 4px; cursor: crosshair; }

  .side-panel {
    background: var(--panel); border-left: 1px solid var(--border);
    padding: 12px; display: flex; flex-direction: column; gap: 9px; overflow-y: auto;
  }
  .side-panel h2 { margin: 0; font-size: 0.98rem; color: var(--accent); }
  .briefing { font-size: 0.81rem; color: var(--muted); line-height: 1.4; background: var(--bg); padding: 8px 10px; border-radius: 6px; }

  .tt-card {
    background: #0b1120; border: 1px solid var(--border); border-radius: 8px; padding: 8px 10px;
  }
  .tt-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px; }
  .tt-header strong { font-size: 0.84rem; color: var(--amber); }
  table.tt-table { width: 100%; border-collapse: collapse; font-family: monospace; font-size: 0.78rem; }
  table.tt-table th, table.tt-table td {
    border: 1px solid #1e293b; padding: 4px 6px; text-align: center;
  }
  table.tt-table th { background: #1e293b; color: #e2e8f0; }
  table.tt-table tr.tt-row { cursor: pointer; }
  table.tt-table tr.tt-row:hover { background: rgba(56, 189, 248, 0.12); }
  table.tt-table tr.tt-active { background: rgba(250, 204, 21, 0.22); outline: 1px solid #facc15; }
  .bit-1 { color: #4ade80; font-weight: 700; }
  .bit-0 { color: #94a3b8; }

  .ref-box {
    background: #0b1120; border: 1px solid var(--border); border-radius: 8px; padding: 8px 10px; font-size: 0.76rem;
    font-family: monospace; color: #cbd5e1; line-height: 1.42;
  }
  .ref-box strong { color: var(--accent); font-family: 'Segoe UI', sans-serif; display: block; margin-bottom: 3px; }

  .console-log {
    flex: 1; min-height: 85px; background: #020617; border: 1px solid var(--border); border-radius: 6px;
    padding: 8px; font-family: monospace; font-size: 0.77rem; overflow-y: auto; line-height: 1.4; color: #cbd5e1;
  }
  .log-ok { color: var(--green); }
  .log-err { color: var(--red); }
  .log-info { color: var(--accent); }

  .modal-backdrop {
    display: none; position: fixed; inset: 0; background: rgba(2, 6, 23, 0.78);
    justify-content: center; align-items: center; z-index: 60; padding: 20px;
  }
  .modal-backdrop.open { display: flex; }
  .modal-card {
    background: var(--panel); border: 1px solid var(--border); border-radius: 10px;
    max-width: 600px; width: 100%; max-height: 85vh; display: flex; flex-direction: column; overflow: hidden;
  }
  .modal-header {
    display: flex; justify-content: space-between; align-items: center;
    padding: 12px 18px; background: #0b1120; border-bottom: 1px solid var(--border);
  }
  .modal-header h3 { margin: 0; font-size: 0.98rem; color: var(--accent); }
  .modal-body { padding: 14px 18px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }
  .modal-footer {
    padding: 11px 18px; background: #0b1120; border-top: 1px solid var(--border);
    display: flex; justify-content: flex-end;
  }
  .step-item {
    background: var(--bg); border: 1px solid var(--border); border-radius: 8px;
    padding: 10px 12px; display: flex; gap: 10px; align-items: flex-start;
  }
  .step-num {
    background: var(--accent); color: #0f172a; font-weight: 800; font-size: 0.8rem;
    width: 22px; height: 22px; border-radius: 50%; display: flex; align-items: center; justify-content: center; flex-shrink: 0;
  }
  .step-content strong { display: block; font-size: 0.87rem; margin-bottom: 2px; color: #f8fafc; }
  .step-content p { margin: 0; font-size: 0.81rem; color: var(--muted); line-height: 1.45; }
</style>
</head>
<body>

<header>
  <div class="brand-level">
    <strong>LogicGate v2</strong>
    <select id="levelSelector" class="level-select" onchange="loadLevel(this.value)"></select>
  </div>
  <div class="hud-stats">
    <div class="stat-box">Gates Used: <span id="gateCountDisplay" class="ok">0</span> / <span id="gateLimitDisplay">0</span></div>
    <div class="stat-box">Vault Status: <span id="vaultStatusDisplay" class="bad">LOCKED (0)</span></div>
  </div>
</header>

<main>
  <section class="board-area">
    <div class="toolbar">
      <button class="tool-btn active" data-tool="wire">🔌 Wire Tool</button>
      <button class="tool-btn" data-tool="AND">AND</button>
      <button class="tool-btn" data-tool="OR">OR</button>
      <button class="tool-btn" data-tool="NOT">NOT</button>
      <button class="tool-btn" data-tool="XOR">XOR</button>
      <button class="tool-btn" data-tool="NAND">NAND</button>
      <button class="tool-btn" data-tool="NOR">NOR</button>
      <button class="tt-btn" onclick="runTruthTableSimulation()">📊 Simulate Truth Table</button>
      <button class="reset-btn" onclick="resetLevel()">Clear</button>
      <button class="sol-btn" onclick="openSolution()">💡 Show Solution</button>
      <button class="run-btn" onclick="verifyVault()">🔓 Test Vault Lock</button>
    </div>

    <div class="hint-bar">
      <strong>Tip:</strong> Click any row in the <strong>Truth Table</strong> to test that variable state, or click <strong>📊 Simulate Truth Table</strong> to step through every state automatically.
    </div>

    <div class="canvas-wrap">
      <canvas id="circuitCanvas" width="760" height="456"></canvas>
    </div>
  </section>

  <aside class="side-panel">
    <h2>Puzzle Briefing</h2>
    <div id="briefingBox" class="briefing"></div>

    <div class="tt-card">
      <div class="tt-header">
        <strong>Live Truth Table</strong>
        <span style="font-size:0.74rem; color:var(--muted);">Click row to apply</span>
      </div>
      <table class="tt-table" id="truthTable"></table>
    </div>

    <div class="ref-box">
      <strong>Gate Logic Summary</strong>
      <div>AND  : 1 only if ALL inputs = 1</div>
      <div>OR   : 1 if ANY input = 1</div>
      <div>NOT  : inverts 1 input (0↔1)</div>
      <div>XOR  : 1 if inputs DIFFER</div>
      <div>NAND : 0 only if ALL inputs = 1</div>
      <div>NOR  : 1 only if ALL inputs = 0</div>
    </div>

    <div class="console-log" id="consoleLog"></div>
  </aside>
</main>

<div id="solutionModal" class="modal-backdrop" role="dialog" aria-modal="true">
  <div class="modal-card">
    <div class="modal-header">
      <h3 id="solutionTitle">Vault Walkthrough</h3>
      <button onclick="closeSolution()">✕ Close</button>
    </div>
    <div id="solutionSteps" class="modal-body"></div>
    <div class="modal-footer">
      <button class="sol-btn" onclick="closeSolution()">Back to Circuit</button>
    </div>
  </div>
</div>

<script>
const LEVELS = [
  {
    id: 1,
    title: "Puzzle 1: Dual Keycard Lock (AND)",
    briefing: "Both KEYCARD_A and BADGE_B must be 1 to unlock VAULT_BOLT. Notice in the truth table that only row (1, 1) produces 1.",
    maxGates: 2,
    nodes: [
      { id: "in_a",  type: "input",  label: "KEYCARD_A",  x: 130, y: 140, val: 1 },
      { id: "in_b",  type: "input",  label: "BADGE_B",    x: 130, y: 310, val: 1 },
      { id: "out_1", type: "output", label: "VAULT_BOLT", x: 610, y: 225, val: 0 }
    ],
    solutionSteps: [
      { title: "Place an AND gate at (370, 225)", detail: "Select 'AND' and click near the middle of the canvas." },
      { title: "Wire KEYCARD_A and BADGE_B to AND", detail: "Use 'Wire Tool' to connect KEYCARD_A → AND and BADGE_B → AND." },
      { title: "Wire AND to VAULT_BOLT", detail: "Connect AND → VAULT_BOLT, set both inputs to 1, and click '🔓 Test Vault Lock'." }
    ]
  },
  {
    id: 2,
    title: "Puzzle 2: Guard Bypass (NOT + OR)",
    briefing: "NIGHT_GUARD starts at 1. Invert NIGHT_GUARD with a NOT gate, then route OVERRIDE_KEY and the inverted guard signal into an OR gate.",
    maxGates: 3,
    nodes: [
      { id: "in_a",  type: "input",  label: "OVERRIDE_KEY", x: 130, y: 130, val: 0 },
      { id: "in_b",  type: "input",  label: "NIGHT_GUARD",  x: 130, y: 320, val: 1 },
      { id: "out_1", type: "output", label: "INNER_DOOR",   x: 610, y: 225, val: 0 }
    ],
    solutionSteps: [
      { title: "Place a NOT gate at (280, 320) and an OR gate at (440, 225)", detail: "Select each gate tool and place them on the canvas." },
      { title: "Wire NIGHT_GUARD → NOT → OR and OVERRIDE_KEY → OR", detail: "Connect NIGHT_GUARD to NOT, then NOT to OR; also connect OVERRIDE_KEY to OR." },
      { title: "Wire OR → INNER_DOOR and test", detail: "Connect OR to INNER_DOOR and click '🔓 Test Vault Lock'." }
    ]
  },
  {
    id: 3,
    title: "Puzzle 3: Laser Disarm (XOR + AND)",
    briefing: "LASER_1 and LASER_2 must differ (XOR = 1) while MASTER_KEY is 1 to disarm CORE_VAULT.",
    maxGates: 3,
    nodes: [
      { id: "in_a",  type: "input",  label: "LASER_1",    x: 130, y: 110, val: 1 },
      { id: "in_b",  type: "input",  label: "LASER_2",    x: 130, y: 230, val: 0 },
      { id: "in_c",  type: "input",  label: "MASTER_KEY", x: 130, y: 350, val: 1 },
      { id: "out_1", type: "output", label: "CORE_VAULT", x: 610, y: 230, val: 0 }
    ],
    solutionSteps: [
      { title: "Place an XOR gate at (310, 170) and an AND gate at (465, 230)", detail: "Place both gates on the canvas." },
      { title: "Wire LASER_1 and LASER_2 into XOR", detail: "Connect LASER_1 → XOR and LASER_2 → XOR." },
      { title: "Wire XOR and MASTER_KEY into AND, then AND → CORE_VAULT", detail: "Complete the path to CORE_VAULT and click '🔓 Test Vault Lock'." }
    ]
  },
  {
    id: 4,
    title: "Puzzle 4: Anti-Tamper Failsafe (NAND)",
    briefing: "SAFE_LOCK stays unlocked (1) UNLESS both SENSOR_X and SENSOR_Y trip together (1, 1). Use a NAND gate to open SAFE_LOCK.",
    maxGates: 2,
    nodes: [
      { id: "in_a",  type: "input",  label: "SENSOR_X",  x: 130, y: 140, val: 1 },
      { id: "in_b",  type: "input",  label: "SENSOR_Y",  x: 130, y: 310, val: 0 },
      { id: "out_1", type: "output", label: "SAFE_LOCK", x: 610, y: 225, val: 0 }
    ],
    solutionSteps: [
      { title: "Place a NAND gate at (370, 225)", detail: "Select 'NAND' and place it near the center." },
      { title: "Wire SENSOR_X, SENSOR_Y → NAND → SAFE_LOCK", detail: "Connect both inputs into NAND and wire NAND to SAFE_LOCK." },
      { title: "Verify in Truth Table & test", detail: "Check that (1, 0) yields 1 and click '🔓 Test Vault Lock'." }
    ]
  },
  {
    id: 5,
    title: "Puzzle 5: Quiet Zone Override (NOR + AND)",
    briefing: "QUIET_VAULT unlocks when BOTH NOISE_1 and NOISE_2 are silent (0, 0) via a NOR gate, AND TECH_PASS is active (1).",
    maxGates: 3,
    nodes: [
      { id: "in_a",  type: "input",  label: "NOISE_1",    x: 130, y: 110, val: 0 },
      { id: "in_b",  type: "input",  label: "NOISE_2",    x: 130, y: 230, val: 0 },
      { id: "in_c",  type: "input",  label: "TECH_PASS",  x: 130, y: 350, val: 1 },
      { id: "out_1", type: "output", label: "QUIET_VAULT", x: 610, y: 230, val: 0 }
    ],
    solutionSteps: [
      { title: "Place a NOR gate at (310, 170) and an AND gate at (465, 230)", detail: "Place both gates on the canvas." },
      { title: "Wire NOISE_1 and NOISE_2 into NOR", detail: "Connect NOISE_1 → NOR and NOISE_2 → NOR." },
      { title: "Wire NOR and TECH_PASS into AND, then AND → QUIET_VAULT", detail: "Connect AND to QUIET_VAULT and click '🔓 Test Vault Lock'." }
    ]
  }
];

const GATE_TYPES = ['AND', 'OR', 'NOT', 'XOR', 'NAND', 'NOR'];
let currentLevel = LEVELS[0];
let nodes = [];
let wires = [];
let activeTool = 'wire';
let selectedSource = null;
let activeRowIdx = null;
let ttSimTimer = null;
let pulseOffset = 0;

const canvas = document.getElementById('circuitCanvas');
const ctx = canvas.getContext('2d');

const levelSel = document.getElementById('levelSelector');
LEVELS.forEach(l => {
  const opt = document.createElement('option');
  opt.value = l.id;
  opt.textContent = l.title;
  levelSel.appendChild(opt);
});

document.querySelectorAll('.tool-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.tool-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    activeTool = btn.dataset.tool;
    selectedSource = null;
    draw();
  });
});

function logMsg(msg, cls = '') {
  const el = document.getElementById('consoleLog');
  el.innerHTML += `<div class="${cls}">&gt; ${msg}</div>`;
  el.scrollTop = el.scrollHeight;
}

function loadLevel(id) {
  stopTtSimulation();
  currentLevel = LEVELS.find(l => l.id === Number(id)) || LEVELS[0];
  levelSel.value = currentLevel.id;
  document.getElementById('briefingBox').textContent = currentLevel.briefing;
  document.getElementById('gateLimitDisplay').textContent = currentLevel.maxGates;
  resetLevel();
}

function resetLevel() {
  stopTtSimulation();
  activeRowIdx = null;
  nodes = JSON.parse(JSON.stringify(currentLevel.nodes));
  wires = [];
  selectedSource = null;
  document.getElementById('consoleLog').innerHTML = `<div class="log-info">&gt; Loaded ${currentLevel.title}.</div>`;
  evaluateAndRenderTruthTable();
  draw();
}

function evalGate(type, inputs) {
  const a = inputs[0] || 0;
  const b = inputs[1] || 0;
  if (type === 'AND')  return (a && b) ? 1 : 0;
  if (type === 'OR')   return (a || b) ? 1 : 0;
  if (type === 'NOT')  return a ? 0 : 1;
  if (type === 'XOR')  return (a !== b) ? 1 : 0;
  if (type === 'NAND') return (a && b) ? 0 : 1;
  if (type === 'NOR')  return (a || b) ? 0 : 1;
  return 0;
}

function computeOutputForInputVector(vec) {
  const simNodes = nodes.map(n => ({ ...n }));
  const inputs = simNodes.filter(n => n.type === 'input');
  inputs.forEach((inp, idx) => { inp.val = vec[idx]; });
  simNodes.forEach(n => { if (n.type !== 'input') n.val = 0; });

  for (let pass = 0; pass < simNodes.length + 2; pass++) {
    simNodes.forEach(n => {
      if (n.type === 'input') return;
      const incoming = wires.filter(w => w.to === n.id).map(w => {
        const src = simNodes.find(s => s.id === w.from);
        return src ? src.val : 0;
      });
      if (n.type === 'output') {
        n.val = incoming.some(v => v === 1) ? 1 : 0;
      } else {
        n.val = evalGate(n.type, incoming);
      }
    });
  }
  const outNode = simNodes.find(n => n.type === 'output');
  return outNode ? outNode.val : 0;
}

function generateTruthRows(numInputs) {
  const rows = [];
  const total = 1 << numInputs;
  for (let mask = 0; mask < total; mask++) {
    const bits = [];
    for (let bit = numInputs - 1; bit >= 0; bit--) {
      bits.push((mask >> bit) & 1);
    }
    rows.push(bits);
  }
  return rows;
}

function evaluateAndRenderTruthTable() {
  const inputs = nodes.filter(n => n.type === 'input');
  const outNode = nodes.find(n => n.type === 'output');
  const currentVec = inputs.map(i => i.val);

  nodes.forEach(n => { if (n.type !== 'input') n.val = 0; });
  for (let pass = 0; pass < nodes.length + 2; pass++) {
    nodes.forEach(n => {
      if (n.type === 'input') return;
      const incoming = wires.filter(w => w.to === n.id).map(w => {
        const src = nodes.find(s => s.id === w.from);
        return src ? src.val : 0;
      });
      if (n.type === 'output') {
        n.val = incoming.some(v => v === 1) ? 1 : 0;
      } else {
        n.val = evalGate(n.type, incoming);
      }
    });
  }

  const gatesUsed = nodes.filter(n => GATE_TYPES.includes(n.type)).length;
  const gEl = document.getElementById('gateCountDisplay');
  gEl.textContent = gatesUsed;
  gEl.className = gatesUsed <= currentLevel.maxGates ? 'ok' : 'bad';

  const vEl = document.getElementById('vaultStatusDisplay');
  if (outNode && outNode.val === 1) {
    vEl.textContent = 'UNLOCKED (1)';
    vEl.className = 'ok';
  } else {
    vEl.textContent = 'LOCKED (0)';
    vEl.className = 'bad';
  }

  const table = document.getElementById('truthTable');
  const combinations = generateTruthRows(inputs.length);
  let html = '<thead><tr>';
  inputs.forEach(inp => { html += `<th>${inp.label.split('_')[0]}</th>`; });
  html += `<th>OUT</th></tr></thead><tbody>`;

  combinations.forEach((vec, rIdx) => {
    const outVal = computeOutputForInputVector(vec);
    const matchesCurrent = vec.every((b, i) => b === currentVec[i]);
    const isHighlighted = activeRowIdx === rIdx || (activeRowIdx === null && matchesCurrent);
    html += `<tr class="tt-row ${isHighlighted ? 'tt-active' : ''}" onclick="applyTruthRow(${rIdx})">`;
    vec.forEach(bit => {
      html += `<td class="${bit ? 'bit-1' : 'bit-0'}">${bit}</td>`;
    });
    html += `<td class="${outVal ? 'bit-1' : 'bit-0'}">${outVal}</td></tr>`;
  });
  html += '</tbody>';
  table.innerHTML = html;
}

function applyTruthRow(rowIdx) {
  const inputs = nodes.filter(n => n.type === 'input');
  const combinations = generateTruthRows(inputs.length);
  const vec = combinations[rowIdx];
  if (!vec) return;
  activeRowIdx = rowIdx;
  inputs.forEach((inp, idx) => { inp.val = vec[idx]; });
  evaluateAndRenderTruthTable();
  draw();
}

function stopTtSimulation() {
  if (ttSimTimer) {
    clearInterval(ttSimTimer);
    ttSimTimer = null;
  }
}

function runTruthTableSimulation() {
  stopTtSimulation();
  const inputs = nodes.filter(n => n.type === 'input');
  const combinations = generateTruthRows(inputs.length);
  let idx = 0;
  logMsg(`Simulating all ${combinations.length} truth table states...`, 'log-info');
  applyTruthRow(0);

  ttSimTimer = setInterval(() => {
    idx++;
    if (idx >= combinations.length) {
      stopTtSimulation();
      activeRowIdx = null;
      evaluateAndRenderTruthTable();
      logMsg(`Truth table sweep complete.`, 'log-ok');
      return;
    }
    applyTruthRow(idx);
  }, 750);
}

function verifyVault() {
  evaluateAndRenderTruthTable();
  const gatesUsed = nodes.filter(n => GATE_TYPES.includes(n.type)).length;
  const outNode = nodes.find(n => n.type === 'output');

  if (gatesUsed > currentLevel.maxGates) {
    logMsg(`Gate limit exceeded (${gatesUsed}/${currentLevel.maxGates}).`, 'log-err');
    return;
  }
  if (outNode && outNode.val === 1) {
    logMsg(`VAULT UNLOCKED! ${outNode.label} = 1 with ${gatesUsed}/${currentLevel.maxGates} gates.`, 'log-ok');
  } else {
    logMsg(`Vault remains LOCKED (0). Check truth table & gate wiring.`, 'log-err');
  }
}

canvas.addEventListener('click', (e) => {
  const rect = canvas.getBoundingClientRect();
  const mx = e.clientX - rect.left;
  const my = e.clientY - rect.top;
  const hitNode = nodes.find(n => Math.hypot(n.x - mx, n.y - my) <= 24);

  if (activeTool === 'wire' && hitNode) {
    if (!selectedSource) {
      selectedSource = hitNode;
    } else if (selectedSource.id !== hitNode.id) {
      const exists = wires.some(w => w.from === selectedSource.id && w.to === hitNode.id);
      if (!exists && hitNode.type !== 'input') {
        wires.push({ from: selectedSource.id, to: hitNode.id });
        logMsg(`Wired ${selectedSource.label} → ${hitNode.label}`);
        activeRowIdx = null;
        evaluateAndRenderTruthTable();
      }
      selectedSource = null;
    }
    draw();
    return;
  }

  if (hitNode && hitNode.type === 'input') {
    hitNode.val = hitNode.val ? 0 : 1;
    activeRowIdx = null;
    logMsg(`Flipped ${hitNode.label} to ${hitNode.val}`);
    evaluateAndRenderTruthTable();
    draw();
    return;
  }

  if (!hitNode && GATE_TYPES.includes(activeTool)) {
    nodes.push({
      id: 'g_' + Date.now() + '_' + Math.random().toString(36).slice(2, 5),
      type: activeTool,
      label: activeTool,
      x: mx,
      y: my,
      val: 0
    });
    logMsg(`Placed ${activeTool} gate at (${Math.round(mx)}, ${Math.round(my)})`);
    activeRowIdx = null;
    evaluateAndRenderTruthTable();
    draw();
  }
});

function openSolution() {
  document.getElementById('solutionTitle').textContent = `${currentLevel.title} — Solution Steps`;
  const box = document.getElementById('solutionSteps');
  box.innerHTML = '';
  currentLevel.solutionSteps.forEach((s, i) => {
    box.innerHTML += `<div class="step-item"><div class="step-num">${i + 1}</div><div class="step-content"><strong>${s.title}</strong><p>${s.detail}</p></div></div>`;
  });
  document.getElementById('solutionModal').classList.add('open');
}
function closeSolution() {
  document.getElementById('solutionModal').classList.remove('open');
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = '#1e293b';
  for (let x = 20; x < canvas.width; x += 28) {
    for (let y = 20; y < canvas.height; y += 28) {
      ctx.fillRect(x, y, 2, 2);
    }
  }

  wires.forEach(w => {
    const a = nodes.find(n => n.id === w.from);
    const b = nodes.find(n => n.id === w.to);
    if (!a || !b) return;
    const high = a.val === 1;
    ctx.strokeStyle = high ? '#22c55e' : '#475569';
    ctx.lineWidth = high ? 3.5 : 2;
    ctx.beginPath();
    ctx.moveTo(a.x, a.y);
    ctx.lineTo(b.x, b.y);
    ctx.stroke();

    if (high) {
      const t = pulseOffset % 1;
      const px = a.x + (b.x - a.x) * t;
      const py = a.y + (b.y - a.y) * t;
      ctx.fillStyle = '#86efac';
      ctx.beginPath();
      ctx.arc(px, py, 4.5, 0, Math.PI * 2);
      ctx.fill();
    }
  });

  nodes.forEach(n => {
    const isHigh = n.val === 1;
    if (selectedSource && selectedSource.id === n.id) {
      ctx.strokeStyle = '#facc15';
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.arc(n.x, n.y, 25, 0, Math.PI * 2);
      ctx.stroke();
    }

    ctx.fillStyle = n.type === 'output'
      ? (isHigh ? '#16a34a' : '#334155')
      : n.type === 'input'
        ? (isHigh ? '#0284c7' : '#1e293b')
        : '#6d28d9';
    ctx.strokeStyle = isHigh ? '#4ade80' : '#94a3b8';
    ctx.lineWidth = 2;

    ctx.beginPath();
    ctx.arc(n.x, n.y, 20, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();

    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 10.5px monospace';
    ctx.textAlign = 'center';
    ctx.fillText(n.type === 'input' || n.type === 'output' ? String(n.val) : n.label, n.x, n.y + 4);

    ctx.fillStyle = '#e2e8f0';
    ctx.font = 'bold 11px sans-serif';
    ctx.fillText(n.label, n.x, n.y - 26);
  });
  ctx.textAlign = 'left';
}

function loop() {
  pulseOffset += 0.02;
  draw();
  requestAnimationFrame(loop);
}

loadLevel(1);
loop();
</script>
</body>
</html>
```