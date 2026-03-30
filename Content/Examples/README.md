# FlowGraph — Example Flows

This directory contains example Flow Assets demonstrating core FlowGraph concepts.
Each example teaches one fundamental pattern.

## What is FlowGraph?

FlowGraph is a visual node-based scripting system for Unreal Engine 5. It provides
an event-driven, async execution model where **nodes** represent self-contained gameplay
features connected by **pins** that define execution flow.

Unlike Blueprints (which model per-frame logic), FlowGraph models **sequences of events**
— making it ideal for narrative triggers, cutscenes, boss phases, and level scripting.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Flow Asset** | A `UFlowAsset` — the container graph. Create via *Content Browser → Miscellaneous → Flow Asset*. |
| **Node** | A `UFlowNode` — each node encapsulates a feature (log, timer, gate, actor notify, etc.). |
| **Input Pin** | Receives execution from upstream nodes. A node activates when any input fires. |
| **Output Pin** | Sends execution downstream. A node can have multiple named outputs for branching. |
| **Flow Subsystem** | `UFlowSubsystem` — the runtime that manages active flow instances. |

### How Execution Works

1. `StartFlow()` is called on a Flow Asset instance
2. The **Start** node fires, triggering its output pin
3. Connected downstream nodes activate in sequence
4. Nodes with async behavior (Timer, actor observers) wait for their condition
5. The **Finish** node terminates all active nodes and the flow instance

---

## Node Reference

### Graph Structure

| Node | Purpose |
|------|---------|
| **Start** | Entry point. One per graph. Fires immediately on `StartFlow()`. |
| **Finish** | Terminates the flow. Deactivates all active nodes and sub-graphs. |
| **SubGraph** | Executes another Flow Asset as a child graph. |
| **Custom Input / Custom Output** | Define named entry/exit points for sub-graph communication. |

### Routing & Logic

| Node | Purpose | Key Properties |
|------|---------|----------------|
| **Execution Sequence** | Fires outputs in order (0, 1, 2…). | — |
| **Multi Gate** | Fires one output per trigger, cycling through them. | `bRandom`, `bLoop`, `StartIndex` |
| **Timer** | Fires after elapsed time. Optional repeating steps. | `CompletionTime`, `StepTime` |
| **Counter** | Fires output once input has been triggered N times. | `Goal` (target count) |
| **AND** | Fires output only after ALL inputs have triggered. | — |
| **OR** | Fires output when ANY input triggers. | `ExecutionLimit` |
| **Switch** | Multi-way conditional routing. | — |

### Actor Interaction

| Node | Purpose |
|------|---------|
| **Notify Actor** | Send a named notification to a registered actor. |
| **On Notify From Actor** | Wait for a notification from a specific actor. |
| **On Actor Registered / Unregistered** | React to actors joining/leaving the flow system. |
| **Execute Component** | Trigger a `UFlowComponent` on an actor. |
| **Play Level Sequence** | Play a cinematic sequence. |

### Developer / Debug

| Node | Purpose | Key Properties |
|------|---------|----------------|
| **Log** | Print a message to console/screen. | `Message`, `Verbosity`, `bPrintToScreen`, `Duration`, `TextColor` |
| **Format Text** | String formatting with parameters. | — |

---

## Example Flows

> **Note:** Flow Assets (`.uasset`) must be created in the Unreal Editor.
> Below are detailed descriptions of each example's node graph for manual recreation.

### 1. FA_HelloWorld — Minimal Flow

**Concept:** Simplest possible flow — entry, action, exit.

```
┌─────────┐     ┌──────────────────────┐     ┌──────────┐
│  Start   │────▶│  Log                 │────▶│  Finish  │
│          │     │  Message: "Hello!"   │     │          │
└─────────┘     │  Verbosity: Display  │     └──────────┘
                │  PrintToScreen: true │
                │  Duration: 5.0       │
                └──────────────────────┘
```

**How to recreate:**
1. Content Browser → Right-click → Miscellaneous → Flow Asset → name it `FA_HelloWorld`
2. Double-click to open the Flow Editor
3. The **Start** node is placed automatically
4. Right-click canvas → add **Log** node → set `Message` to `"Hello World!"`
5. Connect Start output → Log input
6. Add **Finish** node → connect Log output → Finish input
7. Save

### 2. FA_MultiGateDemo — Sequential Routing

**Concept:** Multi Gate fires one output per trigger, cycling through them in order.

```
                              ┌────────────────────┐     ┌──────────┐
                         0 ──▶│ Log "First time!"  │────▶│  Finish  │
┌─────────┐  ┌───────────┐   ├────────────────────┤     └──────────┘
│  Start   │─▶│ Multi Gate│1─▶│ Log "Second time!" │────▶│  Finish  │
└─────────┘  │ bLoop:true│   ├────────────────────┤     └──────────┘
             └───────────┘2─▶│ Log "Third time!"  │────▶│  Finish  │
                              └────────────────────┘     └──────────┘
```

