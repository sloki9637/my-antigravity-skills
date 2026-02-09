---
trigger: always_on
---

# Coding Style (Flutter + Provider + Flame)

This guide assumes:
- Flutter UI + Provider state management
- Game layer built with Flame
- Folder structure is fixed:
  components / constants / extensions / game / helpers / providers / repositories / screens

---

## 1. Immutability (CRITICAL – UI & Provider Only)

### Scope
- **providers/**: ✅ 반드시 immutable
- **Flutter UI state**: ✅ immutable
- **Flame Components**: ❌ 예외 (Flame은 mutation 기반)

### Provider Rules
- Provider state는 value object
- `copyWith` 필수
- 컬렉션 in-place 수정 금지

```dart
// ✅ Provider state
class GameUiState {
  final bool isPaused;
  final int score;

  const GameUiState({
    required this.isPaused,
    required this.score,
  });

  GameUiState copyWith({
    bool? isPaused,
    int? score,
  }) {
    return GameUiState(
      isPaused: isPaused ?? this.isPaused,
      score: score ?? this.score,
    );
  }
}
````

---

## 2. Flame Rule (중요)

Flame은 **ECS + Mutation 기반**이다.
여기서 불변성 강요하면 그냥 자살임.

### game/

* Flame `Component`, `PositionComponent`는 **mutable 허용**
* `update(dt)` 내부 mutation 허용
* 단, **Flame 컴포넌트 상태를 Provider로 직접 노출 금지**

❌ WRONG

```dart
context.read<GameProvider>().setPlayerPosition(player.position);
```

✅ CORRECT

```dart
// Flame → Game Event → Provider
gameEventBus.emit(PlayerDied());
```

---

## 3. Game ↔ UI Boundary (절대 규칙)

### Flame(Game) does NOT know UI

* `BuildContext` ❌
* Provider ❌
* Navigator ❌

### UI does NOT control Flame internals

* Component 직접 접근 ❌
* position / velocity 직접 수정 ❌

### Communication Pattern

```
Flame (game/) → Events → Provider → UI
UI → Intent → Provider → Flame Adapter
```

---

## 4. Folder Responsibilities (Flame 기준 보정)

### game/

* FlameGame
* Components
* Systems (collision, movement, spawn)
* Game-local helpers

Allowed:

* Mutation
* Game loop logic
* Physics, collision

Not allowed:

* Provider 직접 접근
* API / Repository 접근

---

### providers/

* Game UI state only

  * paused
  * score
  * result
* Flame 내부 상태 저장 ❌

---

### screens/

* GameScreen
* Overlay UI
* Pause / Result UI
* FlameGame lifecycle 관리

Allowed:

* `GameWidget`
* `context.watch<GameProvider>()`

---

### components/

* Flutter UI only
* Flame Component ❌ (game/로 가라)

---

## 5. Error Handling (Flame 특화)

* Flame 내부 에러:

  * try/catch → 로그
  * 게임 종료 이벤트 emit

```dart
try {
  spawnEnemy();
} catch (e) {
  eventBus.emit(GameCrashed(e));
}
```

* UI는 Provider state로만 반응

---

## 6. Lifecycle Rules (Flame + Flutter)

### Flame

* `onLoad`에서 async 허용
* `update`에서는 async ❌
* Stream / Timer 직접 관리 → `onRemove`에서 해제

### Flutter

* `GameWidget` 생성은 `initState`
* `build`에서 게임 생성 ❌

---

## 7. File Size & Structure

* Flame Component:

  * 1 Component = 1 파일
* God Component 금지
* System 성격이면 helpers/ 또는 game/systems/

---

## 8. Code Quality Checklist (Flame 포함)

Before marking work complete:

* [ ] Provider state immutable
* [ ] Flame Component는 game/에만 존재
* [ ] Flame ↔ UI 직접 참조 없음
* [ ] Game events로만 상태 전달
* [ ] async 코드가 update()에 없음
* [ ] dispose / onRemove 정리됨
* [ ] GameWidget이 build에서 생성되지 않음
* [ ] game 로직이 screens/에 없음

---