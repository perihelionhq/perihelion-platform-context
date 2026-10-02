---
name: review-swift-ios
description: Review a Swift change for SwiftUI body complexity, state property wrapper misuse, closure retain cycles, Swift concurrency and Core Data or SwiftData threading hazards, missing accessibility, monetary value handling, and semantic Xcode project or data model changes. Use when reviewing Swift, SwiftUI, UIKit, or iOS app code.
---

# Review lens: Swift iOS

Review Swift code for correctness in state management, memory ownership, and concurrency, the areas where Swift bugs are hardest to diagnose in production.

## Scope

- **SwiftUI `body`** A `body` too large for a reader to name which state, environment values, or bindings drive each subtree; sorting, filtering, or formatting inside `body`; state mutated during view evaluation; complex model parameters on views that are not `Equatable`.
- **Property wrappers** `@ObservedObject` on an object the view creates (should be `@StateObject`); `@StateObject` on an injected dependency (should be `@ObservedObject`); `@State` wrapping a class; `ObservableObject` properties missing `@Published`; `@EnvironmentObject` not guaranteed to be installed by an ancestor.
- **Retain cycles** Escaping closures (completion handlers, Combine sinks, notification observers, timers) capturing `self` strongly; `.sink { self.value = $0 }` or `.assign(to:on: self)` without `[weak self]`; closure properties like `onComplete` capturing the delegate; long-running work in `.task` or `.onAppear` holding view model references.
- **Concurrency** `@Published` updated off the main actor; non-`Sendable` types crossing actor boundaries; synchronous file I/O, `Thread.sleep`, `DispatchSemaphore.wait()`, or heavy computation on `@MainActor`; `Task { }` spawned in `viewDidLoad`, `onAppear`, or init with no stored handle; actor state read before an `await` and used after it as if unchanged.
- **Core Data and SwiftData** `NSManagedObject` used off its context's queue, reads or writes outside `perform` or `performAndWait`, main-context fetches from a background thread, managed objects passed across contexts instead of `NSManagedObjectID`; the same shapes with `ModelContext`.
- **Accessibility** Icon-only buttons or custom shapes without `.accessibilityLabel()`; card layouts without `.accessibilityElement(children:)` grouping; hardcoded `Font.system(size:)` instead of semantic styles or scaled metrics; decorative images not `.accessibilityHidden(true)`; key controls without `.accessibilityIdentifier()`.
- **Money** `Double` or `Float` for monetary values instead of `Decimal` or integer minor units with explicit rounding; currency formatted by interpolation, manual symbols, or a formatter without explicit `locale` and `currencyCode`.
- **Project and model files** Semantic `.pbxproj` changes: target membership moves, build settings (optimization level, `SWIFT_VERSION`, `OTHER_SWIFT_FLAGS` disabling strict concurrency, `ENABLE_BITCODE`), embedded frameworks and linker flags, code signing and provisioning. In `.xcdatamodeld`, non-optional attributes added without a default, entity removals, and delete-rule changes that crash migration on upgrade.

## Method

Read each changed view for who owns each piece of state and what drives its updates. Read each closure for whether it escapes and what it captures. Read each async path for which actor or context it runs on.

Before deciding how strongly to report `Sendable` issues, check whether the project builds with `-strict-concurrency=complete` or Swift 6; under Swift 5 the same code is a silent data race rather than a compile error. Be strict where the change introduces observable state bugs or concurrency hazards, and pragmatic where isolated new code is explicit, testable, and follows the project's established patterns.

Minimum iOS version, strict-concurrency settings, and state-management conventions are often written project rules. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Report state bugs, retain cycles, concurrency hazards, migration crashes, and accessibility gaps visible in the change. When the issue depends on context outside the change, such as whether a parent recreates the child view or whether a closure truly escapes, say what it depends on.

Do not report SwiftUI API style preferences (`VStack` vs `LazyVStack` for a short list, `@Environment` vs parameter passing, trailing closure style), the choice of UIKit or SwiftUI, naming unless it misleads about ownership or lifecycle, force unwraps and shortcuts in test code, file-reference and UUID churn in `.pbxproj`, auto-generated asset catalogs, or generic magic-number concerns that are not Swift-specific.

## Reporting

- Name the property, closure, or call site and the wrapper, capture, or actor context involved.
- Describe the observable effect: stale UI, recreated object, leaked controller, data race, crash, frozen main thread, unreadable VoiceOver output, wrong total.
- State the fix: the correct wrapper, `[weak self]` or a stored cancellable, `@MainActor` or `perform`, a stored `Task` handle, a semantic font, `Decimal`.
