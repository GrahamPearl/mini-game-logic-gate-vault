# Generation Instructions: *LogicGate: Vault Breaker v3* (Offline Classroom & Assessment Edition)

Use this specification to generate or extend **LogicGate: Vault Breaker v3** as a single-file, zero-dependency `.html` browser game designed for **Grade 8–9 learners** on an **isolated internal school network (strictly no internet)**.

---

## 1. Curriculum & Pedagogical Alignment
* **South African Coding & Robotics (CAPS Senior Phase):** Focuses on Boolean logic (`AND`, `OR`, `NOT`), interpreting sensor/actuator inputs (`0`/`1`), reading truth tables, and solving constrained control problems.
* **Cambridge Computer Science (IGCSE / O-Level):** Covers the full 6-gate set (`AND`, `OR`, `NOT`, `XOR`, `NAND`, `NOR`), $2^n$ truth tables, live Boolean expression readouts, and universal gate synthesis (e.g., constructing `OR` or `XOR` when specific gates are disabled).

---

## 2. Strict Offline Guarantee & Modular Single-File Architecture
Everything must reside inside **one standalone `.html` file** (vanilla HTML, CSS, and JavaScript—zero external CDNs, fonts, images, or cloud APIs):

1. **`OfflineGuard` Module:**
   * Intercepts and blocks any external `http://` / `https://` `fetch` calls at runtime so browsers on air-gapped LANs never hang.
2. **`ArtRegistry` Module:**
   * Supports optional local file uploads (`FileReader` Data URLs) for the vault canvas background and node sprites (`input`, `output`, `gate`) with automatic fallback to crisp canvas vector rendering.
3. **`LEVELS` & `GATE_TYPES` Schema Module (6 Scaffolded Levels):**
   * Each level defines: `id`, `tier` (1–3), `curriculumTag`, `title`, `briefing`, `baseScore`, `parSeconds`, `maxGates`, `disabledGates` (array of banned gate types), `targetExpression`, `targetOutputs` (all $2^n$ binary truth-table outputs), `nodes`, and `solutionSteps`.
   * **Scaffolded Progression:**
     * *Tier 1 (Levels 1–2):* 2 inputs, core gates (`AND`, `OR`, `NOT`), `100–150` base points.
     * *Tier 2 (Levels 3–4):* 3 inputs ($2^3 = 8$ rows), compound gates (`XOR`, `NAND`, `NOR`), `200–250` base points.
     * *Tier 3 (Levels 5–6):* Gate-restriction synthesis challenges (e.g., building `OR` using only `NAND`, or building parity with `XOR` disabled), `300–350` base points.
4. **`TruthEngine` Module:**
   * Evaluates cascaded circuits across all $2^n$ binary input combinations, displays live Boolean expressions alongside the goal expression, supports click-to-apply rows and automated truth-table sweeps, and highlights the exact row where verification fails.
5. **`PracticeGenerator` Module:**
   * Generates randomized, guaranteed-solvable practice vaults across Tier 1, Tier 2, or Tier 3 with appropriate gate restrictions without inflating official assessment scores.
6. **`StudentSession` & `ExportManager` Modules (Collection Setup C):**
   * Requires a **5–6 digit Admin Number** (`/^\d{5,6}$/`), Full Name, and Class Group.
   * Calculates level score from `baseScore`, unused gate efficiency bonus (`+10`/gate), failed attempt penalty (`-5`/attempt), overtime decay, and walkthrough penalty (`-50%` if **Show Solution** was opened).
   * Exports offline `.json` receipts, `.csv` markbook rows, and copyable receipt strings.
7. **`TeacherHub` Module (PIN-Protected):**
   * Includes a **Formal Assessment / Test Mode** toggle that disables **Show Solution**, **Practice Vault**, and **Level Editor** during formal tests and stamps `"assessmentMode": true` onto student exports.
   * Imports multiple student `.json` files (or pasted JSON receipts), ranks learners by Admin Number in a central high-score table, and exports a consolidated **Class Markbook `.csv`**.
8. **`LevelEditor` Module:**
   * Lets teachers customize level titles, base scores, par times, gate budgets, `disabledGates` checkboxes, and `targetOutputs`, and copy the resulting JSON schema.