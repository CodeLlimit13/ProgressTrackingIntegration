# ProgressTrackingIntegration

## Overview
This project is a .NET 9 .NET MAUI application that demonstrates adding a **progress tracking feature** to a user dashboard. The feature surfaces a user's recent lesson progress as interactive button-like items that navigate to a lesson progress page (resume point placeholder).

## Added Feature: Recent Progress Buttons
The dashboard now displays up to three recent lesson progress entries (status + percentage) as tappable cards. Selecting one navigates (via Shell) to a placeholder lesson progress page with the related identifiers passed as query parameters.

## Key Components
| Area | File(s) | Purpose |
|------|---------|---------|
| Data Service | `Services/TestDatabaseService.cs` | Added methods to query progress tracking data. |
| Data Models (DTOs) | `ProgressSummary` (in service file) | Lightweight representation of lesson progress for UI binding. |
| ViewModel | `ViewModels/UserDashboardViewModel.cs` | Added `RecentProgress`, population logic, and navigation command. |
| Dashboard UI | `Views/UserDashboardPage.xaml` | Added CollectionView rendering progress as tappable Border items. |
| Navigation | `AppShell.xaml.cs` | Registered new route `lessonProgress`. |
| Target Page | `Views/LessonProgressPage.xaml(.cs)` | Placeholder page showing received parameters. |
| Project File | `TestDatabaseApp.csproj` | Included XAML page so `InitializeComponent` compiles. |

## Implementation Steps (Repeatable Pattern)
1. Identify dashboard (container) view + its ViewModel.
2. Extend data service with retrieval methods for the new feature (avoid over-fetching; return slim DTOs):
   - `GetUserRecentProgress(userId, limit)`
   - `GetUserLessonProgress(userId, lessonId)` (future resume logic)
3. Create a DTO (`ProgressSummary`) with only fields required by UI (e.g., `LessonId`, `Status`, `ProgressPercent`). Provide computed helpers (`CourseName`, `DisplayText`).
4. In the ViewModel:
   - Add `ObservableCollection<ProgressSummary> RecentProgress`.
   - Populate it in `Initialize()` after existing data loads.
   - Add `OpenProgressCommand` that receives a `ProgressSummary` and performs `Shell.Current.GoToAsync` with query parameters.
5. In the XAML dashboard page:
   - Insert a `CollectionView` bound to `RecentProgress`.
   - Use a `Border` inside the `DataTemplate` to mimic a button (allows richer layout than a simple Button control).
   - Attach a `TapGestureRecognizer` referencing the ViewModel command via relative binding.
6. Register a Shell route for the destination (e.g., `lessonProgress`).
7. Create the destination page that reads query parameters via `[QueryProperty]` attributes.
8. Add the new XAML page to the project file `<MauiXaml>` list if not auto-included (ensures `InitializeComponent` is generated).
9. Build and fix compile issues:
   - Adjust invalid layout elements (removed unsupported `VerticalListSpanLayout`).
   - Correct nullable `DateTime` parsing (cannot use `out DateTime?`).
10. Verify navigation works and bindings resolve at runtime.

## Date/Time Parsing Note
`DateTime.TryParse` does not support `out DateTime?`. Use a temporary local:
```csharp
DateTime? last = null;
if (!reader.IsDBNull(iLast) && DateTime.TryParse(reader.GetString(iLast), out var tmp))
    last = tmp;
```

## Shell Navigation Pattern
```csharp
var route = $"lessonProgress?userId={userId}&lessonId={p.LessonId}&progressId={p.ProgressId}";
await Shell.Current.GoToAsync(route);
```
Target page declares:
```csharp
[QueryProperty(nameof(UserId), "userId")]
[QueryProperty(nameof(LessonId), "lessonId")]
[QueryProperty(nameof(ProgressId), "progressId")]
```

## Reusable Template (Checklist)
- [ ] Add service method returning DTO list
- [ ] Add DTO with computed properties
- [ ] Add `ObservableCollection<T>` to ViewModel
- [ ] Populate collection in initialization lifecycle
- [ ] Add navigation command (with parameter)
- [ ] Modify XAML: `CollectionView` + template + tap gesture
- [ ] Register route in `AppShell`
- [ ] Create destination page (and optional ViewModel)
- [ ] Add page to `<MauiXaml>` in `.csproj` if needed
- [ ] Build + test navigation + binding

## Future Enhancements
- Replace placeholder `CourseName` with actual lesson titles (needs lessons query).
- Add granular resume data (e.g., `LastPosition`, `SectionIndex`).
- Introduce a dedicated `LessonProgressViewModel` and load full lesson content.
- Visual progress bar: bind `ProgressBar.Progress` to `ProgressPercent / 100.0`.
- Filtering / expanding progress history (show more than three items).

## Constraints & Assumptions
- Progress percentage is currently inferred from status (`Completed`=100, `InProgress`=50) or falls back to score if available.
- Lesson content not yet integrated; placeholder page displays identifiers only.

## Tech Stack
- .NET 9
- .NET MAUI (single project)
- SQLite via `Microsoft.Data.Sqlite.Core`

## Quick Start (Developer)
1. Restore & build: `dotnet build` (or build in IDE).
2. Run on desired target (Android/iOS/Windows/MacCatalyst).
3. Navigate to a user dashboard (route `dashboard?userId={id}` assumed from existing navigation flow).
4. Tap a Recent Progress card to see parameter hand-off to `LessonProgressPage`.

## License
(Add license information here if needed.)

---
Generated implementation notes for reuse across future MAUI dashboard + progress integrations.
