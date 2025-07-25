# Java OCP 21 Flashcards 

### 🃏 Instance Methods vs Variables and Static Methods

**Rule:** Instance methods are **overridden**, while **variables and static methods are hidden**.

- The method invoked depends on the **actual object type** (runtime).
- The field accessed depends on the **reference type** (compile-time).

```java
class Parent {
    String role = "Parent";
    static String familyName() { return "Smith"; }
    String introduce() { return "I am a Parent"; }
}

class Child extends Parent {
    String role = "Child";              // Field hiding
    static String familyName() { return "Johnson"; }  // Method hiding
    String introduce() { return "I am a Child"; }     // Method overriding
}

Parent member = new Child();
System.out.println(member.role);        // Parent (field access - compile-time)
System.out.println(member.familyName()); // Smith (static method - compile-time)
System.out.println(member.introduce());  // I am a Child (instance method - runtime)
```

**💡 Learning Tip:** Remember "HIDE vs OVERRIDE" - static methods and fields are HIDDEN (reference type matters), instance methods are OVERRIDDEN (object type matters).

**Q:** Does overriding a method replace the original method call even if the reference is of parent type?  
**A:** Yes — overridden instance methods use the object type at runtime (dynamic dispatch). Static methods use the reference type (they are hidden, not overridden).

---

### 🃏 Up-bounded vs Down-bounded Wildcards

**PECS Rule:** **P**roducer **E**xtends, **C**onsumer **S**uper

- `? extends Number`: **READ-ONLY** - Can read items of type Number or its subtypes. Cannot add anything (except `null`).
- `? super Number`: **WRITE-ONLY** - Can write Number or its subtypes. Cannot safely read (except `Object`).

```java
// Producer Extends - Reading from a collection
List<? extends Number> numbers = List.of(1, 2.0);
Number n = numbers.get(0);    // OK - can read as Number
// numbers.add(3);            // ❌ Compile error - cannot write

// Consumer Super - Writing to a collection  
List<? super Number> values = new ArrayList<Object>();
values.add(10);       // ✅ OK - can write Number/subtypes
values.add(3.14);     // ✅ OK - can write Number/subtypes
// Number n = values.get(0);  // ❌ Compile error - can only read as Object
Object obj = values.get(0);   // ✅ OK - can read as Object
```

**💡 Learning Tip:** Think of wildcards as "one-way streets" - extends for reading OUT, super for writing IN.

---

### 🃏 `super()` Constructor Rule

**Rule:** If a constructor does not explicitly call `super()`, the compiler inserts `super()` **only if the superclass has a no-arg constructor**.

```java
class Ancestor {
    Ancestor(String msg) {
        System.out.println("Ancestor: " + msg);
    }
    // No no-arg constructor available!
}

class Parent extends Ancestor {
    // ❌ This would cause compile error:
    // Parent() {} // Implicit super() call fails
    
    // ✅ Must explicitly call super with argument:
    Parent() {
        super("Default parent message"); // Explicit call required
    }
    
    Parent(String name) {
        super("Parent: " + name);       // Explicit call required
    }
}
```

**💡 Learning Tip:** "No free lunch" - if parent needs arguments, children must provide them explicitly.

**Q:** Does Java insert a `super()` call if you don't write one?  
**A:** Yes, only if the superclass has a no-arg constructor. Otherwise, you must explicitly call a matching constructor.

---

### 🃏 `equals()` Behaves Like `==`

When a class **does not override** `equals()` from `Object`, `.equals()` compares **references**, just like `==`.

```java
class FamilyName {
    String name;
    FamilyName(String name) { this.name = name; }
    // No equals() override - inherits Object.equals()
}

FamilyName a = new FamilyName("Smith");
FamilyName b = new FamilyName("Smith");
FamilyName c = a;

System.out.println(a.equals(b));  // false - different objects
System.out.println(a == b);       // false - different objects  
System.out.println(a.equals(c));  // true - same reference
System.out.println(a == c);       // true - same reference

// Compare with String (which DOES override equals):
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1.equals(s2)); // true - content comparison
System.out.println(s1 == s2);      // false - different objects
```

