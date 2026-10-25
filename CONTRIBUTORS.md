# Contributors

## hansschenker

Project owner and originator of the RxJS analysis direction that connects:

- Reactive Engine architecture
- Event-Driven Dataflow
- RxJS operator behavior
- BDD `Given / When / Then`
- Mealy state-machine analysis
- Action Policy
- operator-level semantics and teaching models

## GPT-5.6 Sol (OpenAI)

Contributed:

- in-depth analysis of the historical Reactive Engine map,
- comparison with modern RxJS runtime semantics,
- separation of Event Queue, Subscription Graph, and Operator State,
- distinction between JavaScript call stacks and reactive dataflow paths,
- mapping of RxJS operators to Mealy machines,
- synthesis of the Reactive Engine → RxJS Semantics → BDD → Mealy framework,
- examples for `switchMap`, `debounceTime`, `scan`, and stateful operators,
- documentation structure and repository text.

The central joint result is the model:

```text
Event Queue
    ↓
Subscription Graph
    ↓
Operator Machine
    ↓
State Transition + Action Policy
    ↓
Output Notification
```

and the explanatory pipeline:

```text
Reactive Engine
    ↓
Runtime/Dataflow Structure
    ↓
BDD Behavior
    ↓
Mealy Formalization
    ↓
RxJS Implementation
```
