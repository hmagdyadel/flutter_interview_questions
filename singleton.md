# Singleton Pattern - Complete Study Guide

## 🎯 What is a Singleton?

A **Singleton** is a design pattern that ensures:
1. **Only one instance** of a class exists in the entire app
2. That instance is **globally accessible**

### Why Use Singleton?
Perfect for shared resources that should have only one instance:
- Database connections
- Shared preferences  
- API clients
- Caching managers
- App settings

---

## 🔧 Three Ways to Implement Singleton in Dart

### Method 1: Basic Singleton (Factory Constructor)
```dart
class AppConfig {
  // 1. Private constructor
  AppConfig._internal();

  // 2. Single static instance
  static final AppConfig _instance = AppConfig._internal();

  // 3. Factory returns same instance
  factory AppConfig() {
    return _instance;
  }

  // Example property
  String baseUrl = "https://api.example.com";
}

void main() {
  var config1 = AppConfig();
  var config2 = AppConfig();

  print(config1 == config2); // true ✅ same instance
}
```

**How it works:**
- `_internal()` is private → cannot be called outside
- `factory AppConfig()` always returns the same `_instance`
- Every call to `AppConfig()` gives you **the same object**

### Method 2: Lazy Singleton (Created When Needed)
```dart
class Database {
  Database._internal();

  static Database? _instance;

  factory Database() {
    _instance ??= Database._internal();
    return _instance!;
  }

  void query(String sql) {
    print("Running query: $sql");
  }
}

void main() {
  var db1 = Database();
  var db2 = Database();

  print(db1 == db2); // true ✅
  db1.query("SELECT * FROM users");
}
```

**Benefit**: Instance is **created only on first use**, not at app startup

### Method 3: Static Instance Access
```dart
class Logger {
  // Static instance accessible directly
  static final Logger instance = Logger._internal();

  Logger._internal();

  void log(String message) {
    print("LOG: $message");
  }
}

void main() {
  Logger.instance.log("App started");
  Logger.instance.log("User logged in");
}
```

**Usage**: Instead of calling `Logger()`, you directly use `Logger.instance`

---

## 📱 Singleton in Flutter Context

### Practical Example: User Session Management
```dart
class UserSession {
  UserSession._internal();

  static final UserSession _instance = UserSession._internal();
  factory UserSession() => _instance;

  String? username;

  void login(String name) {
    username = name;
    print("Logged in as $name");
  }

  void logout() {
    username = null;
    print("Logged out");
  }
}

void main() {
  var session1 = UserSession();
  session1.login("Haitham");

  var session2 = UserSession();
  print(session2.username); // Haitham ✅ same instance
}
```

---

## 🔍 Analyzing Your Code Examples

### Example 1: `firstNonRepeated(String s)`
```dart
String firstNonRepeated(String s) {
  var freq = <String, int>{};
  
  // Pass 1: Count frequencies - O(n)
  for (var c in s.split('')) {
    freq[c] = (freq[c] ?? 0) + 1;
  }
  
  // Pass 2: Find first non-repeated - O(n)
  return freq.entries.firstWhere((e) => e.value == 1).key;
}
```

**Big O Analysis:**
1. `s.split('')` → **O(n)**
2. First loop → **O(n)** (HashMap operations are O(1))
3. `firstWhere` → **O(n)** worst case

**Final Complexity:**
- **Time**: **O(n)** 
- **Space**: **O(n)** (frequency map)

### Example 2: `twoSum(List<int> nums, int target)`
```dart
List<int>? twoSum(List<int> nums, int target) {
  var map = <int, int>{};
  
  for (int i = 0; i < nums.length; i++) {
    int diff = target - nums[i];
    if (map.containsKey(diff)) return [map[diff]!, i];
    map[nums[i]] = i;
  }
  return null;
}
```

**Big O Analysis:**
1. Single loop → **O(n)**
2. HashMap operations inside loop → **O(1)** each
3. Total work per iteration → **O(1)**

**Final Complexity:**
- **Time**: **O(n)**
- **Space**: **O(n)** (map storing seen numbers)

### Algorithm Comparison
| Function            | Best Case | Worst Case | Space | Notes                    |
| ------------------- | --------- | ---------- | ----- | ------------------------ |
| `firstNonRepeated`  | O(n)      | O(n)       | O(n)  | Always two passes        |
| `twoSum`            | O(1)      | O(n)       | O(n)  | Can return early         |

---

## ✅ When to Use Singleton

### Good Use Cases:
- **Configuration** (`AppConfig`)
- **Logging** (`Logger`)
- **Session management** (`UserSession`)
- **Database/Cache** connections
- **API clients** with shared state

### ⚠️ Be Careful:
- **Don't use for UI state** (use Provider/Riverpod/Bloc instead)
- **Testing challenges** (acts like global variables)
- **Overuse makes code harder to maintain**

---

## 🧠 Quick Memory Aid

**Dictionary Analogy:**
- Flip one page at a time → **O(n)**
- Open middle, discard half → **O(log n)**
- Ask friend who memorized everything → **O(1)** 😅

---

## 💡 Key Takeaways for Study

1. **Singleton ensures one instance** across the entire app
2. **Three implementation methods** in Dart (factory, lazy, static)
3. **Use for shared resources** not UI state
4. **Big O analysis helps predict performance** as data scales
5. **Hash maps enable O(n) solutions** for many problems
6. **Trade space for time** - use extra memory for faster algorithms
7. **Choose patterns based on use case** - not every class needs to be singleton