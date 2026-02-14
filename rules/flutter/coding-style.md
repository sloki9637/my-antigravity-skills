---
trigger: always_on
---

# Flutter State Management Coding Rules

> These rules are **mandatory**.
> Do not infer intent. Follow exactly.
> **Default: Use Provider unless user specifies Riverpod**

---

## 0. State Management Selection

- **Default**: Provider
- **If user specifies "Riverpod" or "riverpod"**: Use Riverpod rules
- Ask for clarification ONLY if ambiguous

---

# PART A: Provider Rules

## A1. Responsibility Separation

- Providers **MUST** handle:
  - State
  - Business logic

- Providers **MUST NOT**:
  - Access `BuildContext`
  - Control navigation
  - Show dialogs/snackbars
  - Contain UI logic

---

## A2. Directory Structure

```txt
lib/
  models/
  providers/
  screens/
  widgets/
```

````

Rules:

- All `ChangeNotifier` classes **MUST** be in `providers/`
- UI widgets **MUST NOT** be placed in `providers/`
- Data models **MUST NOT** depend on Providers

---

## A3. Provider Naming

- Provider class names **MUST** end with `Provider`
- Names **MUST** describe the managed state

Examples:

- ✅ `UserProvider`
- ❌ `CommonProvider`
- ❌ `DataProvider`

---

## A4. State Encapsulation

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

## A5. State Mutation Rules

- State **MUST** be modified only through methods
- Each method **MUST** call `notifyListeners()` at most once
- `notifyListeners()` **MUST** be called immediately after state changes

---

## A6. notifyListeners Usage

- Multiple `notifyListeners()` calls in a single method are **FORBIDDEN**
- Calling `notifyListeners()` without state change is **FORBIDDEN**

---

## A7. UI Access Rules

### Allowed

- `context.read<T>()` for actions
- `context.watch<T>()` for reactive values
- `Consumer<T>` for localized rebuilds

### Forbidden

- Watching providers at the screen root without reason
- Calling provider constructors in UI

---

## A8. Provider Lifecycle Scope

- Global providers **MUST** be created in `main.dart`
- Screen-scoped providers **MUST** be created in the screen entry widget
- Providers **MUST NOT** outlive their usage scope

---

## A9. Async Logic Pattern

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

## A10. Provider Instantiation

- Providers **MUST NOT** be instantiated manually in UI code
- Providers **MUST** be obtained via `Provider.of`, `read`, or `watch`

---

## A11. Rebuild Performance Rules

- `watch` usage **MUST** be minimal
- UI **MUST** isolate rebuilds using `Consumer` or child widgets
- Whole-screen rebuilds are **FORBIDDEN** unless explicitly required

---

# PART B: Riverpod Rules

## B1. Responsibility Separation

- Providers **MUST** handle:
  - State
  - Business logic
  - Data fetching

- Providers **MUST NOT**:
  - Access `BuildContext`
  - Control navigation
  - Show dialogs/snackbars
  - Contain UI logic

---

## B2. Directory Structure

```txt
lib/
  models/
  providers/
  screens/
  widgets/
  services/  (optional: API, repository)
```

Rules:

- All providers **MUST** be in `providers/`
- UI widgets **MUST NOT** be placed in `providers/`
- Data models **MUST NOT** depend on Providers
- Group related providers in the same file or subdirectory

---

## B3. Provider Types & Selection

### Provider Type Rules

- **Provider**: Read-only values that never change
- **StateProvider**: Simple state (int, String, bool, enum)
- **StateNotifierProvider**: Complex state with immutable models
- **FutureProvider**: Async data that loads once
- **StreamProvider**: Async streams
- **NotifierProvider/AsyncNotifierProvider**: Riverpod 2.0+ style (preferred)

### Mandatory Rules

- `StateNotifierProvider` **MUST** use immutable state classes
- `StateProvider` **MUST NOT** be used for complex objects
- Providers **MUST** be declared as `final` global variables
- `.autoDispose` **MUST** be used unless explicit caching is required

