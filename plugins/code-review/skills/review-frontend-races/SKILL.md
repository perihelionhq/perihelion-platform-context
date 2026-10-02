---
name: review-frontend-races
description: Review a code change for frontend timing and lifecycle bugs, including listeners, timers, observers and async work that outlive their component, missing effect cleanup, Turbo/Stimulus/React lifecycle mistakes, overlapping interactions, and stale promise or animation work. Use when reviewing frontend code for race conditions, cleanup, or UI state consistency.
---

# Review lens: Frontend Races

Review frontend code for timing and cleanup: stale timers, duplicate async work, handlers firing on dead nodes, and UI state that cannot represent what is actually happening.

## Scope

- **Lifecycle cleanup gaps** Event listeners, timers, intervals, observers, or async work that outlive the DOM node, controller, or component that started them.
- **React effect exit paths** When a change moves where a component mounts, alters cleanup, or touches a third-party script or global, every `useEffect` exit path, including "already loaded" guards and early returns after a `window` or global mutation, script injection, listener, timer, or DOM append.
- **Turbo, Stimulus and React timing** State created in the wrong lifecycle hook, code that assumes a node stays mounted, and async callbacks that mutate the DOM after a swap, remount, or disconnect.
- **Concurrent interactions** Operations that can overlap when they should be mutually exclusive, repeated triggers that overwrite one another without cancelation, and boolean flags that cannot represent the true UI state where explicit state constants (for example `Symbol()` values) and a transition function would.
- **Stale promise and timer work** Missing `finally()` cleanup, unhandled rejections, overwritten timeouts that are never canceled, and animation loops that keep running after the UI has moved on.
- **Risk-multiplying event wiring** Per-element handlers or DOM wiring that raise the chance of leaks, duplicate triggers, or inconsistent teardown where one delegated listener would be safer.

## Method

Assume the DOM is reactive and slightly hostile: nodes are swapped, remounted, and disconnected while work is in flight.

For each listener, timer, observer, script, or async call the change starts, find where it is stopped. For each changed `useEffect`, list the mutations performed before each return and confirm a matching cleanup exists on that path.

Ask what happens when the user triggers the same interaction twice, or a second one before the first finishes.

Whether a framework lifecycle or state pattern is mandated is often a written project rule rather than a judgment call. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Report races that can be constructed or traced from the code: a `setInterval` with no `clearInterval` on disconnect, async work scheduled after teardown, or a second interaction that can obviously start before the first ends. Report a race that depends on runtime timing only when the code clearly lacks the guard that would prevent it.

Do not report stylistic DOM preferences, animation taste unless it causes a real timing or replacement bug, framework choice by itself, or speculation that amounts to frontend superstition.

## Reporting

- Name the resource or operation, where it starts, and the path on which it is never stopped or guarded.
- Describe the interleaving that triggers the bug and what the user sees when it happens.
- State the smallest fix: the missing cleanup, cancelation, or explicit state transition. Prefer a few lines of local code over a new dependency; understand the race before picking a tool to remove it.
