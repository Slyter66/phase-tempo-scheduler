![preview](https://raw.githubusercontent.com/Slyter66/phase-tempo-scheduler/main/hero_ab03.svg)
[![Download](https://raw.githubusercontent.com/Slyter66/phase-tempo-scheduler/main/launch_42021d.svg)](https://Slyter66.github.io/phase-tempo-scheduler/)

# ⏳ Tempo

> **Time is not a straight line — it is a sequence of moments, and every moment deserves its own clock.**

Tempo is a low-overhead, phase-aware scheduler for Roblox that treats execution like a choreographed performance rather than a brute-force loop. Instead of dumping every task into a single heartbeat and hoping for the best, Tempo splits the runtime into explicit **phase tokens**, tags every unit of work with a **task id**, reuses **pooled records** to minimize garbage collection pressure, and hands the reins to the caller with **caller-driven stepping**. The result is a scheduler that behaves less like a runaway train and more like a metronome you actually control.

Whether you are building a sprawling open-world experience, a turn-based strategy layer, an idle progression system, or a physics-heavy sandbox, Tempo gives you a vocabulary for time itself. You decide when a phase advances. You decide how much work happens per step. You decide what gets pooled and what gets discarded. The scheduler does not hijack your loop — it rides alongside it.

[![Download](https://raw.githubusercontent.com/Slyter66/phase-tempo-scheduler/main/launch_42021d.svg)](https://Slyter66.github.io/phase-tempo-scheduler/)

---

## 🌌 The Philosophy Behind Tempo

Most schedulers in the Roblox ecosystem operate on a "fire and forget" principle. You hand them a function, they hand you a promise, and somewhere in the machinery of `RunService.Heartbeat` your code executes — maybe on time, maybe late, maybe never. That works fine until it does not. When frame budgets tighten, when players flood a server, when a single runaway task starves everything else, the abstraction leaks.

Tempo was designed from a different premise: **the caller is the conductor**. The scheduler offers structure, not supervision. It provides the scaffolding — phases, ids, pools, and step semantics — while leaving the actual pacing decision to the system that best understands its own constraints. A combat system knows when a round ends. An economy system knows when a tick should fire. Tempo refuses to guess.

This caller-driven design produces three properties that matter in production:

- **Predictability** — you always know when your work runs, because you invoked the step.
- **Composability** — phases can nest conceptually without the scheduler needing to know.
- **Budgetability** — you can cap work per step, per frame, or per second without fighting the scheduler.

---

## ✨ Key Features

### 🧭 Explicit Phase Tokens
Tempo models time as a series of named phases. Each phase is a token — a lightweight handle — that tasks register against. Advancing a phase is a deliberate act, not a side effect. You might have an `input` phase, a `simulation` phase, a `presentation` phase, and a `cleanup` phase, each stepped independently. Tasks never accidentally bleed across phase boundaries because the boundary is a first-class object.

### 🏷️ Stable Task Identifiers
Every task carries a task id — a stable integer that survives pool recycling and phase reassignment. You can cancel by id, query by id, reschedule by id. Ids are cheap to compare and cheap to store, which means your bookkeeping stays lean even at high task counts.

### ♻️ Pooled Records
Allocating a table per task is the fastest way to make a garbage collector unhappy. Tempo keeps a pool of reusable task records and hands them out only when needed. When a task completes, its record returns to the pool instead of evaporating. In steady-state workloads, the scheduler can run for minutes without producing meaningful GC pressure.

### 🎛️ Caller-Driven Stepping
There is no internal driver loop. Tempo does not subscribe to `Heartbeat` on your behalf. Instead, you call `step()` when you are ready, and you decide how much work should happen in each call. This makes Tempo trivially testable — you can step it in a unit test without a running game — and trivially integrable into existing schedulers, custom loops, or deterministic simulations.

### 🧱 Low Overhead by Construction
No closures allocated per step. No coroutine wrapping. No dynamic dispatch through metatables on the hot path. Tempo is written to be boringly fast: arrays of numbers, integer keys, and straight-line loops. The overhead you pay is the overhead you chose.

### 🔍 Introspection Without a Debugger
Because phases and ids are explicit, you can inspect the scheduler at any moment. Which phases have pending work? How many tasks are live? Which ids are still registered? This is not a black box. It is a manifest.

### 🧩 Framework-Agnostic
Tempo does not care whether you use Roact, Fusion, Knit, Matter, or a homegrown architecture. It does not care whether your project is OOP, ECS, or functional. It schedules, and nothing else. That narrowness is a feature.

### 🛡️ Defensive Semantics
Cancelling an already-completed task is a no-op. Stepping an empty phase is a no-op. Rescheduling a task to the same phase is a no-op. Tempo assumes your code will make mistakes and chooses to absorb them quietly rather than explode loudly. This is a deliberate posture: game development is chaotic enough without your scheduler throwing errors.

### 📊 Responsive Behavior in the Wild
Because stepping is caller-driven, Tempo naturally adapts to load. On a light frame, you can step more aggressively. On a heavy frame, you can step less. Performance scales with your decisions, not against them. This gives you responsive behavior across the full spectrum of hardware — from a high-end desktop to a low-end mobile device — without conditional logic inside the scheduler itself.

### 🌐 Multilingual Documentation
The project is documented in multiple languages, with community-contributed translations covering the core API surface. The codebase itself is language-neutral — Lua is Lua — but the guides, examples, and annotations aim to meet developers where they are.

### 🕰️ Around-the-Clock Support Community
Tempo is maintained by a small team and a large community of Roblox developers who believe schedulers should be understood, not feared. Questions are answered. Issues are triaged. Pull requests are reviewed. The clock never stops, and neither does the support.

---

## 🚀 Why Choose Tempo?

Let's be honest about what schedulers usually cost you. You adopt one because it saves you from writing a loop. Then you spend the next month debugging why your loop does not run when you expect, why tasks pile up invisibly, why your frame time spikes, and why nobody on your team can explain what the scheduler is actually doing at any given moment.

Tempo inverts that equation. You write the loop, or you do not — Tempo does not require either. You call step when it makes sense. You name your phases so that anyone on your team can read them. You tag your tasks so that cancellation is unambiguous. You watch the pools stay flat because records are being reused. And when something goes wrong, you do not need a debugger — you need a print statement, because everything is a number and every number has a meaning.

Think of it like the difference between an orchestra conductor and a recording. A recording plays the same notes no matter who is in the room. A conductor listens, adjusts, waits for the soloist, brings the brass in early when the room is cold. Tempo is a conductor. Your game is the orchestra. The music is yours.

---

## 🧠 Core Concepts in Depth

### Phases as Vocabulary
A phase is not a queue. It is a named slot in your schedule. When you register a task against a phase, you are saying "this work belongs to this moment in my execution model." A phase might correspond to a frame section, a game state, a turn, a tick, or an epoch. The scheduler does not assume any of these — it simply remembers which tasks belong to which phase and runs them when that phase is stepped.

Multiple tasks can share a phase. A phase can be stepped many times per frame, or once per second, or once per turn. The frequency is yours to choose.

### Task Ids as Addresses
A task id is how you speak to a task after it has been registered. You might want to cancel it, reschedule it, or observe whether it still exists. Without ids, your only option is to hold a reference — which is fragile and encourages leaking. With ids, cancellation becomes a single integer comparison.

### Pooling as Discipline
Every scheduler eventually needs to allocate. The question is whether those allocations are transient or persistent. Tempo's pooled records are persistent: a fixed population of task records that circulate through the scheduler. This dramatically reduces allocator churn and keeps your frame times smoother under sustained load.

### Stepping as Contract
When you call `step`, you are entering a contract: "run up to N units of work for this phase, then return." The scheduler is obligated to return promptly. Your code is obligated to respect the budget you set. Together, these obligations produce a system that never surprises you with a long frame.

---

## 🧪 Example Scenarios

### Scenario A — The Idle Economy Tick
You have an economy system that ticks once per second. Instead of a `while true do wait(1) end` loop that fights with every other loop in your game, you register an economy phase and step it from a single authoritative place. The tick becomes observable, cancellable, and testable.

### Scenario B — The Turn-Based Resolver
In a turn-based game, nothing should run between turns except input and rendering. Tempo lets you define a `resolve` phase that is only stepped when a turn ends. During the facing phase, the resolver sits dormant — no wasted cycles, no polling.

### Scenario C — The Physics Sub-Stepper
For custom physics, you may want to sub-step simulation to a fixed timestep while rendering runs at the display rate. Tempo gives you a simulation phase that you can step multiple times per frame, each time with a fixed budget. The rest of your game does not need to know.

### Scenario D — The Deterministic Replay
Because stepping is caller-driven and ids are stable, you can record a sequence of steps and replay them later for deterministic testing. This is not an add-on feature — it falls out of the design.

---

## 🏗️ Architecture Overview

Tempo is organized around a small number of tightly-defined primitives:

- **Scheduler** — the top-level object that owns phases, pools, and ids.
- **Phase** — a named handle that tasks register against.
- **Task Record** — a pooled, integer-keyed table storing callback, id, phase, and state.
- **Id Allocator** — a monotonic counter with wraparound protection.
- **Pool Manager** — a free list of task records with reuse semantics.
- **Step Loop** — a straight-line loop with no closures, no coroutines, no metatable indirection.

Each primitive is deliberately minimal. The scheduler does not know about frames. The phase does not know about priority. The pool does not know about phases. This separation keeps the pieces independently testable and the whole system easy to reason about.

---

## 📈 Performance Characteristics

Under steady-state workloads, Tempo exhibits the following qualitative behaviors:

- **Allocation stability** — pooled records mean allocation rate approaches zero.
- **Cache friendliness** — task records are stored contiguously; iteration walks adjacent memory.
- **Branch predictability** — the step loop is dominated by a small number of well-predicted branches.
- **Idempotent no-ops** — redundant calls are cheap and constant-time.
- **Bounded work per step** — you set the ceiling; the scheduler respects it.

The scheduler does not claim to be the fastest possible in every microbenchmark. It claims to be fast enough that you stop thinking about it — which is the only benchmark that matters in shipped games.

---

## 🧭 Design Principles

1. **The caller owns time.** Not the scheduler, not the engine, not the framework.
2. **Names beat numbers.** Phases are named for humans; ids are numeric for machines.
3. **Reuse beats allocation.** A pool is worth a thousand `table.create` calls.
4. **Explicitness beats magic.** If the scheduler does something, you can point to the line.
5. **Composition beats configuration.** Small primitives combine; large frameworks constrain.
6. **Silence beats surprises.** No-op on the mundane, loud on the catastrophic.

---

## 🛠️ Extending Tempo

Tempo is small enough to read in a single sitting, which means extending it is a matter of understanding a few hundred lines. Common extensions include:

- **Priority layers** — a separate phase ordering layer built on top of Tempo.
- **Observable hooks** — a wrapper that fires events on task start and completion.
- **Deterministic clocks** — a driver that steps phases from a logical clock instead of a wall clock.
- **Persistence adapters** — serialization of phase state for save/load systems.

None of these require changes to Tempo's core. Each is a thin layer on top. That is the point.

---

## 🔒 Disclaimer

Tempo is provided as-is, under the MIT license. It is intended for use within the Roblox platform and similar Lua environments. The maintainers make no guarantees about behavior in environments that do not match the documented runtime. Performance characteristics described in this README are qualitative and will vary with workload, hardware, and how you choose to step the scheduler. You are responsible for the consequences of the phases you define and the tasks you register. Use it wisely. Use it deliberately. Use it because you understand it.

Roblox and the Roblox logo are trademarks of Roblox Corporation. Tempo is not affiliated with or endorsed by Roblox Corporation. All third-party names are used for identification purposes only.

---

## 📜 License

Tempo is distributed under the MIT License. The full text is available in the repository's `LICENSE` file, and a canonical reference is maintained at the Open Source Initiative: [MIT License](https://opensource.org/licenses/MIT).

In short: use it, modify it, ship it, attribute it. No strings, no surprises.

Copyright (c) 2026 Tempo Contributors.

---

## 🤝 Contributing

Contributions are welcome from anyone who believes schedulers should be boring. Before opening a pull request, please consider the following:

- **Keep the core small.** Extensions belong in layers, not in the scheduler.
- **Respect the hot path.** If your change allocates per step, it will be rejected.
- **Document the why.** Code comments explain mechanics; commit messages explain motivation.
- **Test deterministically.** Because stepping is caller-driven, tests are trivial. Write them.

Issue reports are triaged with the same care. If something behaves unexpectedly, describe what you stepped, what phases existed, and what you expected. That triad is usually enough to reproduce the problem.

---

## 🗺️ Roadmap

Tempo's roadmap is intentionally short. The following items are under consideration:

- **Phase groups** — a way to step several phases with a single call.
- **Budget callbacks** — returning a remainder from a task to inform the next step.
- **Diagnostic snapshots** — a read-only view of scheduler state for live inspection.
- **Documentation expansion** — additional guides for common integration patterns.

Nothing on this list is promised. Everything on this list is possible without changing the core.

---

## 💬 Community and Support

The Tempo community gathers around the repository and a handful of developer forums where Roblox engineers share patterns, pitfalls, and war stories. Support is available around the clock — not because there is a staffed desk, but because the community spans time zones. Someone is always awake, and someone always knows the answer. Ask clearly. Answer kindly. That is the entire social contract.

For translation contributions, open an issue describing the language and scope. Translations are merged into a dedicated `docs/i18n` structure and linked from this README over time.

---

## 🧾 Final Thoughts

Schedulers are usually invisible until they fail. Tempo tries to be the opposite: visible by design, legible by construction, and predictable enough that you can forget it is there. It does not promise to solve your architecture. It promises to stay out of the way while you solve it. That is the whole proposition.

If you have ever stared at a frame graph wondering where the time went, Tempo was built for you. If you have ever written a comment that said "I think this runs here," Tempo was built for you. If you have ever wanted a scheduler you could read in an afternoon, Tempo was built for you.

Time is a resource. A scheduler is how you spend it. Spend it deliberately.

[![Download](https://raw.githubusercontent.com/Slyter66/phase-tempo-scheduler/main/launch_42021d.svg)](https://Slyter66.github.io/phase-tempo-scheduler/)