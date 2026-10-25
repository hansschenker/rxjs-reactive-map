# Reactive Engine Map vs. RxJS Semantics

This note isolates the central comparison between the historical Reactive Engine architecture and the modern semantic view of RxJS.

## 1. Historical Reactive Engine

```text
Event List
    │
    ▼
Current Event
    │
    ▼
Process Data / State
    │
    ▼
Process Description
    │
    ▼
Generated Event
```

The historical engine already separates:

- what happened,
- which process is affected,
- what state the process retains,
- what rule executes,
- what new event/action follows.

That is the skeleton of reactive execution.

## 2. Modern RxJS Semantic View

```text
Event Queue
    │
    ▼
Current Notification
    │
    ▼
Subscription Graph
    │
    ▼
Operator / Subscription State
    │
    ▼
Operator Behavior
    │
    ▼
Downstream Notification
```

The important modern addition is the explicit **subscription graph**.

RxJS execution is not adequately explained by a conventional JavaScript call stack alone. The meaningful reactive trace is often the path a notification follows through the dataflow.

## 3. Correspondence Table

| Historical Reactive Engine | RxJS Semantics |
|---|---|
| Event List | Event Queue |
| Current Event | Current notification/event |
| Process references | Subscription graph edges |
| Process Data | Operator/subscription state |
| Process Stack / Context | Reactive execution context |
| Process Description | Operator semantics |
| Process Activation | Notification propagation |
| Generated Event | Downstream/internal event |

This is a structural comparison, not a claim of identical implementation.

## 4. The Three Runtime Dimensions

### Time / Work

```text
Event Queue
```

Answers:

> What event is waiting or being processed?

### Topology

```text
Subscription Graph
```

Answers:

> Where can this notification travel?

### State / Behavior

```text
Operator Machine
```

Answers:

> What does this operator remember, and what does it do next?

These dimensions are orthogonal and should be analyzed separately.

## 5. Three Different "Stacks"

The word *stack* can hide three distinct concepts.

### JavaScript Call Stack

```text
main
 ↓
foo
 ↓
bar
```

Question:

> Which function called this function?

### Reactive Dataflow Path

```text
source
 ↓
map
 ↓
filter
 ↓
switchMap
 ↓
observer
```

Question:

> Through which reactive nodes did this notification arrive?

### Operator State

```text
switchMap:
    activeInner
    outerCompleted
```

Question:

> What does this machine currently remember?

For RxJS semantics and debugging, the second and third are often more informative than the first.

## 6. Mealy Interpretation

Each operator node can be analyzed as:

```text
              input
                │
                ▼
        ┌───────────────┐
state → │ Operator      │ → next state
        │ T + G         │
        └───────┬───────┘
                │
                ▼
             action
```

with:

```text
M = (S, S₀, Σ, Γ, T, G)
```

where:

- `S` — operator state
- `S₀` — initial state
- `Σ` — input notifications/events
- `Γ` — output notifications/actions
- `T` — transition function
- `G` — Action Policy

## 7. BDD Front-End

The machine can be rendered in readable form:

```text
Given <state>
When  <event>
Then  <state transition>
And   <action>
```

Example:

```text
Given an inner subscription is active
When a new outer value arrives
Then cancel the active inner
And subscribe to the new inner
And emit nothing immediately
```

That is a readable description of the `switchMap` machine.

## 8. Final Synthesis

```text
Reactive Engine
      │
      ▼
Event Queue / Time
      │
      ▼
Subscription Graph / Topology
      │
      ▼
Operator Machine / State
      │
      ▼
Transition T + Action Policy G
      │
      ▼
BDD Description
      │
      ▼
RxJS Implementation
```

The historical Reactive Engine gives the architecture.

Modern RxJS semantics gives the runtime topology.

The Mealy model gives the formal behavior.

BDD gives the human-readable specification.

RxJS gives the compositional implementation.

Together they provide a unified way to explain, analyze, test, and teach RxJS.