---

## B4. Provider Naming

- Provider variable names **MUST** end with `Provider`
- StateNotifier/Notifier class names **MUST** end with `Notifier`
- Names **MUST** describe the managed state

Examples:

- ✅ `final userProvider = StateNotifierProvider<UserNotifier, User>(...)`
- ✅ `class UserNotifier extends StateNotifier<User>`
- ✅ `final counterProvider = NotifierProvider<CounterNotifier, int>(...)`
- ❌ `final dataProvider = ...`
- ❌ `class CommonNotifier ...`

---

## B5. State Immutability

- State classes **MUST** be immutable
- Use `@freezed` or manual immutable classes
- State updates **MUST** use `.copyWith(...)` pattern
- Direct field mutation is **FORBIDDEN**

```dart
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
  }) = _User;
}

class UserNotifier extends StateNotifier<User> {
  UserNotifier() : super(const User(id: '', name: ''));

  void updateName(String name) {
    state = state.copyWith(name: name);
  }
}

final userProvider = StateNotifierProvider<UserNotifier, User>(
  (ref) => UserNotifier(),
);
```

---

## B6. Provider Dependencies

- Providers **MUST** declare dependencies via `ref.watch` or `ref.read`
- `ref.watch` **MUST** be used for reactive dependencies
- `ref.read` **MUST** be used for one-time reads in event handlers
- `ref.listen` **MUST** be used for side effects
- Circular dependencies are **FORBIDDEN**

```dart
final userIdProvider = StateProvider<String?>((ref) => null);

final userProvider = FutureProvider.autoDispose<User>((ref) async {
  final userId = ref.watch(userIdProvider);
  if (userId == null) throw Exception('No user ID');
  return await fetchUser(userId);
});
```

---

## B7. UI Access Rules

### Widget Types

- Widgets **MUST** extend `ConsumerWidget` or `ConsumerStatefulWidget`
- Alternative: `HookConsumerWidget` (with flutter_hooks)
- Regular `StatelessWidget` is **FORBIDDEN** when accessing providers

### Allowed

- `ref.read(provider)` for actions/events (onPressed, etc.)
- `ref.watch(provider)` for reactive values in `build`
- `ref.listen(provider, ...)` for side effects (navigation, dialogs)
- `Consumer` widget for localized rebuilds

### Forbidden

- Watching providers at screen root without reason
- Using `ref.read` for values displayed in UI
- Using `ref.watch` outside `build` method
- Calling provider constructors directly

```dart
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Text('$count'),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).increment(),
      ),
    );
  }
}
```

---

## B8. Side Effects Pattern

- Navigation/dialogs **MUST** use `ref.listen` in UI layer
- `ref.listen` **MUST NOT** be used inside providers
- Side effects **MUST NOT** be in `build` method body (except `ref.listen`)

```dart
class AuthScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.listen(authProvider, (previous, next) {
      if (next.isLoggedOut) {
        Navigator.of(context).pushReplacementNamed('/login');
      }
      if (next.hasError) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text(next.error.toString())),
        );
      }
    });

    return Scaffold(...);
  }
}
```

---

## B9. Async Logic Pattern

- Async providers **MUST** use `FutureProvider` or `AsyncNotifierProvider`
- Loading/error states **MUST** be handled via `AsyncValue`
- Manual loading flags are **FORBIDDEN** when using `AsyncValue`

```dart
final dataProvider = FutureProvider.autoDispose<Data>((ref) async {
  return await api.fetchData();
});

// In UI
Widget build(BuildContext context, WidgetRef ref) {
  final asyncData = ref.watch(dataProvider);

  return asyncData.when(
    data: (data) => Text(data.toString()),
    loading: () => CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
  );
}
```

---

## B10. StateNotifier Pattern

- StateNotifier **MUST** accept `Ref` if it has dependencies
- Async operations **MUST** properly handle state transitions
- State **MUST** be updated atomically

