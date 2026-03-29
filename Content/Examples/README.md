# FlowGraph Example Flows

This directory contains documentation for three minimal example `UFlowAsset` flows.  
The `.uasset` files must be created manually in the Unreal Editor following the instructions
below — binary assets cannot be authored in source control directly.

---

## Table of Contents

1. [FA_HelloWorld](#fa_helloworld) — Start → Log → Finish  
2. [FA_BranchExample](#fa_branchexample) — Conditional branching  
3. [FA_TimerLoop](#fa_timerloop) — Timer-driven counter loop  
4. [Creating a Flow Asset](#creating-a-flow-asset)  
5. [Node Reference](#node-reference)

---

## FA_HelloWorld

**File**: `Content/Examples/FA_HelloWorld.uasset`  
**Concept**: The absolute minimum flow — a single execution path with a log message.

### What It Demonstrates
- How every flow starts with `FlowNode_Start` and ends with `FlowNode_Finish`
- How to use `FlowNode_Log` to emit a message during execution
- The basic pin-connection model (Out → In)

### Node Layout

```
┌─────────┐   Out      In  ┌────────────────────┐   Out      In  ┌────────┐
│  Start  │ ──────────────▶│  Log               │ ──────────────▶│ Finish │
└─────────┘                │  Message: "Hello!" │                └────────┘
                           │  Verbosity: Display│
                           │  PrintToScreen: ✓  │
                           └────────────────────┘
```

### Step-by-Step Setup

1. Create a new `FlowAsset` (see [Creating a Flow Asset](#creating-a-flow-asset)).
2. The graph opens with `Start` and `Finish` nodes already placed.
3. **Add a Log node**: Right-click the graph → search *Log* → select **Log**.
4. **Connect Start → Log**: Drag from `Start.Out` to `Log.In`.
5. **Configure Log**:
   - `Message` = `"Hello, FlowGraph!"`
   - `Verbosity` = `Display`
   - `bPrintToScreen` = `true`
   - `Duration` = `3.0`
6. **Connect Log → Finish**: Drag from `Log.Out` to `Finish.In`.
7. **Compile & Save**.

### Expected Behaviour

When the flow is triggered, `"Hello, FlowGraph!"` appears on screen for 3 seconds and in
`Output Log` at `Display` verbosity. The flow then finishes immediately.

---

## FA_BranchExample

**File**: `Content/Examples/FA_BranchExample.uasset`  
**Concept**: Demonstrates conditional routing via `FlowNode_Branch` and a predicate AddOn.

### What It Demonstrates
- How `FlowNode_Branch` evaluates AddOn predicates to pick `True` or `False` outputs
- How to add a `FlowNodeAddOn` predicate to a Branch node
- Diverging paths that both reconverge at `Finish`

### Node Layout

```
┌─────────┐   Out    In  ┌──────────────────┐
│  Start  │ ──────────▶  │  Branch          │
└─────────┘              │  [AddOn: Check   │──── True ────▶ ┌────────────────┐   Out ──▶ ┌────────┐
                         │   bTestValue]    │                │ Log "True!"    │           │ Finish │
                         └──────────────────┘                └────────────────┘           └────────┘
                                           │
                                           └─── False ───▶ ┌────────────────┐   Out ──▶ ┌────────┐
                                                            │ Log "False!"   │           │ Finish │
                                                            └────────────────┘           └────────┘
```

> Note: A single `Finish` node can receive both paths — connect both Log nodes to the same
> `Finish.In` pin.

### Step-by-Step Setup

1. Create a new `FlowAsset`.
2. **Add a Branch node**: Right-click → search *Branch* → select **Branch**.
3. **Connect Start.Out → Branch.In (Evaluate)**.
4. **Add a predicate AddOn to Branch**:
   - Select the Branch node → Details panel → **AddOns** → `+` → choose
     `FlowNodeAddOn_PredicateAND` (or any available predicate AddOn).
   - Inside the AddOn, set `bTestValue = true` to force the `True` path, or `false` for
     `False`. *(Replace with a gameplay-driven predicate in real usage.)*
5. **Add two Log nodes** — one for each output:
   - Log A: `Message = "Branch result: TRUE"`, connect `Branch.True → LogA.In`.
   - Log B: `Message = "Branch result: FALSE"`, connect `Branch.False → LogB.In`.
6. **Connect both logs to Finish**: `LogA.Out → Finish.In`, `LogB.Out → Finish.In`.
7. **Compile & Save**.

### Expected Behaviour

- With predicate returning `true` → Log A fires, prints `"Branch result: TRUE"`.
- With predicate returning `false` → Log B fires, prints `"Branch result: FALSE"`.
- Either path reaches `Finish` and the flow ends.

---

## FA_TimerLoop

**File**: `Content/Examples/FA_TimerLoop.uasset`  
**Concept**: A timer-driven loop that increments a counter and exits after N iterations.

### What It Demonstrates
- How `FlowNode_Timer` introduces a time delay between steps
- How `FlowNode_Counter` counts loop iterations and fires when its goal is reached
- How to wire a feedback loop using `Branch` (or Counter's built-in `Completed` pin)

### Node Layout

```
┌─────────┐  Out   In  ┌──────────────────┐  Out    In  ┌─────────────────────┐
│  Start  │──────────▶ │  Timer           │────────────▶│  Counter            │
└─────────┘            │  CompletionTime  │             │  Goal: 5            │
                       │    = 2.0s        │ ◀─┐         │                     │
                       └──────────────────┘   │         │ Completed ──────────┼──▶ ┌────────┐
                                              │         │                     │    │ Finish │
                                              │         │ Out (not-completed) │    └────────┘
                                              └─────────┴─────────────────────┘
                                               (loop back to Timer)
```

> `Counter.Out` fires on every activation that has NOT yet reached Goal.  
> `Counter.Completed` fires on the activation that reaches Goal.

### Step-by-Step Setup

1. Create a new `FlowAsset`.
2. **Add a Timer node**: Right-click → search *Timer* → select **Timer**.
   - Set `CompletionTime = 2.0`.
   - Leave `StepTime = 0` (no intermediate steps needed).
3. **Add a Counter node**: Right-click → search *Counter* → select **Counter**.
   - Set `Goal = 5`.
4. **Wire the initial path**: `Start.Out → Timer.In`.
5. **Wire Timer completion to Counter**: `Timer.Out → Counter.In`.
6. **Wire the loop-back**: `Counter.Out → Timer.In`.  
   *(This reconnects when the goal is not yet reached.)*
7. **Wire the exit**: `Counter.Completed → Finish.In`.
8. **Add a Log node** (optional, for observability):
   - Insert between `Timer.Out` and `Counter.In`.
   - `Message = "Loop tick"`, `bPrintToScreen = true`, `Duration = 1.5`.
9. **Compile & Save**.

### Expected Behaviour

| Time  | Event |
|-------|-------|
| 0 s   | Flow starts, Timer begins |
| 2 s   | Timer completes → Counter hit 1/5 → loops back |
| 4 s   | Timer completes → Counter hit 2/5 → loops back |
| 6 s   | Timer completes → Counter hit 3/5 → loops back |
| 8 s   | Timer completes → Counter hit 4/5 → loops back |
| 10 s  | Timer completes → Counter hit 5/5 → `Completed` fires → Finish |

Total runtime: ~10 seconds. Each loop tick prints `"Loop tick"` on screen (if Log added).

---

## Creating a Flow Asset

### In the Content Browser

1. **Right-click** in the Content Browser at `Content/Examples/`.
2. Navigate to **Miscellaneous → Flow Asset** (or search *Flow Asset*).
3. Name the asset (e.g. `FA_HelloWorld`).
4. **Double-click** to open the Flow Graph editor.

### Editor Layout

The Flow Graph editor has three areas:
- **Graph viewport** — the main canvas where nodes are placed and connected
- **Details panel** — shows properties of the selected node
- **Palette / search** — right-click anywhere in the graph to open the node search popup

### Adding Nodes

1. **Right-click** anywhere on the graph canvas.
2. Type the node name in the search box (e.g. `Log`, `Timer`, `Counter`).
3. Click the result to place it.

### Connecting Pins

- **Drag from an output pin** (right side of a node) **to an input pin** (left side) to create a
  connection.
- Output pins are white circles; input pins are white triangles (or named pins like `True`/`False`).
- A pin can accept multiple connections — both a `True` and a `False` path can connect to the same
  `Finish.In`.

### Compiling and Testing

1. Click **Compile** in the toolbar (or press `F7`).
2. Errors appear as red nodes — hover for details.
3. To test: place a `UFlowComponent` on any Actor in your level, assign the Flow Asset, then
   PIE. The flow triggers when the component is activated.

---

## Node Reference

| Node | Category | Key Properties | Output Pins |
|------|----------|----------------|-------------|
| `FlowNode_Start` | Graph | — | `Out` |
| `FlowNode_Finish` | Graph | `FinishPolicy` (Keep/Destroy) | — |
| `FlowNode_Log` | Developer | `Message`, `Verbosity`, `bPrintToScreen`, `Duration` | `Out` |
| `FlowNode_Branch` | Route | `BranchCombinationRule` (AND/OR), AddOn predicates | `True`, `False` |
| `FlowNode_Timer` | Route | `CompletionTime` (float, seconds), `StepTime` | `Out` (on step), `Out` (on complete) |
| `FlowNode_Counter` | Route | `Goal` (int, ≥2) | `Out` (not-completed), `Completed` |
| `FlowNode_Switch` | Route | `Keys` (array of FName) | one pin per key + `Default` |
| `FlowNode_ExecutionSequence` | Route | `NumberOfPins` | `0`, `1`, `2`, … |
| `FlowNode_SubGraph` | Graph | `Asset` (nested `UFlowAsset`) | `Out` |
| `FlowNode_Checkpoint` | Graph | `SaveGame` flag | `Out` |

For actor-interaction nodes (`ComponentObserver`, `NotifyActor`, `PlayLevelSequence`) see
`Source/Flow/Public/Nodes/Actor/`.

---

*All `.uasset` files in this directory must be created via the Unreal Editor.*  
*Source: `Plugins/FlowGraph/Content/Examples/` — FlowGraph plugin v2.2*
