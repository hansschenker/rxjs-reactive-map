# RxJS Reactive Map

A runtime-oriented explanation of RxJS that connects three complementary views:

1. the early **Reactive Engine** architecture,
2. the modern **Semantics of RxJS** runtime model,
3. the **BDD → Mealy → RxJS** operator-analysis method.

The central conclusion is:

> **RxJS can be understood as an Event-Driven Dataflow Engine in which notifications travel through a subscription graph and are processed by small stateful operator machines.**

This repository preserves the analysis that led to that model and turns it into a reusable framework for teaching, specification, debugging, and operator analysis.

---

## 1. The Reactive Engine Map

The historical Reactive Engine can be reduced to this execution model:

```text
Event List
    │
    ▼
Current Event
    │
    ▼
Process State / Data
    │
    ▼
Process Description
    │
    ▼
Generated Event(s)
    │
    └────────────→ Event List
```

Its important ideas are already recognizably reactive:

- events are first-class execution triggers,
- processes retain state between events,
- dependencies decide which processes react,
- process logic is separated from process state,
- one reaction can produce further events,
- execution therefore propagates through a dependency network.

The important shift from ordinary imperative control flow is:

```text
Imperative:
instruction → instruction → instruction

Reactive:
event → dependent process → event → dependent process
```

This is an early form of **Event-Driven Dataflow**.

---

## 2. Mapping the Reactive Engine to RxJS

A useful conceptual mapping is:

| Reactive Engine | RxJS |
|---|---|
| Event List | source/internal notifications over time |
| Current Event | current `next`, `error`, `complete`, or internal event |
| Process Data | operator state |
| Process Stack / Context | retained reactive execution context |
| Process Description | operator behavior + user function |
| Dependency | subscription/dataflow connection |
| Process Activation | notification delivery |
| Generated Event | downstream notification |
| Reactive Engine | RxJS execution runtime |

This is a conceptual correspondence, not an implementation identity.

The historical engine tells us **where the moving parts are**.

---

## 3. RxJS Semantics: Event Queue + Subscription Graph

The modern formal view of RxJS makes the runtime topology more explicit.

A useful abstraction is:

```text
Event Sources
     │
     ▼
Event Queue
     │
     ▼
Current Event
     │
     ▼
Subscription Graph
     │
     ▼
Operator Machine
     │
     ▼
Downstream Notification
```

The subscription graph is essential because a normal JavaScript call stack answers:

> Which function called this function?

but RxJS often requires a different question:

> Through which reactive nodes did this notification travel?

For RxJS debugging and explanation, the meaningful trace is frequently:

```text
source
  ↓
map
  ↓
filter
  ↓
switchMap
  ↓
inner source
  ↓
observer
```

That is a **dataflow path**, not merely a JavaScript call stack.

---

## 4. Three Orthogonal Runtime Structures

The comparison reveals three different structures that should not be conflated.

### 4.1 Event Queue — work over time

Answers:

> What event is waiting to be processed?

```text
A
B
C
D
```

### 4.2 Subscription Graph — dataflow topology

Answers:

> Where can this notification travel?

```text
source
  ↓
operator
  ↓
operator
  ↓
observer
```

### 4.3 Operator State — machine memory

Answers:

> What does this operator remember?

Examples:

```text
distinctUntilChanged:
    previousValue

scan:
    accumulator

debounceTime:
    pendingValue
    timer

switchMap:
    activeInner
    outerCompleted
```

These can be summarized as:

```text
TIME / WORK      → Event Queue
TOPOLOGY         → Subscription Graph
STATE / BEHAVIOR → Operator Machine
```

---

## 5. The Operator as a Small Reactive Machine

Once a notification reaches an operator node, the operator can be analyzed as a state machine.

```text
                    input event
                        │
                        ▼
                ┌───────────────┐
     state ─────→│ Operator Node │────→ next state
                │               │
                │ Transition T  │
                │ Action G      │
                └───────┬───────┘
                        │
                        ▼
                 output action
```

For our RxJS analysis method:

```text
S   = operator state
S₀  = initial state
Σ   = input events / notifications
Γ   = output actions / notifications
T   = transition function
G   = Action Policy
```

The machine can therefore be represented as:

```text
M = (S, S₀, Σ, Γ, T, G)
```

For RxJS, `Σ` and `Γ` should include lifecycle and internal events where relevant:

```text
Input:
    Next<T>
    Error<E>
    Complete
    timer events
    inner notifications
    cancellation-related events

Output / Actions:
    Next<R>
    Error<E>
    Complete
    NoOutput
    Subscribe
    Unsubscribe / Cancel
    Schedule
```

---

## 6. BDD as the Human-Readable Machine Description

The Reactive Engine map gives the structural categories:

```text
current event
process state
process description
resulting action
```

BDD gives us a compact readable notation for the same machine:

```text
Given <current state>
When  <event arrives>
Then  <state transition>
And   <action>
```

Example for `distinctUntilChanged`:

