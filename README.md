
# Java OCP 21 Flashcards 

---

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
    String role = "Child";
    static String familyName() { return "Johnson"; }
    String introduce() { return "I am a Child"; }
}

Parent member = new Child();
System.out.println(member.role); // Parent
System.out.println(member.familyName()); // Smith
System.out.println(member.introduce()); // I am a Child
```

---

### 🃏 Up-bounded vs Down-bounded Wildcards

- `? extends Number`: Can **read** items of type Number or its subtypes. Cannot add anything (except `null`).
- `? super Number`: Can **write** Number or its subtypes. Cannot safely read (except `Object`).

```java
List<? extends Number> numbers = List.of(1, 2.0);
// numbers.add(3); // Compile error

List<? super Number> values = new ArrayList<Object>();
values.add(10); // OK
// Number n = values.get(0); // Compile error
```

---

### 🃏 `super()` Constructor Rule

If a constructor does not explicitly call `super()`, the compiler inserts one **only if the superclass has a no-arg constructor**.

```java
class Ancestor {
    Ancestor(String msg) {}
}

class Parent extends Ancestor {
    // Compile error: no explicit super(), and no no-arg constructor in Ancestor
}
```

---

### 🃏 `equals()` behaves like `==`

When a class **does not override** `equals()` from `Object`, `.equals()` compares **references**, just like `==`.

```java
class FamilyName {}
FamilyName a = new FamilyName();
FamilyName b = new FamilyName();
System.out.println(a.equals(b)); // false
```

---

### 🃏 Pattern Matching `switch` with Guarded Cases

```java
static void describe(Object obj) {
    switch (obj) {
        case String s when s.length() > 3 -> System.out.println("Long String");
        case String s -> System.out.println("Short String");
        default -> System.out.println("Not a String");
    }
}
```

If **none** of the guards match, and there's no fallback, Java throws `MatchException`.

---

### 🃏 Serialization and `transient` Fields

- `transient` fields are **not serialized**.
- On deserialization, JVM invokes **first non-serializable superclass**'s **no-arg constructor**.

```java
class Grandparent {
    Grandparent() { System.out.println("Grandparent ctor"); }
}

class Parent extends Grandparent implements Serializable {
    transient String name = "John";
}
```

---

### 🃏 Interface vs Abstract Class: Abstract Methods

- Interface: A method without body is **implicitly abstract**.
- Abstract Class: A method without body must be **explicitly marked abstract**.

```java
interface Talker {
    void speak(); // implicitly abstract
}

abstract class Human {
    abstract void walk(); // must specify abstract
}
```

---

### 🃏 Try-With-Resources and Suppressed Exceptions

The **exception in the try block is primary**. Exceptions thrown by `close()` are **suppressed**.

```java
try (MyResource res = new MyResource()) {
    throw new RuntimeException("Primary");
} catch (Exception e) {
    System.out.println(e.getMessage()); // Primary
    for (Throwable t : e.getSuppressed()) {
        System.out.println(t); // Suppressed: from close()
    }
}
```

---

### 🃏 `protected` Access Across Packages

- Same package: accessible anywhere.
- Different package: only accessible from **subclass**, and only via **subclass reference**.

```java
package family;

public class Parent {
    protected void guide() {}
}

package extended;

public class Child extends family.Parent {
    void test() {
        guide(); // OK
        Parent p = new Parent();
        // p.guide(); // Not allowed
    }
}
```

---

### 🃏 `Files.mismatch(path1, path2)`

Compares two files byte by byte:

- Returns index of first mismatching byte.
- Returns `-1` if files are identical.
- Throws `IOException` if paths are invalid.

```java
Path path1 = Path.of("photo1.jpg");
Path path2 = Path.of("photo2.jpg");

long diff = Files.mismatch(path1, path2);
System.out.println(diff); // -1 if same, otherwise index
```

---