**💡 Learning Tip:** Classes that don't override equals() are doing reference comparison. StringBuilder is a famous example!

**Q:** When does `.equals()` behave exactly like `==`?  
**A:** When the class does not override `equals()` from `Object`. Examples: `StringBuilder`, or any custom class that inherits `Object.equals()`.

---

### 🃏 Pattern Matching `switch` with Guarded Cases

```java
static void describe(Object obj) {
    switch (obj) {
        case String s when s.length() > 3 -> System.out.println("Long String: " + s);
        case String s -> System.out.println("Short String: " + s);
        case Integer i when i > 10 -> System.out.println("Big number: " + i);
        case Integer i -> System.out.println("Small number: " + i);
        default -> System.out.println("Something else: " + obj.getClass());
    }
}

// Testing the method:
describe("hi");      // Short String: hi
describe("hello");   // Long String: hello  
describe(5);         // Small number: 5
describe(15);        // Big number: 15
describe(3.14);      // Something else: class java.lang.Double
```

**Dangerous example - without default:**
```java
static String categorize(Object obj) {
    return switch (obj) {
        case String s when s.startsWith("A") -> "A-String";
        case String s when s.startsWith("B") -> "B-String";
        // ❌ What if string starts with "C"? MatchException at runtime!
    };
}
```

**💡 Learning Tip:** Guarded patterns are checked in order. Always have a fallback case or default to avoid MatchException.

**Q:** What happens if a `switch` expression has only guarded cases and none match at runtime?  
**A:** ✅ It compiles, but at runtime, it throws a `MatchException` if no case matches and there's no default clause.

---

### 🃏 Serialization and `transient` Fields

- `transient` fields are **not serialized** - they become `null` (objects) or default values (primitives).
- On deserialization, JVM invokes **first non-serializable superclass**'s **no-arg constructor**.

```java
import java.io.*;

class Grandparent {
    String grandparentField = "GP";
    Grandparent() { 
        System.out.println("Grandparent constructor called during deserialization"); 
    }
}

class Parent extends Grandparent implements Serializable {
    String parentField = "Parent";
    transient String secretPassword = "secret123";  // Won't be serialized
    transient int tempValue = 42;                   // Won't be serialized
    
    // Constructor won't be called during deserialization (class is Serializable)
    Parent() { 
        System.out.println("Parent constructor - only called for new objects"); 
    }
}

// After deserialization:
// - parentField = "Parent" ✅ (serialized)  
// - secretPassword = null ❌ (transient)
// - tempValue = 0 ❌ (transient, default int value)
// - grandparentField = "GP" ✅ (set by Grandparent constructor)
```

**💡 Learning Tip:** "Transient = Temporary" - these fields don't survive the serialization journey.

---

### 🃏 Interface vs Abstract Class: Abstract Methods

- **Interface:** A method without body is **implicitly abstract** (and public).
- **Abstract Class:** A method without body must be **explicitly marked abstract**.

```java
interface Talker {
    void speak();                    // Implicitly: public abstract void speak();
    default void whisper() {         // Default methods allowed
        System.out.println("...");
    }
    static void shout() {            // Static methods allowed  
        System.out.println("HELLO!");
    }
}

abstract class Human {
    abstract void walk();            // Must explicitly declare abstract
    
    // ❌ This would be a compile error:
    // void run();  // Missing body and no 'abstract' keyword
    
    void breathe() {                 // Concrete methods allowed
        System.out.println("Inhale, exhale");
    }
}
```

**💡 Learning Tip:** Interfaces assume you want abstract methods, abstract classes make you be explicit.

---

### 🃏 Try-With-Resources and Suppressed Exceptions

The **exception in the try block is primary**. Exceptions thrown by `close()` are **suppressed** and attached to the primary exception.

