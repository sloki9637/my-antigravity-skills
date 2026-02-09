---
trigger: always_on
---

# Flutter + Provider Coding Rules

> These rules are **mandatory**.
> Do not infer intent. Follow exactly.

---

## 1. Responsibility Separation

- Providers **MUST** handle:
  - State
  - Business logic

- Providers **MUST NOT**:
  - Access `BuildContext`
  - Control navigation
  - Show dialogs/snackbars
  - Contain UI logic

---

## 2. Directory Structure

```txt
lib/
  models/
  providers/
  screens/
  widgets/
```

Rules:

- All `ChangeNotifier` classes **MUST** be in `providers/`
- UI widgets **MUST NOT** be placed in `providers/`
- Data models **MUST NOT** depend on Providers

---

## 3. Provider Naming

- Provider class names **MUST** end with `Provider`
- Names **MUST** describe the managed state

Examples:

- ✅ `UserProvider`
- ❌ `CommonProvider`
- ❌ `DataProvider`

---

## 4. State Encapsulation

- All state fields **MUST** be `private`
- Public access **MUST** be through `getter` only
- Public setters are **FORBIDDEN**

```dart
class CounterProvider extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
}
```

---

## 5. State Mutation Rules

- State **MUST** be modified only through methods
- Each method **MUST** call `notifyListeners()` at most once
- `notifyListeners()` **MUST** be called immediately after state changes

---

## 6. notifyListeners Usage

- Multiple `notifyListeners()` calls in a single method are **FORBIDDEN**
- Calling `notifyListeners()` without state change is **FORBIDDEN**

---

## 7. UI Access Rules

### Allowed

- `context.read<T>()` for actions
- `context.watch<T>()` for reactive values
- `Consumer<T>` for localized rebuilds

### Forbidden

- Watching providers at the screen root without reason
- Calling provider constructors in UI

---

## 8. Provider Lifecycle Scope

- Global providers **MUST** be created in `main.dart`
- Screen-scoped providers **MUST** be created in the screen entry widget
- Providers **MUST NOT** outlive their usage scope

---

## 9. Async Logic Pattern

- Async methods **MUST**:
  - Manage loading state internally
  - Use `try / finally`

- Loading state **MUST** be exposed via getter

```dart
Future<void> fetchData() async {
  _loading = true;
  notifyListeners();
  try {
    _data = await api.fetch();
  } finally {
    _loading = false;
    notifyListeners();
  }
}
```

---

## 10. Provider Instantiation

- Providers **MUST NOT** be instantiated manually in UI code
- Providers **MUST** be obtained via `Provider.of`, `read`, or `watch`

---

## 11. Rebuild Performance Rules

- `watch` usage **MUST** be minimal
- UI **MUST** isolate rebuilds using `Consumer` or child widgets
- Whole-screen rebuilds are **FORBIDDEN** unless explicitly required

---

## 12. Rule Priority

1. Correctness
2. Readability
3. Performance
4. Convenience (lowest priority)

---

## 13. Enforcement

- Any code violating these rules **MUST** be rejected
- No exceptions unless explicitly documented
