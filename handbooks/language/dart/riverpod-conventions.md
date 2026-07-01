# Riverpod Conventions

ENFORCEMENT: advisory

## Provider definition style

Use `riverpod_generator` (code-gen) for all providers:

```dart
// DO: annotated function for simple computed values
@riverpod
Future<List<Todo>> todoList(Ref ref) async {
  final repo = ref.watch(todoRepositoryProvider);
  return repo.getAll();
}

// DO: annotated class for notifiers with mutation methods
@riverpod
class TodoListNotifier extends _$TodoListNotifier {
  @override
  Future<List<Todo>> build() async {
    final repo = ref.watch(todoRepositoryProvider);
    return repo.getAll().first;
  }

  Future<void> add(String title) async {
    final repo = ref.read(todoRepositoryProvider);
    await repo.create(title);
    ref.invalidateSelf();
  }
}
```

## Dependency injection

- Repository is injected via a provider, not constructed in widgets:
  ```dart
  @riverpod
  TodoRepository todoRepository(Ref ref) => DriftTodoRepository(ref.watch(appDatabaseProvider));
  ```
- Database provider is a singleton: use `keepAlive: true` on the Drift DB provider

## Reading providers in widgets

- `ref.watch` — for values that rebuild the widget when they change
- `ref.read` — only inside callbacks/event handlers; never in `build()`
- `ref.listen` — for side effects (show SnackBar on error, navigate on success)

## Error states

- Use `AsyncValue<T>` (Riverpod's built-in) — covers data / loading / error
- In UI: always handle all three cases with `when(data:, loading:, error:)`
- Never `.value!` force-unwrap an `AsyncValue`

## Testing

```dart
// Use ProviderContainer for unit tests — no widget overhead
final container = ProviderContainer(overrides: [
  todoRepositoryProvider.overrideWithValue(FakeTodoRepository()),
]);
addTearDown(container.dispose);

final todos = await container.read(todoListProvider.future);
expect(todos, isEmpty);
```
