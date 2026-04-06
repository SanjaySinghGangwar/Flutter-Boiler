# Flutter & Dart Interview Study Guide

## Table of Contents

1. [Dart Language](#1-dart-language)
2. [Flutter Core](#2-flutter-core)
3. [State Management](#3-state-management)
4. [Architecture & Patterns](#4-architecture--patterns)
5. [Networking & Data](#5-networking--data)
6. [Testing](#6-testing)
7. [Performance & Optimisation](#7-performance--optimisation)
8. [Platform & Native](#8-platform--native)
9. [Tooling & CI/CD](#9-tooling--cicd)

---

## 1. Dart Language

### What is Dart?

Dart is a general-purpose, object-oriented language developed by Google (2011). It compiles to native machine code (AOT) for production and uses JIT during development for hot reload.

**Key features:** null safety, strong typing, async/await, generics, mixins, extensions, garbage collection.

**Use cases:** Flutter (mobile/web/desktop), backend servers, CLI tools.

---

### Data Types

| Type | Example |
|------|---------|
| `int` | `int x = 42;` |
| `double` | `double pi = 3.14;` |
| `String` | `String s = "hello";` |
| `bool` | `bool flag = true;` |
| `List<T>` | `List<int> nums = [1, 2, 3];` |
| `Map<K,V>` | `Map<String, int> m = {'a': 1};` |
| `dynamic` | `dynamic d = "anything";` |

---

### `var` vs `dynamic` vs `final` vs `const`

| Feature | `var` | `dynamic` | `final` | `const` |
|---------|-------|-----------|---------|---------|
| Can change type? | No | Yes | No | No |
| Can change value? | Yes | Yes | No | No |
| Type inferred? | Yes | No | Yes | Yes |
| Compile-time constant? | No | No | No | Yes |
| Mutability | Mutable | Mutable | Immutable after assignment | Immutable, compile-time |

```dart
var name = "Sanjay";          // inferred as String, cannot change type
dynamic anything = 42;        // can be reassigned to any type later
final now = DateTime.now();   // set at runtime, never reassigned
const pi = 3.14159;           // must be known at compile time
```

---

### `final` vs `const` vs `static`

| Feature | `final` | `const` | `static` |
|---------|---------|---------|---------|
| Mutability | Immutable after assignment | Immutable compile-time constant | Can be mutable |
| Initialization | Runtime | Compile time | Class-level |
| Shared among instances? | No | No | Yes |
| Use inside methods? | Yes | No | Yes |

---

### Null Safety & Null-Aware Operators

| Operator | Name | Usage |
|----------|------|-------|
| `??` | Null coalescing | `name ?? "Guest"` — returns right side if left is null |
| `?.` | Null-aware access | `user?.name` — safe property access |
| `??=` | Null-aware assignment | `name ??= "Guest"` — assigns only if null |
| `!` | Null assertion | `name!` — asserts value is non-null |

---

### `async` / `await` / `Future`

- `async` marks a function as asynchronous; it returns a `Future<T>`.
- `await` pauses execution until the Future completes — non-blocking.
- Must be used inside an `async` function.

```dart
Future<String> fetchUser() async {
  await Future.delayed(Duration(seconds: 1));
  return "Sanjay";
}

void main() async {
  final user = await fetchUser();
  print(user); // Sanjay
}
```

---

### `async` vs `async*`

| Feature | `async` | `async*` |
|---------|---------|----------|
| Returns | `Future<T>` | `Stream<T>` |
| Emits | One value | Multiple values over time |
| Keyword | `return` | `yield` |
| Consumed with | `await` | `await for` / `listen()` |

```dart
// async — one result
Future<int> add(int a, int b) async => a + b;

// async* — multiple results over time
Stream<int> count(int n) async* {
  for (int i = 1; i <= n; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}
```

---

### Streams

A Stream is a sequence of asynchronous events — like a pipeline where data enters one end and listeners receive it at the other.

**Two types:**
- **Single-subscription** — one listener, events delivered in order (e.g., reading a file).
- **Broadcast** — multiple listeners can subscribe simultaneously (e.g., event notifications).

```dart
Future<int> sumStream(Stream<int> stream) async {
  int sum = 0;
  await for (var value in stream) sum += value;
  return sum;
}
```

**StreamController** lets you add data to a stream manually. Always `close()` it in `dispose()`.

---

### `FutureBuilder` vs `StreamBuilder`

| Feature | `FutureBuilder` | `StreamBuilder` |
|---------|-----------------|-----------------|
| Data type | `Future<T>` | `Stream<T>` |
| Response count | One | Multiple |
| Common use | API call, DB query | Location updates, WebSocket, timer |
| Analogy | JS Promise | JS async iterator |

```dart
// FutureBuilder
FutureBuilder<String>(
  future: fetchData(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting)
      return CircularProgressIndicator();
    if (snapshot.hasError) return Text('Error: ${snapshot.error}');
    return Text(snapshot.data ?? '');
  },
);

// StreamBuilder
StreamBuilder<int>(
  stream: numberStream(),
  builder: (context, snapshot) => Text('${snapshot.data}'),
);
```

---

### Isolates & Concurrency

Dart is single-threaded. Isolates enable true parallel execution — each has its own memory (no shared state, no race conditions). Communication happens via `SendPort` / `ReceivePort`.

```dart
import 'dart:isolate';

void heavyTask(SendPort sendPort) {
  int sum = 0;
  for (int i = 0; i < 100000000; i++) sum += i;
  sendPort.send(sum);
}

void main() async {
  final receivePort = ReceivePort();
  await Isolate.spawn(heavyTask, receivePort.sendPort);
  receivePort.listen((msg) {
    print("Result: $msg");
    receivePort.close();
  });
}
```

`compute()` is the simpler wrapper for a one-off background task:

```dart
final result = await compute(parseData, rawJson);
```

| API | Use Case |
|-----|----------|
| `compute()` | Simple one-off background task |
| `Isolate.spawn()` | Long-running workers, full control |
| `SendPort` / `ReceivePort` | Bidirectional communication |

---

### Mixins

Dart does not support multiple inheritance. Mixins let you share behaviour across class hierarchies without inheritance.

```dart
mixin Walkable { void walk() => print("Walking"); }
mixin Runnable { void run()  => print("Running"); }

class Human with Walkable, Runnable {}

void main() {
  Human().walk();
  Human().run();
}
```

---

### Extension Methods

Add functionality to existing classes without modifying their source code (introduced in Dart 2.7).

```dart
extension StringExtension on String {
  String toTitleCase() => split(' ')
      .map((w) => w.isEmpty ? w : '${w[0].toUpperCase()}${w.substring(1)}')
      .join(' ');
}

void main() => print("hello dart".toTitleCase()); // Hello Dart
```

---

### Generics

Provide type-safe collections and functions. Avoid runtime type errors.

```dart
// Without generics — no type safety
List logs = [];
logs.add("WARNING");
logs.add(404); // no error, but dangerous

// With generics — compile-time safety
List<String> logs = [];
logs.add("WARNING");
// logs.add(404); // compile error
```

---

### Factory Constructors

A factory constructor does not always create a new instance — it can return an existing instance, a subclass, or apply custom logic.

```dart
class Singleton {
  static final Singleton _instance = Singleton._internal();

  factory Singleton() => _instance;

  Singleton._internal();
}
```

**When to use:**
- Singleton pattern
- Returning a subclass based on conditions
- JSON deserialization (`factory User.fromJson(Map json)`)
- Expensive object caching

---

### Constructors Summary

| Type | Description |
|------|-------------|
| Default | Basic, no parameters |
| Named | `ClassName.name()` — multiple constructors |
| Factory | Custom instantiation logic, can return existing instance |
| Super | `super()` — calls parent class constructor |

---

### Anonymous Functions (Lambdas / Closures)

```dart
var add = (int a, int b) => a + b;
print(add(5, 3)); // 8
```

---

### Typedef

Function-type alias — acts as a pointer to executable code.

```dart
typedef Operation(int a, int b);

void add(int a, int b) => print(a + b);
void Calculator(int a, int b, Operation op) => op(a, b);

void main() => Calculator(10, 5, add); // 15
```

---

### Spread Operator (`...`)

```dart
var a = [1, 2, 3];
var b = [4, 5];
var c = [...a, ...b]; // [1, 2, 3, 4, 5]
```

Null-aware spread: `[...?nullableList]`

---

### Parameter Types

| Type | Syntax | Notes |
|------|--------|-------|
| Positional (required) | `void f(String s)` | Must be passed in order |
| Named (optional) | `void f({String? s})` | Pass by name, optional |
| Named (required) | `void f({required String s})` | Pass by name, mandatory |
| Optional positional | `void f([String? s])` | Positional but optional |

---

### Dart Compilation Modes

| Mode | When | Notes |
|------|------|-------|
| JIT (Just-In-Time) | Development | Enables hot reload, slower execution |
| AOT (Ahead-Of-Time) | Production | Compiles to native machine code, fast startup |
| JS compilation | Web | Compiled to JavaScript via `dart2js` |

---

### Callable Classes

```dart
class Adder {
  int call(int a, int b) => a + b;
}

void main() {
  final adder = Adder();
  print(adder(3, 4)); // 7
}
```

---

### Runes

Integers representing Unicode code points. Dart strings are UTF-16; runes handle characters beyond `U+FFFF`.

```dart
String emoji = "heart";
print(emoji.runes); // Unicode code points
```

---

### `Symbol`

Opaque dynamic string names used in reflection to get metadata at runtime.

```dart
Symbol lib = Symbol("foo_lib");
print(lib); // Symbol("foo_lib")
```

---

### Dart FFI (Foreign Function Interface)

Call native C/C++ code directly — no platform channels needed. Ideal for CPU-intensive tasks.

**Supported:** Android, iOS, Windows, macOS, Linux (C APIs). Web uses JS interop instead.

---

### Iterable

Abstract collection class — cannot be instantiated directly. `List` and `Set` implement `Iterable`.

```dart
Iterable<int> nums = [1, 2, 3, 4, 5];
for (var n in nums) print(n);
```

---

### Equatable vs Freezed

| Feature | Equatable | Freezed |
|---------|-----------|---------|
| Purpose | Simplify equality comparison | Immutable data classes |
| Equality | Yes | Yes (deep) |
| `copyWith()` | No | Yes |
| Union types | No | Yes |
| JSON serialization | No | Yes |
| Boilerplate | Minimal | Requires code generation |

```dart
// Equatable
class User extends Equatable {
  final String name;
  final int age;
  const User({required this.name, required this.age});
  @override
  List<Object?> get props => [name, age];
}
```

---

## 2. Flutter Core

### What is Flutter?

Flutter is Google's open-source UI toolkit for building natively compiled apps for mobile, web, and desktop from a single Dart codebase.

**Technology stack:**

| Layer | Technology |
|-------|-----------|
| Framework | Dart — widgets, rendering, gestures, animations |
| Engine | C++ — Skia/Impeller graphics, Dart runtime, platform channels |
| Embedder | Platform-specific (iOS, Android, Web, Desktop) — event loops, OS APIs |

---

### Flutter Pros & Cons

**Pros:**
- Single codebase for Android, iOS, Web, Desktop
- Hot reload — real-time UI updates without restart
- AOT compilation — near-native performance
- Own rendering engine (Skia/Impeller) — consistent cross-platform UI
- Rich widget library and growing community

**Cons:**
- Dart has a smaller ecosystem than JavaScript/Kotlin
- No code push (updates require a full app release)
- Larger APK/IPA size than native
- Complex native integrations still require platform channels
- Limited 3D graphics support

---

### The Three Trees

| Tree | Role |
|------|------|
| **Widget Tree** | Immutable UI description — rebuilt on state changes |
| **Element Tree** | Mutable, manages widget lifecycle, bridges widgets and render objects |
| **Render Object Tree** | Handles layout, sizing, painting, and compositing |

**Rendering pipeline:**
```
Widget Tree → Element Tree → Render Object Tree → Layout → Paint → Compositor → Screen
```

---

### Widget Types

| Category | Examples |
|----------|---------|
| Structural | `Container`, `Column`, `Row`, `Stack`, `GridView` |
| Interactive | `GestureDetector`, `InkWell`, `Form`, `TextField`, `Checkbox` |
| Styling | `Padding`, `Align`, `Center`, `SizedBox`, `DecoratedBox` |
| Text & Media | `Text`, `Image`, `Icon`, `RichText` |
| Scrolling | `ListView`, `GridView`, `SingleChildScrollView` |
| Navigation | `Navigator`, `PageView`, `BottomNavigationBar`, `Drawer` |
| Material | `Scaffold`, `AppBar`, `FloatingActionButton`, `SnackBar` |
| Cupertino | `CupertinoButton`, `CupertinoAlertDialog`, `CupertinoNavigationBar` |

---

### StatelessWidget vs StatefulWidget

| Feature | StatelessWidget | StatefulWidget |
|---------|-----------------|----------------|
| State | None | Has mutable `State` object |
| Rebuilds | Only when parent changes | Can rebuild itself via `setState()` |
| Examples | `Text`, `Icon`, `Container` | `Checkbox`, `Slider`, `TextField` |
| Use when | UI never changes dynamically | UI updates on interaction or data |

---

### StatefulWidget Lifecycle

| Method | When Called |
|--------|-------------|
| `createState()` | Widget is first created — returns `State` object |
| `initState()` | State inserted into tree — initialize variables, subscribe to streams |
| `didChangeDependencies()` | After `initState()` or when `InheritedWidget` dependency changes |
| `build()` | On every rebuild — return the UI |
| `didUpdateWidget()` | Parent widget passes new configuration |
| `deactivate()` | Widget temporarily removed from tree |
| `dispose()` | Widget permanently removed — clean up controllers, streams, listeners |

**Rule:** Always call `dispose()` to release `AnimationController`, `StreamSubscription`, `TextEditingController`, etc.

---

### `build()` is on `State`, Not `StatefulWidget`

`StatefulWidget` is immutable — it holds configuration only. The `State` class is mutable and survives widget rebuilds. Flutter can recreate the widget but retains the `State`, so `build()` lives there to allow efficient UI updates.

---

### AppLifecycleState

| State | Description |
|-------|-------------|
| `resumed` | App visible, receiving user input |
| `inactive` | App not receiving input (iOS — transition between states) |
| `paused` | App not visible, running in background |
| `detached` | App suspended (Android only) |

Use `WidgetsBindingObserver` to listen for these changes (e.g., pause video, save state).

---

### `setState()`

Notifies Flutter that the widget's state has changed and schedules a rebuild of `build()`.

```dart
void _increment() {
  setState(() {
    _counter++;
  });
}
```

**Only call it when state actually changes.** Avoid heavy computation inside `setState`.

---

### `initState()` Common Uses

- Initialize variables
- Subscribe to streams
- Fetch initial data (via `async` helper, not `initState` itself)
- Set up `AnimationController`

---

### `SafeArea`

Adds padding to prevent UI from being obscured by status bars, notches, and navigation bars.

```dart
SafeArea(
  child: Center(child: Text("Safe content")),
)
```

---

### `vsync`

Vertical synchronisation — prevents animation rendering when the screen is not visible. Requires `TickerProviderStateMixin` or `SingleTickerProviderStateMixin`.

```dart
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: Duration(seconds: 2));
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

---

### Animations

| Type | API | When to use |
|------|-----|-------------|
| Implicit | `AnimatedContainer`, `AnimatedOpacity` | Simple property transitions |
| Explicit | `AnimationController` + `Tween` | Full timing/curve control |
| Hero | `Hero` widget | Shared-element transitions between screens |
| Tween | `Tween<double>(begin: 0, end: 1).animate(controller)` | Interpolate values |

```dart
// Explicit animation with AnimatedBuilder
_scaleAnimation = Tween<double>(begin: 0.5, end: 1.5).animate(
  CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
);

AnimatedBuilder(
  animation: _scaleAnimation,
  builder: (context, child) => Transform.scale(scale: _scaleAnimation.value, child: child),
  child: Container(width: 100, height: 100, color: Colors.blue),
);
```

Custom animation curve:
```dart
class BounceCurve extends Curve {
  @override
  double transform(double t) => t * t * (3 - 2 * t);
}
```

---

### Navigation

| Approach | Best for |
|----------|---------|
| `Navigator.push/pop` | Small apps, simple flows |
| Named routes | Medium apps, better organization |
| `onGenerateRoute` | Dynamic routes, arguments |
| GoRouter / AutoRoute | Large apps, deep linking, URL-based |

```dart
// Push
Navigator.push(context, MaterialPageRoute(builder: (_) => SecondPage()));

// Named
Navigator.pushNamed(context, '/second', arguments: data);

// Pop with data
Navigator.pop(context, returnValue);
```

Custom transition using `PageRouteBuilder`:
```dart
class FadeRoute extends PageRouteBuilder {
  final Widget page;
  FadeRoute({required this.page}) : super(
    pageBuilder: (_, __, ___) => page,
    transitionsBuilder: (_, animation, __, child) =>
        FadeTransition(opacity: animation, child: child),
  );
}
```

---

### Gestures

`GestureDetector` is the primary widget for detecting user input.

```dart
GestureDetector(
  onTap: () => print("Tap"),
  onDoubleTap: () => print("Double tap"),
  onLongPress: () => print("Long press"),
  child: Container(color: Colors.blue, width: 100, height: 100),
)
```

---

### Layout System

Flutter layout follows a **constraint-based** model:
1. Parent passes **constraints** (min/max width & height) to child.
2. Child decides its **size** within those constraints.
3. Parent **positions** the child.

**Common layout widgets:**

| Widget | Purpose |
|--------|---------|
| `Row` / `Column` | Horizontal / vertical arrangement |
| `Stack` | Overlapping widgets |
| `Expanded` | Takes all remaining space in Row/Column |
| `Flexible` | Takes space proportionally; child can be smaller |
| `Spacer` | Flexible empty space between widgets |
| `Align` | Positions child within parent |
| `Container` | Box with padding, margin, decoration, size |
| `SizedBox` | Fixed-size box; also used as whitespace |

---

### `Expanded` vs `Flexible`

| Feature | `Expanded` | `Flexible` |
|---------|------------|------------|
| Space taken | All remaining space (forced) | As much as child allows |
| Flex behavior | Child must fill space | Child can be smaller |

---

### `Row`/`Column` Alignment

- `mainAxisAlignment` — along the primary axis (horizontal for `Row`, vertical for `Column`)
- `crossAxisAlignment` — along the perpendicular axis

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [...],
)
```

---

### `Container` vs `SizedBox`

| Feature | `Container` | `SizedBox` |
|---------|-------------|------------|
| Styling | Yes (padding, margin, decoration, color) | No |
| Size enforcement | Yes | Yes |
| Use when | Need styling + sizing | Only need fixed size or spacing |

Cannot use `color` and `decoration` together in `Container` — put `color` inside `BoxDecoration`.

---

### `double.infinity` vs `MediaQuery`

| Feature | `double.infinity` | `MediaQuery` |
|---------|-------------------|--------------|
| What it measures | Parent's max allowed size | Device screen size |
| Scope | Widget-local | App-wide |

```dart
Container(width: double.infinity)         // expands to parent's max width
Container(width: MediaQuery.of(context).size.width)  // actual screen width
```

---

### `MediaQuery` vs `LayoutBuilder`

| Feature | `MediaQuery` | `LayoutBuilder` |
|---------|-------------|-----------------|
| Measures | Full screen size | Parent widget constraints |
| Best for | Device dimensions, safe area | Adaptive layouts inside widget trees |

---

### `AspectRatio`

Maintains a specific width-to-height ratio regardless of screen size.

```dart
AspectRatio(
  aspectRatio: 16 / 9,
  child: Container(color: Colors.blue),
)
```

---

### `BuildContext`

A reference to a widget's location in the widget tree. Used to:
- Access `Theme`, `Navigator`, `MediaQuery`
- Find `InheritedWidget` ancestors

```dart
Theme.of(context);
Navigator.of(context).push(route);
```

---

### `InheritedWidget` vs `Provider`

| Feature | `InheritedWidget` | `Provider` |
|---------|-------------------|------------|
| Boilerplate | High | Low |
| Performance | O(N²) listeners | O(N) |
| Flutter team recommendation | Legacy | Recommended |
| Granular rebuilds | No | Yes (via `Consumer`) |

Provider is built on top of `InheritedWidget` but adds selective rebuild, better API, and cleaner code.

---

### `WidgetsBindingObserver`

Listens for app lifecycle changes. Attach in `initState`, remove in `dispose`.

**Use cases:** Pause/resume video, save state, handle push notifications by lifecycle.

---

### `WidgetsApp` vs `MaterialApp`

| Feature | `WidgetsApp` | `MaterialApp` |
|---------|-------------|---------------|
| Material Design | No | Yes |
| Theme support | Basic | Full `ThemeData` |
| Use when | Custom UI | Material Design apps |

---

### `Scaffold` Key Features

| Feature | Description |
|---------|-------------|
| `appBar` | Top navigation bar |
| `body` | Main content |
| `floatingActionButton` | Primary action button |
| `bottomNavigationBar` | Tab navigation |
| `drawer` | Side navigation menu |
| `SnackBar` | Temporary bottom message |

---

### `MaterialApp` Key Features

| Feature | Description |
|---------|-------------|
| Navigation | Manages routes via `routes` map or `onGenerateRoute` |
| Theming | `ThemeData` for colors, typography, shapes |
| Localization | `supportedLocales` + `localizationsDelegates` |
| Accessibility | Built-in screen reader support |

---

### `ListView` Types

| Type | Description |
|------|-------------|
| `ListView()` | Simple scrollable list |
| `ListView.builder()` | Lazy-builds items — use for large lists |
| `ListView.separated()` | Adds dividers between items |
| `ListView.custom()` | Full customisation via `SliverChildDelegate` |

---

### `GridView` Types

| Type | Description |
|------|-------------|
| `GridView.count()` | Fixed column count |
| `GridView.builder()` | Dynamic/lazy-built grid |
| `GridView.extent()` | Fixed item width, auto column count |
| `GridView.custom()` | Full customisation |

---

### `ListView` vs `GridView`

| Feature | `ListView` | `GridView` |
|---------|------------|------------|
| Layout | Single column/row | Rows and columns |
| Item size | Variable height | Usually fixed |
| Best for | Linear lists | Galleries, grids |

---

### `FittedBox`

Scales and positions its child to fit within the available space while maintaining the child's aspect ratio. Useful for responsive text or image sizing.

---

### `RefreshIndicator`

Enables pull-to-refresh. The `onRefresh` callback is triggered when the user pulls down.

---

### `Image.network()`

Loads images from a URL asynchronously with built-in `loadingBuilder` and `errorBuilder`.

```dart
Image.network(
  'https://example.com/image.png',
  loadingBuilder: (ctx, child, progress) =>
      progress == null ? child : CircularProgressIndicator(),
  errorBuilder: (ctx, error, stack) => Icon(Icons.error),
);
```

---

### Keys

Unique identifiers that help Flutter retain widget state across rebuilds.

| Key Type | Use case |
|----------|---------|
| `ValueKey` | Identify by value (e.g., item id) |
| `ObjectKey` | Identify by object instance |
| `UniqueKey` | Always unique, forces rebuild |
| `GlobalKey` | Access widget state/context from anywhere in the tree |

**When to use:** reordering lists, preserving state in stateful widgets across rebuilds.

---

### `pubspec.yaml`

Configuration file for Flutter projects.

| Section | Purpose |
|---------|---------|
| `name` | Project name |
| `version` | App version (`1.0.0+1`) |
| `dependencies` | Packages needed in production build |
| `dev_dependencies` | Packages for testing/code gen only |
| `flutter.assets` | Images, fonts, JSON files |

```yaml
dependencies:
  http: ^0.14.0
  provider: ^6.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.0.0
```

---

### Hot Reload vs Hot Restart

| Feature | Hot Reload | Hot Restart |
|---------|------------|-------------|
| Speed | Fast | Slower |
| State preserved? | Yes | No — resets |
| Best for | UI tweaks | Structural changes, state reset |

---

### Build Modes

| Mode | Purpose | Performance | Command |
|------|---------|-------------|---------|
| Debug | Development | Slowest — extra checks | `flutter run` |
| Profile | Performance analysis | Near-production | `flutter run --profile` |
| Release | App store deployment | Fastest, optimised | `flutter run --release` |

---

### Platform Detection

```dart
import 'dart:io' show Platform;
import 'package:flutter/foundation.dart' show kIsWeb;

String get platformName => kIsWeb ? "Web"
    : Platform.isAndroid ? "Android"
    : Platform.isIOS ? "iOS"
    : Platform.isMacOS ? "macOS"
    : Platform.isWindows ? "Windows"
    : "Linux";
```

Debug-only code:
```dart
import 'package:flutter/foundation.dart';
if (kDebugMode) print('Debug only');
```

---

### `Form`, `TextField`, `TextFormField`

| Feature | `Form` | `TextField` | `TextFormField` |
|---------|--------|-------------|-----------------|
| Purpose | Groups fields for validation | Single input | Input with validation |
| Validation | Yes (all fields at once) | No | Yes |
| Use when | Multiple fields + submit | One-off input | Inside a `Form` |

---

### `BuildContext` — `main()` vs `runApp()`

| Function | Role |
|----------|------|
| `main()` | Entry point — every Dart program starts here |
| `runApp(widget)` | Attaches the root widget to the screen and starts the widget tree |

---

### Android Status Bar

```dart
// Hide
SystemChrome.setEnabledSystemUIOverlays([]);

// Restore
SystemChrome.setEnabledSystemUIOverlays(SystemUiOverlay.values);
```

---

### `FlutterActivity` Responsibilities (Android)

- Configure status bar
- Show Android and Flutter splash screens
- Determine Dart execution path
- Save/restore instance state
- Support transparent backgrounds

---

### Internationalization (i18n)

```dart
MaterialApp(
  supportedLocales: [Locale('en', 'US'), Locale('es', 'ES')],
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  home: HomeScreen(),
)
```

---

## 3. State Management

### Ephemeral vs App State

| Type | Scope | Tool |
|------|-------|------|
| Ephemeral (local) | Single widget | `setState()` |
| App state (global) | Multiple widgets / sessions | Provider, Riverpod, BLoC, etc. |

---

### State Management Comparison

| Solution | Pros | Cons | Best For |
|----------|------|------|---------|
| `setState` | Simple, built-in | Not scalable | Single widget, prototypes |
| Provider | Simple, Flutter-recommended | Boilerplate for complex apps | Small–medium apps |
| Riverpod | Type-safe, no `BuildContext`, async-friendly | Learning curve | Modern standard, new apps |
| BLoC | Strict separation, highly testable | Verbose, higher learning curve | Large/enterprise apps |
| GetX | Minimal code, built-in routing+DI | No enforced architecture | Quick prototypes only |
| Redux | Predictable, undo/redo | Complex setup | Strict state history |
| MobX | Reactive, less boilerplate | "Magic" — less transparent | Reactive state systems |

**Recommendation:** Riverpod for new apps, BLoC for enterprise, Provider for legacy.

---

### `setState()` vs `Provider`

| Feature | `setState()` | `Provider` |
|---------|-------------|------------|
| Scope | Single widget | App-wide |
| Rebuilds | Full widget tree at call site | Only listening widgets |
| State persistence | Lost on widget removal | Persists |
| Complexity | Minimal | Requires `ChangeNotifier` + `Consumer` |

---

### BLoC (Business Logic Component)

Reactive state management using Streams. Events go in, States come out.

**Flow:** `UI Event → BLoC → State → UI`

```
BLoC vs Other Architectures
MVP: BLoC acts as Presenter
MVVM: BLoC acts as ViewModel
```

**Key components:**
- `StreamController` — emits events
- `StreamBuilder` — listens and rebuilds UI
- `Sink` — input channel for events

---

### BLoC vs Other Frameworks

| Feature | BLoC | Provider | GetX |
|---------|------|----------|------|
| State management | Streams + Events | ChangeNotifier | Observable `.obs` |
| Testability | High | Medium | Low |
| Separation of concerns | Strict | Moderate | Minimal |
| Boilerplate | High | Medium | Low |
| Best for | Large apps | Medium apps | Prototypes |

---

### RxDart vs Vanilla Streams

| Feature | RxDart | Vanilla Dart Streams |
|---------|--------|----------------------|
| Operators | Rich (filter, map, debounce, merge) | Basic |
| Multiple subscriptions | `BehaviorSubject`, `ReplaySubject` | Single-subscription only |
| Use case | Complex reactive apps, BLoC | Simple async data |

---

### Provider Usage

```dart
// pubspec.yaml: provider: ^6.0.5

// Model
class Counter extends ChangeNotifier {
  int value = 0;
  void increment() { value++; notifyListeners(); }
}

// Root
ChangeNotifierProvider(create: (_) => Counter(), child: MyApp())

// Consume
context.watch<Counter>().value  // rebuilds on change
context.read<Counter>().increment()  // read only, no rebuild
```

---

### StreamProvider

Provides a stream of data to child widgets. Automatically updates children when new data arrives.

```dart
StreamProvider<int>(
  create: (_) => myStream(),
  initialData: 0,
  child: MyWidget(),
)
```

Access data with `context.watch<int>()`.

---

### GetX

All-in-one: state management + navigation + dependency injection.

```dart
class CounterController extends GetxController {
  var count = 0.obs;
  void increment() => count++;
}

final c = Get.put(CounterController());
Obx(() => Text('${c.count}'));

// Navigation — no BuildContext needed
Get.to(SecondScreen());
Get.back();
```

Avoid for large apps — lacks enforced architecture.

---

## 4. Architecture & Patterns

### MVC vs MVP vs MVVM

| Pattern | Components | Key Difference |
|---------|------------|----------------|
| MVC | Model, View, Controller | Tight coupling; Controller updates both Model and View |
| MVP | Model, View, Presenter | Presenter fetches data, View is passive |
| MVVM | Model, View, ViewModel | View observes ViewModel reactively; best testability |

**In Flutter:** BLoC = ViewModel. MVVM/BLoC is the most common architecture.

---

### Clean Architecture Layers

```
Presentation (UI) → Domain (Use Cases) → Data (Repository + API/DB)
```

1. **Entities** — business objects, framework-independent
2. **Use Cases** — application-specific business rules
3. **Interface Adapters** — convert data between layers
4. **Frameworks & Drivers** — UI, databases, external APIs

**Goal:** Decouple business logic from implementation details.

---

### Repository Pattern

Abstracts data sources. Business logic doesn't care if data comes from API, DB, or cache.

```dart
// Domain layer — interface
abstract class UserRepository {
  Future<User> getUser(int id);
}

// Data layer — implementation
class UserRepositoryImpl implements UserRepository {
  final ApiService _api;
  UserRepositoryImpl(this._api);

  @override
  Future<User> getUser(int id) => _api.fetchUser(id);
}
```

---

### Facade Pattern

One simple interface hiding a complex subsystem.

```dart
class UserFacade {
  Future<User> getUser(int id) async =>
      _cache.get(id) ?? await _db.find(id) ?? await _api.fetch(id);
}
```

---

### Singleton Pattern

One instance globally accessible.

```dart
class MySingleton {
  static final MySingleton _instance = MySingleton._internal();
  factory MySingleton() => _instance;
  MySingleton._internal();
}
```

---

### SOLID Principles

| Principle | Short Rule | Flutter Example |
|-----------|------------|----------------|
| **S** — Single Responsibility | One class, one reason to change | `UserService` handles auth; `UserModel` holds data |
| **O** — Open/Closed | Open for extension, closed for modification | Use abstract classes + inheritance |
| **L** — Liskov Substitution | Subclass can replace parent without breaking app | `Square` behaves as `Rectangle` |
| **I** — Interface Segregation | Don't force unused interface methods | Split `FileManager` into `Readable` + `Writable` |
| **D** — Dependency Inversion | Depend on abstractions, not concretions | Inject `DatabaseInterface`, not `SQLiteService` directly |

---

### Dependency Injection

An object receives its dependencies from outside instead of creating them internally. Improves testability and decoupling.

```dart
class MyApp {
  final DatabaseService db;
  MyApp(this.db);  // injected
}

void main() {
  final app = MyApp(DatabaseService());
}
```

**Tools:** `GetIt` (service locator), `Injectable` (code generation), `Riverpod` (built-in DI).

---

### GetIt

Service locator — global dependency access without `BuildContext`.

```dart
final getIt = GetIt.instance;

void setup() {
  getIt.registerSingleton<ApiService>(ApiService());
  getIt.registerFactory<UserRepository>(() => UserRepositoryImpl(getIt()));
}

final api = getIt<ApiService>(); // access anywhere
```

---

### Injectable

Auto-generates `GetIt` registration code via annotations.

```dart
@injectable class ApiService { ... }
@singleton class AuthService { ... }
@lazySingleton class DatabaseService { ... }
```

Run `flutter pub run build_runner build` to generate.

---

### BLoC Design Patterns

BLoC integrates with MVC and MVVM:
- **MVC** — BLoC acts as Controller
- **MVVM** — BLoC acts as ViewModel

**Common BLoC patterns:**
- `StreamController` emits events
- `StreamBuilder` listens to state and rebuilds UI
- `Sink` accepts external inputs

---

### Multiple Inheritance — Why Dart Avoids It

Dart does not support multiple inheritance to avoid the **Diamond Problem** (ambiguity when two parents have the same method). Use **mixins** for code reuse across class hierarchies instead.

---

### Adaptive vs Responsive Design

| Approach | Description |
|----------|-------------|
| **Adaptive** | Multiple fixed layouts selected based on device type |
| **Responsive** | Fluid layout that adjusts dynamically |

```dart
// Adaptive
if (MediaQuery.of(context).size.width > 600)
  return TabletLayout();
return MobileLayout();

// Responsive
Container(width: MediaQuery.of(context).size.width * 0.8)
```

---

### Getter and Setter

```dart
class Person {
  String _name = "";

  String get name => _name;

  set name(String value) {
    if (value.isNotEmpty) _name = value;
  }
}
```

---

## 5. Networking & Data

### HTTP Requests

Use `http` package or `Dio` instead of the built-in `HttpClient`.

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

Future<void> fetchData() async {
  final response = await http.get(Uri.parse('https://api.example.com/data'));
  if (response.statusCode == 200) {
    final data = jsonDecode(response.body);
    print(data);
  } else {
    throw Exception('Failed to load data');
  }
}
```

**Best practices:**
- Use a dedicated service/repository class
- Always `try-catch` network calls
- Use `FutureBuilder` to handle loading/error/data states
- Cache and paginate large data sets

---

### Dio

Powerful HTTP client — more features than the default `http` package.

**Key features:** Interceptors, timeouts, file upload/download, FormData, cancellation tokens.

```dart
final dio = Dio(BaseOptions(baseUrl: 'https://api.example.com'));

final response = await dio.get('/users');
final res = await dio.post('/login', data: {'email': 'a@b.com', 'password': '123'});
```

---

### Dio Interceptors

Intercept requests, responses, or errors globally.

| Type | Use |
|------|-----|
| Request | Add auth token, modify headers |
| Response | Transform response data |
| Error | Handle 401/500, token refresh |

```dart
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    options.headers['Authorization'] = 'Bearer $token';
    return handler.next(options);
  },
  onError: (error, handler) {
    if (error.response?.statusCode == 401) { /* refresh token */ }
    return handler.next(error);
  },
));
```

---

### Local Databases

| Database | Package | Type | Best For |
|---------|---------|------|---------|
| SQLite | `sqflite` | Relational / SQL | Structured data with relationships |
| Hive | `hive` | Key-value NoSQL | Caching, settings, fast reads |
| Isar | `isar` | NoSQL (Dart API) | Large datasets, complex filtering |
| Firebase Firestore | `cloud_firestore` | Cloud NoSQL | Real-time sync, serverless backend |

```dart
// SQLite
final db = await openDatabase('app.db', onCreate: (db, v) {
  return db.execute('CREATE TABLE users(id INTEGER PRIMARY KEY, name TEXT)');
}, version: 1);
await db.insert('users', {'name': 'Sanjay'});

// Hive
await Hive.initFlutter();
final box = await Hive.openBox('settings');
box.put('theme', 'dark');
final theme = box.get('theme');

// Isar
final isar = await Isar.open([UserSchema]);
await isar.writeTxn(() async => await isar.users.put(User()..name = 'Sanjay'));
```

---

### Secure Storage

Use `flutter_secure_storage` for sensitive data (tokens, passwords). Data is stored in the device keychain (iOS) or EncryptedSharedPreferences (Android).

---

### Platform Channels

Allow Flutter to call native Android (Java/Kotlin) and iOS (Swift/Obj-C) code.

```dart
// Flutter side
static const _channel = MethodChannel('custom_channel');
final version = await _channel.invokeMethod<String>('getPlatformVersion');
```

```kotlin
// Android (Kotlin)
MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "custom_channel")
  .setMethodCallHandler { call, result ->
    if (call.method == "getPlatformVersion")
      result.success("Android ${Build.VERSION.RELEASE}")
  }
```

```swift
// iOS (Swift)
let channel = FlutterMethodChannel(name: "custom_channel", binaryMessenger: controller.binaryMessenger)
channel.setMethodCallHandler { call, result in
  if call.method == "getPlatformVersion" { result("iOS \(UIDevice.current.systemVersion)") }
}
```

---

### Building APK / IPA

```sh
flutter build apk --release    # Android APK
flutter build ios --release    # iOS IPA (requires macOS + Xcode)
```

---

## 6. Testing

### Three Types of Tests

| Type | Purpose | Speed |
|------|---------|-------|
| Unit | Test individual functions/classes — no UI | Fastest |
| Widget | Test single widget UI and interactions | Medium |
| Integration | Test full app flow — UI + API + DB | Slowest |

---

### Unit Testing

```dart
import 'package:flutter_test/flutter_test.dart';

int add(int a, int b) => a + b;

void main() {
  test('adds two numbers', () {
    expect(add(2, 3), equals(5));
  });
}
```

Run: `flutter test`

---

### Widget Testing

```dart
void main() {
  testWidgets('button increments counter', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());
    expect(find.text('0'), findsOneWidget);
    await tester.tap(find.byType(FloatingActionButton));
    await tester.pump();
    expect(find.text('1'), findsOneWidget);
  });
}
```

---

### Integration Testing

```dart
// integration_test/app_test.dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('login flow', (tester) async {
    await tester.pumpWidget(MyApp());
    await tester.tap(find.byKey(Key('loginButton')));
    await tester.pumpAndSettle();
    expect(find.text('Home'), findsOneWidget);
  });
}
```

Run: `flutter test integration_test`

---

### Mocking with Mockito

```dart
@GenerateMocks([UserRepository])
void main() {
  late MockUserRepository mockRepo;
  setUp(() => mockRepo = MockUserRepository());

  test('returns user on success', () async {
    when(mockRepo.getUser(1)).thenAnswer((_) async => User(id: 1, name: 'Test'));
    final user = await mockRepo.getUser(1);
    expect(user.name, 'Test');
  });
}
```

---

## 7. Performance & Optimisation

### Reducing Widget Rebuilds

- Use `const` constructors for immutable widgets — Flutter skips their rebuild entirely.
- Lift state up; avoid rebuilding large subtrees.
- Use `Consumer` / `Selector` with Provider/Riverpod to rebuild only affected widgets.
- Replace `setState` with scoped state management for complex UIs.

```dart
const Text('Hello'); // never rebuilt
```

---

### Keys for Performance

Use `ValueKey` or `UniqueKey` in `ListView.builder` when reordering items to preserve widget state.

---

### Lazy Loading

Defer object initialisation until it is needed.

```dart
class LazyData {
  String? _data;
  String get data => _data ??= "Loaded on first access";
}
```

`ListView.builder()` is lazy by default — builds items only as they scroll into view.

---

### `compute()` for Background Work

Use `compute()` to run a heavy function on a separate isolate and keep the UI responsive.

```dart
List<int> parseData(String json) => jsonDecode(json).cast<int>();
final result = await compute(parseData, rawJson);
```

---

### Performance Optimisation Checklist

- Use `const` where possible
- Use `ListView.builder` over `ListView` for large lists
- Prefer `SizedBox` over `Container` for spacing
- Dispose controllers, streams, and listeners in `dispose()`
- Use `RepaintBoundary` to isolate frequently repainted widgets
- Profile with Flutter DevTools — check "Performance" and "Widget Rebuild" tabs
- Use AOT compilation in release builds
- Tree-shake unused code; keep package footprint small

---

### Adaptive vs Responsive (Performance Angle)

Adaptive layouts switch layouts at breakpoints (more predictable). Responsive layouts use `LayoutBuilder` and `MediaQuery` to scale dynamically (more flexible but can cause more rebuilds if not scoped).

---

### `FittedBox` for Responsive Text

Automatically scales child to fit available space — prevents text overflow.

---

## 8. Platform & Native

### Android & iOS Directories

Flutter requires separate `android/` and `ios/` directories because the final app is compiled into native platform binaries.

| Directory | Contents |
|-----------|---------|
| `android/` | Gradle files, `AndroidManifest.xml`, Kotlin/Java native code |
| `ios/` | Xcode project, `Info.plist`, Swift/Objective-C native code |

**Why:** Platform-specific dependency managers (Gradle vs CocoaPods), app store configurations, and plugin native implementations all differ per platform.

---

### Flutter Plugin vs Package

| Type | Description |
|------|-------------|
| Package | Pure Dart code — no native dependencies |
| Plugin | Dart + native code (Kotlin/Swift) via platform channels |

To create a plugin: write platform-specific code, expose it via `MethodChannel` in Dart.

---

### App Security Checklist

- Code obfuscation (`flutter build apk --obfuscate`)
- Secure storage (keychain/EncryptedSharedPreferences)
- SSL pinning
- Local authentication (biometrics)
- Restrict network traffic (certificate pinning)
- Clear sensitive data from memory on app background
- Jailbreak/root detection

---

### `FlutterActivity` (Android)

Entry point for Flutter on Android. Responsibilities:
- Status bar configuration
- Splash screen management (native + Flutter)
- Instance state save/restore
- Dart execution path

---

## 9. Tooling & CI/CD

### Flutter SDK Components

| Component | Purpose |
|-----------|---------|
| Dart SDK | Language + standard libraries |
| Rendering engine | Widget framework + Skia |
| Compilation tools | Native code generation |
| DevTools | Debug, profile, inspect |
| CLI tools | `flutter run`, `flutter build`, `flutter test` |

---

### Flutter DevTools

| Tab | Purpose |
|-----|---------|
| Flutter Inspector | Visualise widget tree, check constraints |
| Performance | Find jank, frame rendering timeline |
| CPU Profiler | Identify expensive Dart/native methods |
| Memory | Detect leaks, track allocations |
| Network | Inspect HTTP requests/responses |

---

### CI/CD Tools

| Tool | Purpose |
|------|---------|
| Codemagic | Flutter-first CI/CD, auto builds & deploys |
| GitHub Actions | Custom workflows, free for public repos |
| Fastlane | Automate signing, screenshots, App Store upload |
| Firebase App Distribution | Beta testing distribution |

---

### `pubspec.yaml` — `dependencies` vs `dev_dependencies`

| Section | Included in production? | Examples |
|---------|------------------------|---------|
| `dependencies` | Yes | `http`, `provider`, `flutter_bloc` |
| `dev_dependencies` | No | `mockito`, `flutter_test`, `build_runner` |

---

### Why First Build Takes Long

Flutter compiles device-specific APK (Gradle) or IPA (Xcode) on first build. Subsequent builds are faster due to caching.

---

### App State Examples

- Login info (persisted session)
- User preferences (dark mode, language)
- Shopping cart contents
- Real-time notification counts

---

### Streams — `StreamTransformer`

Transform a stream of data from one form to another.

```dart
final squareTransformer = StreamTransformer<int, int>.fromHandlers(
  handleData: (value, sink) => sink.add(value * value),
);