```java
class MyResource implements AutoCloseable {
    private String name;
    
    MyResource(String name) { this.name = name; }
    
    void doWork() throws Exception {
        throw new RuntimeException("Work failed in " + name);
    }
    
    @Override
    public void close() throws Exception {
        throw new RuntimeException("Close failed for " + name);
    }
}

// Example usage:
try (MyResource res = new MyResource("Database")) {
    res.doWork();  // Throws primary exception
    // close() will be called automatically and its exception suppressed
} catch (Exception e) {
    System.out.println("Primary: " + e.getMessage());  // Work failed in Database
    
    // Check suppressed exceptions:
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println("Suppressed: " + suppressed.getMessage()); // Close failed for Database
    }
}
```

**Multiple resources example:**
```java
try (MyResource r1 = new MyResource("DB1");
     MyResource r2 = new MyResource("DB2")) {
    // Resources closed in reverse order: r2.close(), then r1.close()
    throw new RuntimeException("Business logic error");
} catch (Exception e) {
    // Primary: Business logic error
    // Suppressed: Close failed for DB2, Close failed for DB1
}
```

**💡 Learning Tip:** Primary exception is the "star of the show" - suppressed exceptions are the "supporting cast."

---

### 🃏 `protected` Access Across Packages

- **Same package:** accessible anywhere.
- **Different package:** only accessible from **subclass**, and only via **subclass reference** (not parent reference).

```java
// File: family/Parent.java
package family;

public class Parent {
    protected void guide() { System.out.println("Parent guidance"); }
    protected String advice = "Listen to your parents";
}

// File: extended/Child.java  
package extended;
import family.Parent;

public class Child extends Parent {
    void test() {
        // ✅ Accessing through subclass (this):
        guide();                    // OK - implicit this.guide()
        this.guide();              // OK - explicit this
        System.out.println(advice); // OK - inherited field
        
        // ✅ Accessing through subclass reference:
        Child child = new Child();
        child.guide();             // OK - subclass reference
        
        // ❌ Accessing through parent reference (different package):
        Parent parent = new Parent();
        // parent.guide();         // Compile error!
        // parent.advice;          // Compile error!
        
        // ✅ But this works (casting):
        Parent parentRef = new Child();
        // parentRef.guide();      // Still compile error - reference type matters
    }
}

// File: extended/Sibling.java
package extended;

public class Sibling extends family.Parent {
    void testAccess() {
        Child child = new Child();
        // child.guide();          // ❌ Compile error! Can't access Child's protected via different subclass
        
        guide();                   // ✅ OK - accessing own inherited method
    }
}
```

**💡 Learning Tip:** Protected across packages = "Family only, and only through your own family line."

---

### 🃏 `Files.mismatch(path1, path2)`

Compares two files **byte by byte**:
- Returns **index of first mismatching byte** (0-based).
- Returns **-1** if files are identical.
- Throws `IOException` if paths are invalid or inaccessible.

```java
import java.nio.file.*;
import java.io.IOException;

try {
    Path file1 = Path.of("document1.txt");  // Content: "Hello World"
    Path file2 = Path.of("document2.txt");  // Content: "Hello Mars"
    Path file3 = Path.of("document3.txt");  // Content: "Hello World"
    
    long result1 = Files.mismatch(file1, file2);  // Returns 6 (index of 'W' vs 'M')
    long result2 = Files.mismatch(file1, file3);  // Returns -1 (identical)
    
    System.out.println("Mismatch at byte: " + result1);  // 6
    System.out.println("Files identical: " + (result2 == -1));  // true
    
} catch (IOException e) {
    System.out.println("Error reading files: " + e.getMessage());
}
```

**Practical use cases:**
```java
// Check if files are identical:
boolean areIdentical = Files.mismatch(path1, path2) == -1;

// Find first difference position:
long diffPos = Files.mismatch(path1, path2);
if (diffPos != -1) {
    System.out.println("Files differ starting at byte " + diffPos);
}
```

**💡 Learning Tip:** Mismatch = "Find the first difference" (-1 means no differences found).

---

### 🃏 Pre-decrement vs Post-decrement in For Loop

**Q:** In a for loop, what's the difference between `--i` and `i--` in the update expression?  
**A:** **No difference** — the returned value of the update expression is ignored. Both decrement `i` after the loop body runs.

