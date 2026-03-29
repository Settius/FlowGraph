# FlowGraph — Example Asset Setup Guide

Technical reference for creating, configuring, and testing `UFlowAsset` instances in the editor.

---

## Prerequisites

- Unreal Engine 5.x with the **Flow** plugin enabled (`Flow.uplugin` → `"Enabled": true` in
  your project's `.uproject` or via the Plugin Manager).
- A level with at least one Actor that can host a `UFlowComponent`.

## Enabling the Plugin

In `YourProject.uproject` (or via **Edit → Plugins**):

```json
{
  "Name": "Flow",
  "Enabled": true
}
```

---

## Step 1 — Create a FlowAsset

| Action | Where |
|--------|-------|
| Right-click in Content Browser | `Content/Examples/` |
| Choose | **Miscellaneous → Flow Asset** |
| Name it | `FA_HelloWorld` (or any `FA_` prefixed name) |

Double-click the new asset to open the **Flow Graph editor**.

---

## Step 2 — Graph Editor Overview

```
┌──────────────────────────────────────────────────────────┐
│  Toolbar:  [Compile ▶]  [Save 💾]  [Find 🔍]            │
├────────────────────────┬─────────────────────────────────┤
│                        │                                 │
│   Graph Canvas         │   Details Panel                 │
│   (nodes + wires)      │   (selected node properties)    │
│                        │                                 │
└────────────────────────┴─────────────────────────────────┘
```

The canvas opens with **Start** and **Finish** already placed.

---

## Step 3 — Adding Nodes

1. **Right-click** anywhere on the canvas → node search popup appears.
2. Type the node name (e.g. `log`, `timer`, `branch`).
3. Click a result to place it at your cursor position.

**Keyboard shortcut**: hold `Ctrl` while right-clicking to bypass the search and get a
filtered context menu.

---

## Step 4 — Connecting Pins

| Action | How |
|--------|-----|
| Connect two nodes | Drag from an **output pin** (right side) to an **input pin** (left side) |
| Disconnect a wire | Alt-click on a pin, or right-click the wire → **Break Link** |
| Fan-out (one output → many inputs) | Drag from the same output pin to multiple input pins |
| Fan-in (many outputs → one input) | Connect multiple wires to a single input pin |

Output pins = **circles** on the right edge of a node.  
Input pins = **triangles** on the left edge of a node.  
Named pins (e.g. `True`, `False`, `Completed`) appear as labelled rows on the node.

---

## Step 5 — Configuring Node Properties

1. **Select** a node by clicking it.
2. The **Details panel** (right side) shows its UPROPERTY fields.
3. Edit values directly — changes take effect after the next compile.

Common properties:

| Node | Property | Type | Notes |
|------|----------|------|-------|
| Log | `Message` | FString | Static text shown in log |
| Log | `Verbosity` | Enum | `Display` shows in Output Log + screen |
| Log | `bPrintToScreen` | bool | Overlays message on viewport |
| Timer | `CompletionTime` | float | Seconds before `Out` fires |
| Timer | `StepTime` | float | Interval for intermediate `Out` pulses (0 = off) |
| Counter | `Goal` | int32 | Min 2; `Completed` fires on this activation count |
| Branch | AddOns | array | Add predicate AddOns to control `True`/`False` routing |

---

## Step 6 — Compile

Click **Compile** in the toolbar (or `F7`).

- **Green checkmark** = no errors, asset is ready.
- **Red nodes** = compile error — hover the node for the message.
- Unconnected required pins do NOT block compilation but will warn at runtime.

---

## Step 7 — Running a Flow in PIE

### Option A — Via FlowComponent on an Actor

1. Select any Actor in the level (or create an empty one).
2. **Add Component** → search *Flow* → add **FlowComponent**.
3. In the FlowComponent Details:
   - `FlowAsset` = your new asset (e.g. `FA_HelloWorld`).
   - `AutoStartMode` = `OnBeginPlay` to trigger immediately.
4. Press **Play (PIE)** — the flow executes automatically.

### Option B — Via Blueprint call

```cpp
// In any Blueprint or C++ that has a reference to the FlowComponent:
FlowComponent->StartFlow();
```

### Observing Execution

- **Output Log** (`Window → Output Log`): shows `Display`/`Warning`/`Error` messages.
- **Flow Debugger** (`Window → Flow Debugger`): shows active nodes highlighted in the graph
  in real time during PIE.

---

## Common Pitfalls

| Symptom | Cause | Fix |
|---------|-------|-----|
| Flow never starts | `AutoStartMode` not set | Set to `OnBeginPlay` or call `StartFlow()` manually |
| Branch always takes `False` | No predicate AddOn added | Add an AddOn to the Branch node |
| Timer fires instantly | `CompletionTime = 0` | Set to a value > 0 (e.g. `2.0`) |
| Counter never reaches `Completed` | `Goal` is too high or loop not wired back | Check loop-back wire from `Counter.Out` to `Timer.In` |
| Log not visible on screen | `bPrintToScreen = false` | Enable in the Log node Details |
| "Uninitialized Flow" warning | FlowAsset not assigned to FlowComponent | Assign the asset in the Details panel |

---

## Saving and Version Control

- Flow assets are binary `.uasset` files — **do not hand-edit them**.
- Commit via normal Unreal source control (Perforce/Git LFS).
- The `Content/Examples/` directory is tracked by this repository for documentation only;
  actual `.uasset` files should be added after editor creation.

---

*FlowGraph plugin v2.2 — `Plugins/FlowGraph/`*