```dart
class TodoListNotifier extends StateNotifier<AsyncValue<List<Todo>>> {
  TodoListNotifier(this._ref) : super(const AsyncValue.loading()) {
    _fetchTodos();
  }

  final Ref _ref;

  Future<void> _fetchTodos() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final api = _ref.read(apiProvider);
      return await api.fetchTodos();
    });
  }

  Future<void> addTodo(String title) async {
    final currentState = state.value;
    if (currentState == null) return;

    state = AsyncValue.data([...currentState, Todo(title: title)]);

    try {
      final api = _ref.read(apiProvider);
      await api.createTodo(title);
    } catch (e) {
      state = AsyncValue.data(currentState); // rollback
      rethrow;
    }
  }
}

final todoListProvider = StateNotifierProvider<TodoListNotifier, AsyncValue<List<Todo>>>(
  (ref) => TodoListNotifier(ref),
);
```

---

## B11. Notifier Pattern (Riverpod 2.0+)

- Prefer `Notifier`/`AsyncNotifier` over `StateNotifier` for new code
- `build` method **MUST** initialize state
- Methods **MUST** update `state` directly

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// Usage in UI
final count = ref.watch(counterProvider);
ref.read(counterProvider.notifier).increment();
```

---

## B12. Code Generation (@riverpod)

- Code generation with `@riverpod` is **RECOMMENDED**
- Generated providers **MUST** follow part file convention
- Run `flutter pub run build_runner watch` during development

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'user_provider.g.dart';

@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  User build() => const User.empty();

  void updateName(String name) {
    state = state.copyWith(name: name);
  }
}

@riverpod
Future<List<User>> userList(UserListRef ref) async {
  final api = ref.watch(apiProvider);
  return await api.fetchUsers();
}
```

---

## B13. Provider Scope

- Global providers **MUST** be declared at top level
- Override providers **MUST** use `ProviderScope` or `.overrideWith`
- Family providers **MUST** use `.autoDispose` unless explicitly needed

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

// Override for testing
ProviderScope(
  overrides: [
    userProvider.overrideWith((ref) => mockUser),
  ],
  child: MyApp(),
)
```

---

## B14. Family Provider Rules

- Family providers **MUST** use `.autoDispose.family`
- Family parameters **MUST** be immutable
- Family parameters **MUST** implement `==` and `hashCode`

```dart
@riverpod
Future<Todo> todo(TodoRef ref, String id) async {
  return await api.fetchTodo(id);
}

// Usage
final todo = ref.watch(todoProvider('123'));
```

---

## B15. Rebuild Performance Rules

- `ref.watch` usage **MUST** be minimal
- Use `select` for partial state watching
- Whole-screen rebuilds are **FORBIDDEN** unless explicitly required

```dart
// ❌ Bad: rebuilds on any user change
final userName = ref.watch(userProvider).name;

// ✅ Good: rebuilds only when name changes
final userName = ref.watch(userProvider.select((user) => user.name));
```

---

## B16. Testing Rules

- Providers **MUST** be testable without UI
- Use `ProviderContainer` for unit tests
- Override providers using `.overrideWith`

```dart
test('counter increments', () {
  final container = ProviderContainer();

  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);

  container.dispose();
});
```

---

# COMMON RULES (Both Provider & Riverpod)

## C1. Rule Priority

1. Correctness
2. Type Safety
3. Readability
4. Performance
5. Convenience (lowest priority)

---

## C2. Enforcement

- Any code violating these rules **MUST** be rejected
- No exceptions unless explicitly documented

---

## C3. Migration Guide

When user requests migration from Provider to Riverpod:

- `ChangeNotifier` → `StateNotifier` or `Notifier`
- `ChangeNotifierProvider` → `StateNotifierProvider` or `NotifierProvider`
- `context.read` → `ref.read`
- `context.watch` → `ref.watch`
- `Consumer` → `Consumer` (same, but with `WidgetRef`)
- `StatelessWidget` → `ConsumerWidget`
- `StatefulWidget` → `ConsumerStatefulWidget`

---
````