controller.stream.transform(squareTransformer).listen(print);
controller.add(3); // prints 9
```

---

### `yield` Keyword

In an `async*` function, `yield` emits a value to the output `Stream` without terminating the function.

```dart
Stream<int> naturalsTo(int n) async* {
  for (int i = 0; i < n; i++) yield i;
}
```

---

### Ticker

Tracks time and calls a callback at every frame (typically 60 FPS). Used internally by `AnimationController`. Requires a `TickerProvider` (via `SingleTickerProviderStateMixin`).

---

### Tween Animation

Defines start and end values; interpolates smoothly over a duration.

```dart
Tween<double>(begin: 0, end: 100).animate(controller);
```

---

### HTML DOM (Dart Web)

Use `dart:html` to interact with the browser DOM — access elements, handle events, modify styles.

```dart
import 'dart:html';
```

---

### Method vs Function

| | Function | Method |
|--|----------|--------|
| Scope | Global or local | Inside a class |
| Access | Only passed parameters | Class fields and properties |
| Call syntax | `greet()` | `person.greet()` |

---

### Synchronous vs Asynchronous

| | Synchronous | Asynchronous |
|--|-------------|--------------|
| Execution | Sequential, blocking | Non-blocking |
| Keyword | — | `async`, `await` |
| Return type | Any | `Future<T>` or `Stream<T>` |

```dart
// Sync
void syncFn() { print("Step 1"); print("Step 2"); }

// Async
Future<void> asyncFn() async {
  print("Step 1");
  await Future.delayed(Duration(seconds: 1));
  print("Step 2");
}
```

---

### Checking Types at Runtime

```dart
print(42.runtimeType); // int
if (value is String) { ... }
if (value is! int) { ... }
```

---

*Study tip: For each section, try explaining the concept aloud in one sentence, then reproduce the code example from memory.*
