# Big O Notation - Complete Study Guide

## 🎯 What is Big O?

**Big O notation** describes the **time** (or **space**) complexity of an algorithm as the **input size grows**.

### Core Concept
- Ignores constants and small terms
- Focuses only on the **dominant growth rate**
- Answers: *"How will my algorithm scale if I double, triple, or 10x the data size?"*

---

## 📊 Common Big O Complexities

| Big O        | Name          | Example                    | Growth Rate                    |
| ------------ | ------------- | -------------------------- | ------------------------------ |
| **O(1)**     | Constant      | Accessing `list[i]`        | ✅ Fastest – always same time   |
| **O(log n)** | Logarithmic   | Binary Search              | Grows slowly                   |
| **O(n)**     | Linear        | Loop through a list        | Doubles with input             |
| **O(n log n)** | Linearithmic | Merge Sort, QuickSort      | Medium fast                    |
| **O(n²)**    | Quadratic     | Nested loops               | Becomes slow with large n      |
| **O(2^n)**   | Exponential   | Recursive Fibonacci        | 🚨 Very slow                   |
| **O(n!)**    | Factorial     | Permutations/TSP           | 🚨🚨 Impractical               |

---

## 📈 Visual Growth Pattern

**Input size (n) vs Operations:**
- **O(1)**: Always same (constant)
- **O(log n)**: Small increase even with big n
- **O(n)**: Straight line growth
- **O(n²)**: Steep curve (bad for large n)
- **O(2^n), O(n!)**: Explodes 🚀 (only usable for very small n)

---

## 💻 Dart Code Examples

### O(1) – Constant Time
```dart
void main() {
  List<int> nums = [1, 2, 3, 4, 5];
  print(nums[2]); // Always O(1) - direct access
}
```

### O(n) – Linear Time
```dart
void printAll(List<int> nums) {
  for (var num in nums) {
    print(num); // Executes n times
  }
}
```

### O(n²) – Quadratic Time
```dart
void printPairs(List<int> nums) {
  for (var i in nums) {
    for (var j in nums) {
      print('$i, $j'); // Executes n*n times
    }
  }
}
```

### O(log n) – Binary Search
```dart
int binarySearch(List<int> nums, int target) {
  int left = 0, right = nums.length - 1;
  
  while (left <= right) {
    int mid = (left + right) ~/ 2;
    if (nums[mid] == target) return mid;
    if (nums[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return -1; // Not found
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

**Analysis:**
- `s.split('')`: **O(n)**
- First loop: **O(n)** (HashMap operations are O(1))
- `firstWhere`: **O(n)** worst case
- **Final Time Complexity: O(n)**
- **Space Complexity: O(n)** (frequency map)

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

**Analysis:**
- Single loop: **O(n)**
- HashMap operations (`containsKey`, assignment): **O(1)**
- **Final Time Complexity: O(n)**
- **Space Complexity: O(n)** (map storing seen numbers)

### Comparison Table
| Function            | Best Case | Worst Case | Space | Notes                    |
| ------------------- | --------- | ---------- | ----- | ------------------------ |
| `firstNonRepeated`  | O(n)      | O(n)       | O(n)  | Always two passes        |
| `twoSum`            | O(1)      | O(n)       | O(n)  | Can return early         |

---

## 🎯 Big O in Flutter Context

### Performance Examples
- **ListView/GridView**: Using `ListView.builder` (lazy O(1) item build) vs `ListView(children: [...])` (eager O(n) build)
- **Search**: Linear search in list (O(n)) vs binary search on sorted list (O(log n))
- **State Management**: Rebuilding whole widget tree (O(n)) vs only affected widgets with keys (closer to O(1))

---

## 🧠 Quick Memory Aid

**Dictionary Search Analogy:**
- Flip one page at a time → **O(n)**
- Open middle, discard half → **O(log n)**
- Ask friend who memorized everything → **O(1)** 😅

---

## 💡 Key Takeaways for Study

1. **Big O predicts performance** as data grows
2. **Focus on worst-case scenarios** for reliable estimates
3. **Ignore constants** - O(2n) becomes O(n)
4. **Space vs Time tradeoffs** - often use extra memory for speed
5. **Hash maps are powerful** - O(1) lookups enable many O(n) solutions
6. **Choose algorithms wisely** - O(n²) vs O(n) makes huge difference at scale
7. **Flutter optimization** depends on understanding these patterns