```java
// These are functionally identical in for loops:
for (int i = 5; i > 0; i--) {           // Post-decrement
    System.out.print(i + " ");          // Prints: 5 4 3 2 1
}

for (int i = 5; i > 0; --i) {           // Pre-decrement  
    System.out.print(i + " ");          // Prints: 5 4 3 2 1
}

// The difference matters when you use the return value:
int a = 5;
int b = a--;    // b = 5, a = 4 (post: return then decrement)
int c = --a;    // c = 3, a = 3 (pre: decrement then return)

// But in for loops, the return value is discarded:
for (int i = 5; i > 0; /* return value ignored */ i--) { }
```

**💡 Learning Tip:** For loop update expressions are "fire and forget" - the return value doesn't matter.

---

### 🃏 Arrays.binarySearch() and Negative Results

Binary search requires a **sorted array**. Returns:
- **Positive index** if element found
- **Negative value** if not found: `-(insertion point) - 1`

```java
int[] sortedArray = {10, 20, 30, 40, 50};

// Element found:
int found = Arrays.binarySearch(sortedArray, 30);    // Returns 2
System.out.println("Found at index: " + found);

// Element not found:
int notFound = Arrays.binarySearch(sortedArray, 25); // Returns -3
System.out.println("Not found, result: " + notFound);

// Calculate insertion point:
int insertionPoint = -notFound - 1;  // -(-3) - 1 = 2
System.out.println("Would insert at index: " + insertionPoint);

// Verify: 25 would go at index 2 to maintain sorted order
// [10, 20, ?, 30, 40, 50] -> [10, 20, 25, 30, 40, 50]
//          ^
//      index 2
```

**More examples:**
```java
int[] arr = {1, 3, 5, 7, 9};

Arrays.binarySearch(arr, 0);   // -1  (would insert at index 0)
Arrays.binarySearch(arr, 2);   // -2  (would insert at index 1) 
Arrays.binarySearch(arr, 4);   // -3  (would insert at index 2)
Arrays.binarySearch(arr, 10);  // -6  (would insert at index 5)
```

**💡 Learning Tip:** "Negative means missing" - use the formula `-(result) - 1` to find where it would go.

---

### 🃏 Arrays.compare() vs Arrays.mismatch()

Both methods compare arrays, but return different information:

**Arrays.compare()** - Lexicographic comparison:
- **0** if arrays are equal
- **< 0** if first array is lexicographically less  
- **> 0** if first array is lexicographically greater

**Arrays.mismatch()** - Find difference location:
- **-1** if arrays are identical
- **index** of first differing element

```java
int[] a = {1, 2, 3, 4};
int[] b = {1, 2, 3, 4};
int[] c = {1, 2, 5, 4};
int[] d = {1, 2, 3};        // shorter

// Equal arrays:
System.out.println(Arrays.compare(a, b));   // 0 (equal)
System.out.println(Arrays.mismatch(a, b));  // -1 (no mismatch)

// Different elements:
System.out.println(Arrays.compare(a, c));   // -2 (3 < 5, so negative)
System.out.println(Arrays.mismatch(a, c));  // 2 (differ at index 2)

// Different lengths:
System.out.println(Arrays.compare(a, d));   // 1 (longer array is "greater")
System.out.println(Arrays.mismatch(a, d));  // 3 (differ at index 3 - length difference)
```

**Lexicographic comparison examples:**
```java
String[] words1 = {"apple", "banana"};
String[] words2 = {"apple", "cherry"};

Arrays.compare(words1, words2);  // negative ("banana" < "cherry")
Arrays.mismatch(words1, words2); // 1 (differ at index 1)
```

**💡 Learning Tip:** compare() tells you "who wins", mismatch() tells you "where they differ."

---

### 🃏 StringBuilder and Reference Reassignment

Java is **pass-by-value** for references. You get a copy of the reference, not the reference itself.