```text
Given previous value is 17
When value 42 arrives
Then remember 42
And emit 42
```

Example for a repeated value:

```text
Given previous value is 42
When value 42 arrives
Then keep the state
And emit nothing
```

This makes BDD a readable front-end to the Mealy model.

---

## 7. The Complete Analysis Pipeline

The combined framework is:

```text
                  RxJS ANALYSIS

               Reactive Engine View
                       │
                Event Queue / Time
                       │
                       ▼
                Dataflow Graph
                       │
               Subscription Path
                       │
                       ▼
               Operator Machine
                       │
                 S, S₀, Σ, Γ
                    T, G
                       │
                       ▼
                 BDD Behavior
                       │
                Given / When
                 Then / And
                       │
                       ▼
               RxJS Implementation
```

Each layer answers a different question:

| Layer | Question |
|---|---|
| Reactive Engine | Where are the moving runtime parts? |
| Event Queue | What happens next in time? |
| Subscription Graph | Where can the notification travel? |
| Operator Machine | What state and behavior does this node have? |
| BDD | How can a human read the behavior? |
| Mealy Model | What is the formal machine? |
| RxJS | How is the behavior implemented and composed? |

---

## 8. Why This Helps with RxJS Operators

For every operator we can now ask the same questions:

1. What notifications/events can arrive?
2. What state is remembered?
3. What changes on each event?
4. What is emitted?
5. What is suppressed?
6. What subscriptions/resources are created?
7. What gets cancelled?
8. What happens on `error`?
9. What happens on `complete`?
10. Where does the resulting notification travel next?

This gives one common framework for query, time, stateful, combination, and higher-order operators.

---

## 9. Example: switchMap

`switchMap` is not best explained as “async mapping”.

Its runtime machine is:

```text
State:
    activeInner
    outerCompleted
```

Behavior:

```text
Given an inner subscription is active
When a new outer value arrives
Then cancel the active inner
And subscribe to the new inner
And emit nothing immediately
```

Its concurrency policy is therefore:

> **only latest**

The same engine framework exposes the other flattening policies:

```text
mergeMap  → set of active inners        → allow overlap
concatMap → active inner + queue        → queue while busy
switchMap → zero/one active inner       → only latest
exhaustMap→ zero/one active + busy rule → ignore while busy
```

---

## 10. Example: debounceTime

```text
State:
    pendingValue
    timerActive
```

BDD form:

```text
Given a silence timer is active
When a source value arrives
Then replace the pending value
And restart the timer
And emit nothing
```

```text
Given a pending value exists
When the silence timer expires
Then emit the pending value
And clear the timer state
```

Again the Reactive Engine categories appear directly:

```text
current event
+ stored process state
+ process description
→ resulting action
```

---

## 11. Definition Phase vs Execution Phase

RxJS adds an important abstraction to the older Reactive Engine model.

```text
Definition Phase

Observable.pipe(...)
        │
        ▼
dataflow blueprint
```

Nothing necessarily executes yet.

Then:

```text
Execution Phase

subscribe()
    │
    ▼
subscription graph created
    │
    ▼
notifications propagate
```

This means an Observable is a **description of a dataflow** and execution begins on subscription for ordinary cold Observables.

---

## 12. Final Model

The most useful compact runtime model is:

```text
             EVENT SOURCE
                  │
                  ▼
            EVENT QUEUE
                  │
             current event
                  │
                  ▼
        SUBSCRIPTION GRAPH
                  │
            dataflow path
                  │
                  ▼
          OPERATOR MACHINE
          ┌──────────────┐
          │ state S      │
          │ transition T │
          │ action G     │
          └──────┬───────┘
                 │
                 ▼
        OUTPUT NOTIFICATION
                 │
                 ▼
        next graph node / event
```

This leads to the repository's core statement:

> **RxJS is a compositional Event-Driven Dataflow Engine. An Observable describes values and lifecycle notifications moving over time. Subscription instantiates the dataflow. Notifications propagate through a subscription graph. Each operator behaves like a small machine that may retain state, applies a transition rule, and follows an Action Policy that determines what is forwarded, suppressed, scheduled, subscribed, or cancelled.**

The domain can change.

The RxJS machine stays the same.

---

## Sources and Inspiration

- Evgeny Poberezkin, *What is Reactive Programming?* — historical Reactive Engine discussion.
- Tian Zhao and Yonglun Li, *Semantics of RxJS* — formal runtime treatment of event queues, subscription/dataflow graphs, and reactive stack traces.
- RxJS 7.x source and semantics.

---

## Contributors

- **hansschenker** — project owner, RxJS analysis direction, BDD/Mealy/Reactive Map synthesis.
- **GPT-5.6 Sol (OpenAI)** — analysis, comparison of the historical Reactive Engine with RxJS semantics, and synthesis of the Reactive Engine → Subscription Graph → Mealy → BDD framework.

See [CONTRIBUTORS.md](./CONTRIBUTORS.md).