Each time the flow runs, the Multi Gate fires the **next** output in sequence.
With `bLoop: true`, it wraps back to output 0 after the last output.

**How to recreate:**
1. Create Flow Asset `FA_MultiGateDemo`
2. Add **Multi Gate** node with 3 outputs, set `bLoop: true`
3. Connect Start → Multi Gate
4. Add 3 **Log** nodes with messages "First time!", "Second time!", "Third time!"
5. Connect each Multi Gate output (0, 1, 2) to its respective Log node
6. Connect each Log → Finish

### 3. FA_TimerLoop — Timed Execution with Steps

**Concept:** Timer node fires "Step" output at regular intervals, then "Completed" when done.

```
┌─────────┐     ┌──────────────────────┐
│  Start   │────▶│  Timer               │
└─────────┘     │  CompletionTime: 10s │
                │  StepTime: 2s        │
                └──┬───────────┬───────┘
                   │ Step      │ Completed
                   ▼           ▼
        ┌──────────────┐  ┌──────────────────┐  ┌──────────┐
        │ Log          │  │ Log              │─▶│  Finish  │
        │ "Tick!"      │  │ "Timer done!"    │  └──────────┘
        │ Screen: true │  └──────────────────┘
        └──────────────┘
```

The Timer fires "Step" every 2 seconds (5 times total), then fires "Completed" at 10 seconds.
This pattern is useful for wave spawning, countdowns, or periodic checks.

**How to recreate:**
1. Create Flow Asset `FA_TimerLoop`
2. Add **Timer** node → set `CompletionTime: 10.0`, `StepTime: 2.0`
3. Connect Start → Timer
4. Add **Log** "Tick!" → connect Timer Step output → Log input
5. Add **Log** "Timer complete!" → connect Timer Completed output → Log → Finish

### 4. FA_ActorNotify — Actor Communication

**Concept:** Flow graphs communicate with world actors via the notify system.

```
┌─────────┐     ┌───────────────────────┐     ┌────────────────────────┐     ┌──────────┐
│  Start   │────▶│ On Actor Registered   │────▶│ Notify Actor           │────▶│  Finish  │
│          │     │ (waits for actor      │     │ Identity: "MyDoor"     │     │          │
└─────────┘     │  with FlowComponent)  │     │ Notification: "Open"   │     └──────────┘
                └───────────────────────┘     └────────────────────────┘
```

Actors participate in FlowGraph by having a `UFlowComponent`. The component registers
the actor with an **Identity Tag** that nodes reference. Notify Actor sends a named
signal that the actor's FlowComponent receives and can respond to in Blueprint.

**How to recreate:**
1. Create Flow Asset `FA_ActorNotify`
2. Add **On Actor Registered** node (waits for a tagged actor to register)
3. Add **Notify Actor** node → set Identity Tag and notification name
4. Wire: Start → On Actor Registered → Notify Actor → Finish
5. In your level, add a `UFlowComponent` to an actor and set its Identity Tag

---

## Creating New Flow Assets

### Step-by-Step

1. **Content Browser** → Right-click → **Miscellaneous** → **Flow Asset**
2. Name your asset (convention: `FA_` prefix, e.g., `FA_BossPhase1`)
3. Double-click to open the **Flow Editor**
4. The **Start** node is auto-placed — this is your entry point
5. Right-click the canvas to add nodes from the palette
6. Drag from output pins to input pins to create connections
7. Always end with a **Finish** node to properly terminate the flow

### Running a Flow at Runtime

Flows execute through the `UFlowSubsystem` (a World Subsystem). To trigger a flow:

- Place a `UFlowComponent` on an actor and assign your Flow Asset
- Or call `UFlowSubsystem::StartRootFlow()` from C++ / Blueprint

### Tips

- Nodes are **async by default** — Timer, actor observers, etc. pause execution until their condition is met
- Multiple output pins = multiple parallel execution paths
- Use **SubGraph** nodes to break complex flows into reusable pieces
- The **AND** gate is useful for "wait for all conditions" patterns
- `SaveGame` properties on nodes mean flow state survives save/load

---

## Further Reading

- [FlowGraph Wiki — Getting Started](https://github.com/MothCocoon/FlowGraph/wiki/Getting-Started)
- [FlowGraph Wiki — Design Philosophy](https://github.com/MothCocoon/FlowGraph/wiki)
- Plugin source: `Plugins/FlowGraph/Source/Flow/Public/Nodes/`