```java
public class StringBuilderExample {
    static void modifyContent(StringBuilder sb) {
        sb.append(" modified");     // ✅ Modifies the object - caller sees this
        System.out.println("Inside method after append: " + sb);
    }
    
    static void reassignReference(StringBuilder sb) {
        sb.append(" first");        // ✅ Modifies original object
        sb = new StringBuilder("completely new");  // ❌ Only changes local copy of reference
        sb.append(" content");      // ❌ Modifies the new object, not original
        System.out.println("Inside method after reassign: " + sb);
    }
    
    public static void main(String[] args) {
        StringBuilder original = new StringBuilder("start");
        
        modifyContent(original);
        System.out.println("After modifyContent: " + original);  // "start modified"
        
        reassignReference(original);  
        System.out.println("After reassignReference: " + original);  // "start modified first"
        // Note: "completely new content" is lost!
    }
}
```

**Visual representation:**
```java
// Before method call:
// original -----> [StringBuilder: "start"]

// Inside reassignReference after reassignment:
// original -----> [StringBuilder: "start modified first"]  (caller's object)  
// sb -----> [StringBuilder: "completely new content"]       (method's new object)

// After method returns:
// original -----> [StringBuilder: "start modified first"]  (unchanged reference)
// The "completely new content" object is eligible for garbage collection
```

**💡 Learning Tip:** You can change the object's content through the reference, but you can't change where the original reference points.

---

### 🃏 Stream Collectors: partitioningBy and Function.identity()

**Collectors.partitioningBy()** - Always creates exactly **2 groups** based on a boolean predicate:

```java
import java.util.stream.*;
import java.util.*;
import java.util.function.Function;

List<String> words = List.of("a", "bb", "ccc", "dddd", "e");

// Partition by length > 2:
Map<Boolean, List<String>> byLength = words.stream()
    .collect(Collectors.partitioningBy(word -> word.length() > 2));

System.out.println(byLength);
// {false=[a, bb, e], true=[ccc, dddd]}

// Always has both keys (true/false), even if one group is empty:
List<String> longWords = List.of("hello", "world");
Map<Boolean, List<String>> result = longWords.stream()
    .collect(Collectors.partitioningBy(word -> word.length() < 3));
// {false=[hello, world], true=[]}  // true group is empty but present
```

**Function.identity()** - Returns a function that returns its input unchanged:

```java
// These are equivalent:
Function.identity()           // Method reference
x -> x                       // Lambda expression  
Function.<String>identity()  // With explicit type

// Common usage - as key mapper in toMap():
List<String> fruits = List.of("apple", "banana", "cherry");

// Map each string to its uppercase version:
Map<String, String> fruitMap = fruits.stream()
    .collect(Collectors.toMap(
        Function.identity(),    // key = original string
        String::toUpperCase     // value = uppercase string
    ));
// {apple=APPLE, banana=BANANA, cherry=CHERRY}

// Map each string to its length:
Map<String, Integer> lengthMap = fruits.stream()
    .collect(Collectors.toMap(
        Function.identity(),    // key = original string  
        String::length         // value = length
    ));
// {apple=5, banana=6, cherry=6}
```

**Comparison with groupingBy():**
```java
// partitioningBy - exactly 2 groups (boolean):
Map<Boolean, List<String>> partitioned = words.stream()
    .collect(Collectors.partitioningBy(w -> w.length() > 2));

// groupingBy - multiple groups (any classifier):
Map<Integer, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(String::length));
// {1=[a, e], 2=[bb], 3=[ccc], 4=[dddd]}
```

**💡 Learning Tip:** partitioningBy = "split in half", groupingBy = "organize by category", identity = "keep as-is".

---

### 🃏 Stream Operations: Terminal vs Intermediate

**Intermediate Operations** - Return a new Stream (lazy evaluation):
```java
Stream<String> words = Stream.of("apple", "banana", "cherry");

Stream<String> filtered = words
    .filter(s -> s.startsWith("a"))     // Intermediate
    .map(String::toUpperCase)           // Intermediate  
    .limit(2);                          // Intermediate
    
// Nothing executed yet - streams are lazy!
```

