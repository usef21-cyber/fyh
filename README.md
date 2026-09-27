# THE LAST TOMORROW — Engineering Scaffold

This is a real Visual Studio solution implementing the foundation of the spec in
`docs/SPEC.md` (your original master spec), built in the order the spec itself mandates
(section 8): **Architecture → Domain → Persistence → Integrity → Tests**, before any UI
polish, animation, or art.

This was scaffolded in a Linux sandbox with no network access, so **it has not been
compiled or run**. The code was written carefully and cross-checked by hand (every
interface method matches every implementation, every constructor call matches its
signature — see `docs/ARCHITECTURE.md` → "How this was verified without a compiler"),
but treat the first build in Visual Studio as the real first test.

## Prerequisites

- Windows 10 (19041+) or Windows 11
- Visual Studio 2022, version 17.9 or later
- Workloads: **.NET Desktop Development** and **Windows App SDK C# Templates**
  (Visual Studio Installer → Individual Components → search "Windows App SDK")
- .NET 8 SDK (installed automatically by the workload above, or via https://dotnet.microsoft.com)

## Opening and building

1. Open `TheLastTomorrow.sln` in Visual Studio.
2. Let NuGet restore run (needs internet access — this is the step that couldn't happen
   in the sandbox this was built in).
3. Set `TheLastTomorrow.UI` as the startup project, platform `x64`.
4. Build → Run (F5).

On first launch, the app creates its SQLite database at
`%LOCALAPPDATA%\TheLastTomorrow\citadel.db` and runs migrations automatically before any
window opens.

## Running the tests

```
dotnet test tests\TheLastTomorrow.Domain.Tests
dotnet test tests\TheLastTomorrow.Application.Tests
dotnet test tests\TheLastTomorrow.Infrastructure.Tests
```

The Domain and Application test projects are pure .NET 8 (no Windows dependency) and
should run anywhere the .NET 8 SDK is installed, including CI on Linux. The
Infrastructure tests spin up real temporary SQLite databases to prove the schema
constraints actually hold — they also don't need Windows.

## What's implemented

| Layer | Status |
|---|---|
| Domain (entities, value objects, state machines, pure rule services) | Complete for the core loop |
| Application (use cases: CompleteTask, CreateStake, ResolveStake, DailyProcessing, RewardPresentation) | Complete for the core loop |
| Infrastructure (SQLite schema + migrations, repositories, Time Service, JSON rules, CSPRNG, file logger) | Complete for the core loop |
| UI (WinUI 3) | Minimal working shell: one screen, task list, Complete button, reward/status readout |
| Boss system, World Zones, Rank ladder narrative events, Revival Challenge UI, Localization, Asset/Audio systems | **Not built yet** — see "What's next" |

"Core loop" means: complete a task → XP, vitals, an optional Stake settles, THE LEVER
rolls and commits a reward, rank may advance, everything is audited — and Daily
Processing safely catches up on any days the app was closed, exactly once per day, with
clock-tamper detection. This is deliberately where the effort went first, because the
spec's own acceptance criteria (section 7.4) rank Data Integrity, Correct Game Logic and
Crash Recovery above UI Polish and Animation Polish.

## What's next

In the order the spec's own build order would continue:

1. **Wire the remaining use cases into the UI** — CreateStake/ResolveStake have no XAML
   yet (Application logic is done; only the "Oath" screen is missing).
2. **Boss Campaigns** — no Domain entity yet. Sketch: a `BossCampaign` entity with a
   target/progress and a link from `TaskItem`/`HabitDefinition`, following the exact same
   pattern as Stake (its own state machine, its own resolution use case).
3. **World Zones** — `HabitDefinition.ZoneKey` already exists as a hook; needs a
   `ZoneState` entity (Locked/Restored/Corrupted per spec 2.9) and the calculation that
   derives zone state from habit consistency.
4. **Revival Challenge** — `PlayerState.Hearts` and `IsTodayLost()` already exist; needs
   the "spend 45 minutes on your hardest task to recover a heart" use case and UI.
5. **Rank promotion narrative events** (the "لم تعد المشكلة..." message at Sergeant, etc.)
   — hook these off `AuditEventType.RankAdvanced`, which is already raised.
6. **Asset/Audio systems** (spec section 6) — entirely unstarted; correctly last, per the
   spec's own priority order.
7. **Reconciliation Mode** (spec section 4.27) — `DomainInvariantViolationException` is
   thrown everywhere it should be; nothing yet catches it at the top of the Application
   layer and routes into a recovery screen instead of crashing. This should be one of the
   next things built, since it's Data Integrity, not a feature.

See `docs/ARCHITECTURE.md` for the reasoning behind each layer and a list of the smaller,
explicitly-flagged gaps left for a real build to close.
