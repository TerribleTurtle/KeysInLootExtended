# Architectural Discoveries (Disk Brain)

## Architecture & Initialization
*   **Initialization Hook:** The mod hooks into SPT using `IOnLoad` decorated with `[Injectable(TypePriority = OnLoadOrder.PostDBModLoader + 1)]`.
*   **Metadata Record:** Requires a manifest file using a record struct inheriting `AbstractModMetadata` (defines `ModGuid`, `Name`, `Version`, `SptVersion`).
*   **Dependency Injection:** Constructors in the hook class automatically receive injected services from SPT (e.g., `DatabaseServer`, `ISptLogger`, `FileSystem`).

## Configuration & Data Handling
*   **JSONC Parsing:** Configuration uses `.jsonc`. C# implementation MUST use `Newtonsoft.Json` (or `System.Text.Json` with `JsonCommentHandling.Skip`) to parse comments natively.
*   **Null-Coalescing Safety:** User-provided JSONC files may contain malformed or explicit `null` overrides (e.g. `"jacketContainer": null`). To prevent raw C# `NullReferenceException`s during object instantiation (where a serializer overrides `new()` defaults with `null`), use the null-coalescing operator `??` to gracefully fall back to global config weights.
*   **Fail-Loudly Principles:** Misconfigurations or broken type checks must explicitly log and fail loudly instead of being swallowed.

## Core Loot Engine & Memory Optimizations
*   **Static Loot Manipulation:** Accessed via `DatabaseServer.GetTables().Locations[...].StaticLoot`.
*   **Dynamic Modded Maps:** Reflection (`db.Locations.GetType().GetProperties()`) must be used to dynamically fetch maps rather than relying on hardcoded lists, guaranteeing compatibility with custom user maps.
*   **Shared Array References (CRITICAL):** SPT heavily shares `ItemDistribution` array references across different maps to save memory. Mutating an array element in one map (e.g. Customs) will unintentionally affect other maps (e.g. Woods). When applying map-specific overrides (`locations/*.jsonc`), the mod must clone the reference (e.g. `Select(...)`) to ensure configurations remain isolated.
*   **Blast Radius (Int Overflow):** SPT uses standard 32-bit integers for relative probabilities. The sum of all container weights MUST be clamped to prevent exceeding `Int32.MaxValue` (`2,147,483,647`), which triggers an integer overflow and completely breaks map loot generation.

## Item Template & Database Manipulation
*   **Memory Optimizations (GC Pressure):** Looping over lists inside generic enumerations forces C# to allocate a closure Lambda per execution, heavily impacting GC during server boot. **Always convert lists to pre-computed `HashSet<T>` and execute O(1) memory-safe `Contains` operations.**
*   **Economy Adjustments:** Flea Prices accessed via `Templates.Prices`. Trader Assorts via `Traders[traderId].Assort`. Adjustments MUST use O(1) hash sets of injected keys to limit blast radius.
*   **MongoId Parsing Crashes:** Items added by other mods might contain invalid `MongoId` formats, causing server crashes when iterating arrays. The C# port must wrap `new MongoId(item.Id)` in a try-catch block for `FormatException` to skip malformed items and prevent server halts.
*   **Serialization & Testing:** Standard JSON deserialization fails on `MongoId` arrays. For isolated O(1) testing without heavy Server environments, mock properties via reflection using `System.Runtime.CompilerServices.RuntimeHelpers.GetUninitializedObject`.