**Terminal Operations** - Return a result and close the stream:
```java
List<String> result = Stream.of("apple", "banana", "cherry")
    .filter(s -> s.length() > 5)       // Intermediate
    .collect(Collectors.toList());      // Terminal - execution happens here

// Other terminal operations:
long count = stream.count();                    // Terminal
Optional<String> first = stream.findFirst();   // Terminal  
stream.forEach(System.out::println);           // Terminal
boolean anyMatch = stream.anyMatch(s -> s.startsWith("a")); // Terminal
```

**Stream Reuse Error:**
```java
Stream<String> stream = Stream.of("a", "b", "c");

stream.forEach(System.out::print);  // Terminal operation - stream is consumed
// stream.count();                  // ❌ IllegalStateException: stream has already been operated upon

// Solution - create a new stream:
List<String> data = List.of("a", "b", "c");
data.stream().forEach(System.out::print);  // First stream
long count = data.stream().count();         // Second stream - OK
```

**Debugging tip - peek() for intermediate inspection:**
```java
List<String> result = Stream.of("apple", "Banana", "cherry")
    .peek(s -> System.out.println("Original: " + s))        // Intermediate - for debugging
    .filter(s -> s.length() > 5)
    .peek(s -> System.out.println("After filter: " + s))    // Intermediate - for debugging
    .map(String::toUpperCase)
    .peek(s -> System.out.println("After map: " + s))       // Intermediate - for debugging
    .collect(Collectors.toList());                           // Terminal
```

**💡 Learning Tip:** Intermediate = "keep the pipeline flowing", Terminal = "time for results".

---

### 🃏 Stream + Optional Exception Handling

Understanding when Optional methods throw exceptions:

```java
private static void demonstrateOptionalExceptions() {
    // Safe stream that produces a result:
    Stream<Integer> numbers = Stream.of(1, 3, 7, 2, 8);
    Optional<Integer> maxOpt = numbers
        .filter(x -> x < 10)           // All numbers pass
        .limit(3)                      // Take first 3: [1, 3, 7]
        .max(Integer::compareTo);      // Find max: Optional[7]
    
    System.out.println(maxOpt.get()); // ✅ 7 - safe because Optional has value
    
    // Dangerous stream that produces empty Optional:
    Stream<Integer> emptyStream = Stream.of(15, 20, 25);
    Optional<Integer> emptyOpt = emptyStream
        .filter(x -> x < 5)           // No numbers pass filter
        .limit(3)                     // Still empty
        .max(Integer::compareTo);     // Returns Optional.empty()
    
    // System.out.println(emptyOpt.get()); // ❌ NoSuchElementException!
    
    // Safe alternatives:
    System.out.println(emptyOpt.orElse(-1));              // -1 (default value)
    System.out.println(emptyOpt.orElseGet(() -> 0));      // 0 (computed default)
    emptyOpt.ifPresent(System.out::println);              // Does nothing if empty
    
    if (emptyOpt.isPresent()) {                           // Check before get()
        System.out.println(emptyOpt.get());
    }
}
```

**Comparator examples - both correct approaches:**
```java
// Method 1: Integer::compareTo (recommended)
Optional<Integer> max1 = stream.max(Integer::compareTo);

// Method 2: Lambda comparator  
Optional<Integer> max2 = stream.max((x, y) -> x.compareTo(y));

// Method 3: Comparator.naturalOrder()
Optional<Integer> max3 = stream.max(Comparator.naturalOrder());

// Method 4: Simple lambda (works but less clear for complex types)
Optional<Integer> max4 = stream.max((x, y) -> x - y);  // Can overflow!
```

**Custom object comparison:**
```java
class Person {
    String name;
    int age;
    
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

List<Person> people = List.of(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 20)
);

// Find oldest person:
Optional<Person> oldest = people.stream()
    .max(Comparator.comparing(Person::getAge));        // Method reference
    // .max((p1, p2) -> p1.age - p2.age);             // Lambda alternative

oldest.ifPresent(p -> System.out.println("Oldest: " + p.name)); // Bob
```

**💡 Learning Tip:** Optional.get() = "Russian roulette" - always check isPresent() or use orElse()/ifPresent() for safety.
