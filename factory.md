# Dart Factory Constructors - Complete Study Guide

## 🎯 What is a Factory Constructor in Dart/Flutter?

A **factory constructor** gives you control over object creation - it doesn't always create a new instance like normal constructors do.

### Key Difference from Normal Constructors
- **Normal constructor**: Always creates a new object when called
- **Factory constructor**: Can return a new object, cached object, or even a different class instance

### Why Use Factory Constructors?
- **Reuse existing instances** (singleton pattern)
- **Return subtypes** instead of the main class
- **Add custom logic** before object creation
- **Implement caching** to improve performance

---

## 🔧 Factory Constructor Examples

### Example 1: Normal vs Factory Constructor

#### Normal Constructor (Always Creates New)
```dart
class User {
  final String name;

  // Normal constructor
  User(this.name);
}

void main() {
  var u1 = User("Alice");
  var u2 = User("Alice");

  print(u1 == u2); // false (different objects)
}
```

#### Factory Constructor (With Caching)
```dart
class User {
  final String name;

  static final Map<String, User> _cache = {};

  // Factory constructor
  factory User(String name) {
    if (_cache.containsKey(name)) {
      return _cache[name]!; // return cached
    } else {
      final user = User._internal(name);
      _cache[name] = user;
      return user;
    }
  }

  // Private real constructor
  User._internal(this.name);
}

void main() {
  var u1 = User("Alice");
  var u2 = User("Alice");

  print(u1 == u2); // true (same object, cached)
}
```

✅ **Result**: Factory constructor controls object creation and reuses existing instances

### Example 2: Singleton Pattern
```dart
class AppConfig {
  static final AppConfig _instance = AppConfig._internal();

  // Private constructor
  AppConfig._internal();

  // Factory always returns same instance
  factory AppConfig() {
    return _instance;
  }

  String theme = "light";
}

void main() {
  var c1 = AppConfig();
  var c2 = AppConfig();

  c1.theme = "dark";

  print(c2.theme); // dark (same object!)
}
```

✅ **Use case**: Perfect for app configuration, services, or dependency injection

### Example 3: Returning Different Subtypes
```dart
abstract class Shape {
  void draw();

  factory Shape(String type) {
    if (type == "circle") return Circle();
    if (type == "square") return Square();
    throw "Unknown shape";
  }
}

class Circle implements Shape {
  @override
  void draw() => print("Drawing Circle");
}

class Square implements Shape {
  @override
  void draw() => print("Drawing Square");
}

void main() {
  Shape s1 = Shape("circle");
  Shape s2 = Shape("square");

  s1.draw(); // Drawing Circle
  s2.draw(); // Drawing Square
}
```

✅ **Result**: Factory returns different classes based on input parameters

---

## 📋 Quick Comparison Table

| Feature                     | Normal Constructor | Factory Constructor   |
| --------------------------- | ------------------ | --------------------- |
| Always creates new instance | ✅ Yes              | ❌ Not always          |
| Can return cached instance  | ❌ No               | ✅ Yes                 |
| Can return subtype          | ❌ No               | ✅ Yes                 |
| Syntax                      | `ClassName()`      | `factory ClassName()` |

---

## 🚀 When to Use Factory Constructors in Flutter

1. **Singleton services** (e.g., SharedPreferences, FirebaseAuth)
2. **Cached objects** instead of creating new ones repeatedly
3. **Different subtypes** depending on conditions
4. **Custom initialization logic** before returning an object

---

## 💡 Key Takeaways for Study

1. **Factory constructors don't always create new objects** - they give you control
2. **Use `factory` keyword** to define them
3. **Common patterns**: Singleton, caching, and subtype selection
4. **Performance benefit**: Avoid unnecessary object creation through caching
5. **Flexibility**: Can return any instance that matches the return type
6. **Real-world usage**: Essential for services and state management in Flutter apps