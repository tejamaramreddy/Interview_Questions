# Java 8 Streams — Object-Based Interview Questions & Solutions

## Common `Employee` Class

All examples below assume this class:

```java
public class Employee {

    private int id;
    private String name;
    private String department;
    private double salary;
    private int age;

    public Employee(int id, String name, String department,
                    double salary, int age) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getDepartment() {
        return department;
    }

    public double getSalary() {
        return salary;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return name + " - " + department + " - " + salary;
    }
}
```

Sample data:

```java
List<Employee> employees = Arrays.asList(
    new Employee(1, "John", "IT", 70000, 30),
    new Employee(2, "Alice", "HR", 50000, 28),
    new Employee(3, "Bob", "IT", 90000, 35),
    new Employee(4, "David", "Finance", 60000, 40),
    new Employee(5, "Emma", "HR", 55000, 32),
    new Employee(6, "Mike", "IT", 75000, 26),
    new Employee(7, "Sophia", "Finance", 80000, 29)
);
```

Imports:

```java
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;
```

---

# 🟢 Basic Questions

## 1. Find all employees whose salary is greater than 60,000

```java
List<Employee> result = employees.stream()
        .filter(e -> e.getSalary() > 60000)
        .toList();
```

Output:

```text
John
Bob
Mike
Sophia
```

### Key concept

`filter()` is used when we want to select elements based on a condition.

---

## 2. Find all employees from the IT department

```java
List<Employee> result = employees.stream()
        .filter(e -> e.getDepartment().equals("IT"))
        .toList();
```

Output:

```text
John
Bob
Mike
```

---

## 3. Get a list containing only employee names

```java
List<String> names = employees.stream()
        .map(Employee::getName)
        .toList();
```

Output:

```text
[John, Alice, Bob, David, Emma, Mike, Sophia]
```

### Key concept

`map()` transforms one object into another object/value.

---

## 4. Find the number of employees

```java
long count = employees.stream()
        .count();
```

Output:

```text
7
```

---

## 5. Find employees whose age is greater than 30

```java
List<Employee> result = employees.stream()
        .filter(e -> e.getAge() > 30)
        .toList();
```

Output:

```text
Bob
David
Emma
```

---

## 6. Sort employees by salary in ascending order

```java
List<Employee> result = employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary))
        .toList();
```

Output:

```text
Alice   50000
Emma    55000
David   60000
John    70000
Mike    75000
Sophia  80000
Bob     90000
```

---

## 7. Sort employees by salary in descending order

```java
List<Employee> result = employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).reversed())
        .toList();
```

Output:

```text
Bob     90000
Sophia  80000
Mike    75000
John    70000
David   60000
Emma    55000
Alice   50000
```

---

## 8. Find the employee with the highest salary

```java
Optional<Employee> result = employees.stream()
        .max(Comparator.comparing(Employee::getSalary));
```

To get the employee:

```java
result.ifPresent(System.out::println);
```

Output:

```text
Bob
```

### Alternative

```java
Employee result = employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).reversed())
        .findFirst()
        .orElse(null);
```

`max()` is preferable because sorting the entire list is unnecessary.

---

## 9. Find the employee with the lowest salary

```java
Optional<Employee> result = employees.stream()
        .min(Comparator.comparing(Employee::getSalary));
```

Output:

```text
Alice
```

---

## 10. Find the average salary

```java
double averageSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0.0);
```

Output:

```text
67142.86
```

### Key concept

Use `mapToDouble()` when converting objects into primitive `double` values for numeric operations.

---

# 🟡 Intermediate Questions

## 11. Find the highest-paid employee from the IT department

```java
Optional<Employee> result = employees.stream()
        .filter(e -> e.getDepartment().equals("IT"))
        .max(Comparator.comparing(Employee::getSalary));
```

Output:

```text
Bob
```

---

## 12. Find the average salary of employees in IT

```java
double averageSalary = employees.stream()
        .filter(e -> e.getDepartment().equals("IT"))
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0.0);
```

Output:

```text
78333.33
```

---

## 13. Find the number of employees in each department

```java
Map<String, Long> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
        ));
```

Output:

```text
IT       = 3
HR       = 2
Finance  = 2
```

### Key concept

```java
groupingBy()
```

groups objects based on a property.

```java
counting()
```

counts the objects in each group.

---

## 14. Find the average salary for each department

```java
Map<String, Double> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
        ));
```

Output:

```text
IT       = 78333.33
HR       = 52500.00
Finance  = 70000.00
```

---

## 15. Find the highest-paid employee in each department

```java
Map<String, Optional<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.maxBy(
                        Comparator.comparing(Employee::getSalary)
                )
        ));
```

Output:

```text
IT       -> Bob
HR       -> Emma
Finance  -> Sophia
```

### Cleaner version

If you want `Map<String, Employee>` instead of `Map<String, Optional<Employee>>`:

```java
Map<String, Employee> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                        Collectors.maxBy(
                                Comparator.comparing(Employee::getSalary)
                        ),
                        Optional::get
                )
        ));
```

---

## 16. Find the lowest-paid employee in each department

```java
Map<String, Employee> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                        Collectors.minBy(
                                Comparator.comparing(Employee::getSalary)
                        ),
                        Optional::get
                )
        ));
```

Output:

```text
IT       -> John
HR       -> Alice
Finance  -> David
```

---

## 17. Find the second-highest salary

```java
Optional<Double> result = employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .findFirst();
```

Output:

```text
80000
```

### Important

`distinct()` makes this the **second-highest distinct salary**.

---

## 18. Find the second-highest-paid employee

```java
Optional<Employee> result = employees.stream()
        .sorted(
                Comparator.comparing(Employee::getSalary)
                        .reversed()
        )
        .skip(1)
        .findFirst();
```

For duplicate salaries, this returns the second employee in the sorted sequence, not necessarily the employee with the second-highest **distinct** salary.

For the distinct-salary interpretation:

```java
Optional<Employee> result = employees.stream()
        .sorted(
                Comparator.comparing(Employee::getSalary)
                        .reversed()
        )
        .collect(Collectors.groupingBy(
                Employee::getSalary,
                LinkedHashMap::new,
                Collectors.toList()
        ))
        .values()
        .stream()
        .skip(1)
        .flatMap(Collection::stream)
        .findFirst();
```

---

## 19. Find the second-highest salary in each department

```java
Map<String, Optional<Double>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(
                        Employee::getSalary,
                        Collectors.collectingAndThen(
                                Collectors.toSet(),
                                salaries -> salaries.stream()
                                        .sorted(Comparator.reverseOrder())
                                        .skip(1)
                                        .findFirst()
                        )
                )
        ));
```

Output:

```text
IT       -> 75000
HR       -> 50000
Finance  -> 60000
```

### Important

The `toSet()` removes duplicate salaries before finding the second-highest salary.

---

## 20. Find employees whose salary is between 50,000 and 80,000

```java
List<Employee> result = employees.stream()
        .filter(e -> e.getSalary() >= 50000 &&
                     e.getSalary() <= 80000)
        .toList();
```

Output:

```text
John
Alice
David
Emma
Mike
Sophia
```

---

# 🟠 Advanced Questions

## 21. Group employees by department

```java
Map<String, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment
        ));
```

Output:

```text
IT       -> [John, Bob, Mike]
HR       -> [Alice, Emma]
Finance  -> [David, Sophia]
```

---

## 22. Group employees by department and return only their names

```java
Map<String, List<String>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(
                        Employee::getName,
                        Collectors.toList()
                )
        ));
```

Output:

```text
IT       -> [John, Bob, Mike]
HR       -> [Alice, Emma]
Finance  -> [David, Sophia]
```

### Key concept

This combines:

```java
groupingBy()
```

with:

```java
mapping()
```

---

## 23. Find the department with the highest average salary

```java
Map.Entry<String, Double> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
        ))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue())
        .orElseThrow();
```

Output:

```text
IT -> 78333.33
```

---

## 24. Find the department with the highest total salary

```java
Map.Entry<String, Double> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summingDouble(Employee::getSalary)
        ))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue())
        .orElseThrow();
```

Output:

```text
IT -> 235000
```

---

## 25. Find the highest-paid employee from each department

Return type:

```java
Map<String, Employee>
```

Solution:

```java
Map<String, Employee> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                        Collectors.maxBy(
                                Comparator.comparing(Employee::getSalary)
                        ),
                        Optional::get
                )
        ));
```

Output:

```text
IT       -> Bob
HR       -> Emma
Finance  -> Sophia
```

---

## 26. Partition employees based on salary > 60,000

```java
Map<Boolean, List<Employee>> result = employees.stream()
        .collect(Collectors.partitioningBy(
                e -> e.getSalary() > 60000
        ));
```

The map contains:

```text
true  -> employees earning > 60000
false -> employees earning <= 60000
```

### Key concept

Use:

```java
partitioningBy()
```

when you have a **true/false condition**.

Use:

```java
groupingBy()
```

when you want to group by multiple categories.

---

## 27. Find employees whose names start with "A"

```java
List<Employee> result = employees.stream()
        .filter(e -> e.getName().startsWith("A"))
        .toList();
```

Output:

```text
Alice
```

---

## 28. Find the employee whose name has the maximum number of characters

```java
Optional<Employee> result = employees.stream()
        .max(Comparator.comparingInt(
                e -> e.getName().length()
        ));
```

Output:

```text
Sophia
```

If multiple employees have the same maximum length, `max()` returns one of them.

---

## 29. Find the youngest employee

```java
Optional<Employee> result = employees.stream()
        .min(Comparator.comparingInt(Employee::getAge));
```

Output:

```text
Mike
```

---

## 30. Find the oldest employee in each department

```java
Map<String, Employee> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                        Collectors.maxBy(
                                Comparator.comparingInt(Employee::getAge)
                        ),
                        Optional::get
                )
        ));
```

Output:

```text
IT       -> Bob
HR       -> Emma
Finance  -> David
```

---

# 🔴 Tricky Questions

## 31. Find duplicate employee names

Suppose we have:

```java
List<Employee> employees = Arrays.asList(
    new Employee(1, "John", "IT", 70000, 30),
    new Employee(2, "Alice", "HR", 50000, 28),
    new Employee(3, "John", "IT", 90000, 35),
    new Employee(4, "Bob", "Finance", 60000, 40),
    new Employee(5, "Alice", "HR", 55000, 32)
);
```

Solution:

```java
Set<String> duplicates = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getName,
                Collectors.counting()
        ))
        .entrySet()
        .stream()
        .filter(entry -> entry.getValue() > 1)
        .map(Map.Entry::getKey)
        .collect(Collectors.toSet());
```

Output:

```text
[John, Alice]
```

### Alternative using `Set`

```java
Set<String> seen = new HashSet<>();

Set<String> duplicates = employees.stream()
        .map(Employee::getName)
        .filter(name -> !seen.add(name))
        .collect(Collectors.toSet());
```

---

## 32. Find duplicate employee IDs

```java
Set<Integer> duplicates = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getId,
                Collectors.counting()
        ))
        .entrySet()
        .stream()
        .filter(entry -> entry.getValue() > 1)
        .map(Map.Entry::getKey)
        .collect(Collectors.toSet());
```

---

## 33. Find employees having the same salary

```java
Map<Double, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getSalary
        ))
        .entrySet()
        .stream()
        .filter(entry -> entry.getValue().size() > 1)
        .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue
        ));
```

Output conceptually:

```text
60000 -> [Employee1, Employee2]
```

---

## 34. Find departments having more than 2 employees

```java
List<String> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
        ))
        .entrySet()
        .stream()
        .filter(entry -> entry.getValue() > 2)
        .map(Map.Entry::getKey)
        .toList();
```

Output:

```text
[IT]
```

---

## 35. Find departments where average salary is greater than 60,000

```java
Map<String, Double> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
        ))
        .entrySet()
        .stream()
        .filter(entry -> entry.getValue() > 60000)
        .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue
        ));
```

Output:

```text
IT       -> 78333.33
Finance  -> 70000
```

---

## 36. Find the top 3 highest-paid employees

```java
List<Employee> result = employees.stream()
        .sorted(
                Comparator.comparing(Employee::getSalary)
                        .reversed()
        )
        .limit(3)
        .toList();
```

Output:

```text
Bob
Sophia
Mike
```

### Important

```java
sorted()
    .limit(3)
```

means:

1. Sort employees.
2. Take the first 3.

---

## 37. Find the top 2 highest-paid employees from each department

```java
Map<String, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                        Collectors.toList(),
                        list -> list.stream()
                                .sorted(
                                        Comparator.comparing(
                                                Employee::getSalary
                                        ).reversed()
                                )
                                .limit(2)
                                .toList()
                )
        ));
```

Output:

```text
IT       -> [Bob, Mike]
HR       -> [Emma, Alice]
Finance  -> [Sophia, David]
```

This is a very good **intermediate/advanced interview question**.

---

## 38. Find the third-highest distinct salary

```java
Optional<Double> result = employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(2)
        .findFirst();
```

For:

```text
90000
80000
75000
70000
```

Output:

```text
75000
```

### Remember

For nth-highest:

```java
.skip(n - 1)
```

---

## 39. Find the employee with the nth-highest salary

For example, find the **3rd-highest employee**:

```java
int n = 3;

Optional<Employee> result = employees.stream()
        .sorted(
                Comparator.comparing(Employee::getSalary)
                        .reversed()
        )
        .skip(n - 1)
        .findFirst();
```

Output:

```text
Mike -> 75000
```

### For distinct salaries

If duplicate salaries should be treated as one rank:

```java
int n = 3;

Optional<Employee> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getSalary,
                () -> new TreeMap<Double, List<Employee>>(
                        Comparator.reverseOrder()
                ),
                Collectors.toList()
        ))
        .values()
        .stream()
        .skip(n - 1)
        .flatMap(Collection::stream)
        .findFirst();
```

---

## 40. Check whether every employee earns more than 40,000

```java
boolean result = employees.stream()
        .allMatch(e -> e.getSalary() > 40000);
```

Output:

```text
true
```

### Other useful matching operations

#### `allMatch()`

Checks whether **all** elements satisfy a condition.

```java
employees.stream()
        .allMatch(e -> e.getSalary() > 40000);
```

#### `anyMatch()`

Checks whether **at least one** element satisfies a condition.

```java
employees.stream()
        .anyMatch(e -> e.getSalary() > 100000);
```

#### `noneMatch()`

Checks whether **no** element satisfies a condition.

```java
employees.stream()
        .noneMatch(e -> e.getSalary() < 30000);
```

---

# ⭐ Bonus Interview Questions

These are commonly asked as follow-up questions.

## 41. Convert List<Employee> to Map<Id, Employee>

```java
Map<Integer, Employee> result = employees.stream()
        .collect(Collectors.toMap(
                Employee::getId,
                Function.identity()
        ));
```

Output:

```text
1 -> John
2 -> Alice
3 -> Bob
...
```

---

## 42. Convert List<Employee> to Map<Id, Name>

```java
Map<Integer, String> result = employees.stream()
        .collect(Collectors.toMap(
                Employee::getId,
                Employee::getName
        ));
```

Output:

```text
1 -> John
2 -> Alice
3 -> Bob
...
```

---

## 43. Convert List<Employee> to Map<Department, List<Employee>>

```java
Map<String, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment
        ));
```

---

## 44. Find total salary of all employees

```java
double totalSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .sum();
```

Output:

```text
470000
```

---

## 45. Find employees whose salary is greater than the average salary

```java
double average = employees.stream()
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0);

List<Employee> result = employees.stream()
        .filter(e -> e.getSalary() > average)
        .toList();
```

---

## 46. Find the department with the most employees

```java
Map.Entry<String, Long> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
        ))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue())
        .orElseThrow();
```

Output:

```text
IT -> 3
```

---

## 47. Find employees whose salary is greater than their department's average salary

```java
Map<String, Double> departmentAverage = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
        ));

List<Employee> result = employees.stream()
        .filter(e -> e.getSalary() >
                departmentAverage.get(e.getDepartment()))
        .toList();
```

This is a **very good interview question** because it requires two Stream operations.

---

## 48. Find the highest salary in the company without sorting

```java
double maxSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .max()
        .orElse(0);
```

This is better than:

```java
sorted().reversed().findFirst()
```

because we don't need to sort the entire collection.

---

## 49. Find the total number of employees in IT and HR

```java
long count = employees.stream()
        .filter(e ->
                e.getDepartment().equals("IT") ||
                e.getDepartment().equals("HR")
        )
        .count();
```

---

## 50. Get employee names sorted alphabetically

```java
List<String> result = employees.stream()
        .map(Employee::getName)
        .sorted()
        .toList();
```

Output:

```text
[Alice, Bob, David, Emma, John, Mike, Sophia]
```

---

# 🧠 Important Stream Methods to Remember

| Method             | Purpose                           |
| ------------------ | --------------------------------- |
| `filter()`         | Filter objects                    |
| `map()`            | Transform objects                 |
| `mapToInt()`       | Convert to `IntStream`            |
| `mapToDouble()`    | Convert to `DoubleStream`         |
| `sorted()`         | Sort elements                     |
| `distinct()`       | Remove duplicates                 |
| `limit()`          | Take first N elements             |
| `skip()`           | Skip first N elements             |
| `count()`          | Count elements                    |
| `min()`            | Find minimum                      |
| `max()`            | Find maximum                      |
| `sum()`            | Calculate sum                     |
| `average()`        | Calculate average                 |
| `reduce()`         | Reduce to one value               |
| `collect()`        | Collect results                   |
| `groupingBy()`     | Group objects                     |
| `partitioningBy()` | Divide into true/false groups     |
| `mapping()`        | Transform values inside collector |
| `joining()`        | Join strings                      |
| `anyMatch()`       | At least one matches              |
| `allMatch()`       | Every element matches             |
| `noneMatch()`      | No element matches                |
| `findFirst()`      | Find first element                |
| `findAny()`        | Find any element                  |

---

# 🎯 Most Important Patterns for Interviews

## Pattern 1 — Filter

```java
employees.stream()
        .filter(e -> e.getSalary() > 60000)
        .toList();
```

Think:

> "I need only employees satisfying a condition."

---

## Pattern 2 — Map

```java
employees.stream()
        .map(Employee::getName)
        .toList();
```

Think:

> "I need to transform Employee into something else."

---

## Pattern 3 — Sorting

```java
employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary))
        .toList();
```

Descending:

```java
employees.stream()
        .sorted(
                Comparator.comparing(Employee::getSalary)
                        .reversed()
        )
        .toList();
```

---

## Pattern 4 — Maximum/Minimum

```java
employees.stream()
        .max(Comparator.comparing(Employee::getSalary));
```

```java
employees.stream()
        .min(Comparator.comparing(Employee::getSalary));
```

---

## Pattern 5 — Grouping

```java
employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment
        ));
```

Think:

> "I need a Map where one property becomes the key."

---

## Pattern 6 — Group + Count

```java
employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
        ));
```

---

## Pattern 7 — Group + Average

```java
employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
        ));
```

---

## Pattern 8 — Group + Maximum

```java
employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.maxBy(
                        Comparator.comparing(Employee::getSalary)
                )
        ));
```

---

## Pattern 9 — Top N

```java
employees.stream()
        .sorted(
                Comparator.comparing(Employee::getSalary)
                        .reversed()
        )
        .limit(3)
        .toList();
```

---

## Pattern 10 — Nth Highest

```java
employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(n - 1)
        .findFirst();
```

---

# 🔥 Top 10 Questions to Master Before an Interview

If you don't have much time, focus on these:

1. **Find the highest-paid employee.**
2. **Find the second-highest distinct salary.**
3. **Find the highest-paid employee in each department.**
4. **Find the average salary of each department.**
5. **Count employees in each department.**
6. **Find the department with the highest average salary.**
7. **Find the top 3 highest-paid employees.**
8. **Find duplicate employee names/IDs.**
9. **Find the second-highest salary in each department.**
10. **Find employees earning more than their department's average salary.**

These questions cover most of the important Stream concepts:

```text
filter()
   ↓
map()
   ↓
sorted()
   ↓
distinct()
   ↓
skip() / limit()
   ↓
max() / min()
   ↓
collect()
   ↓
groupingBy()
   ↓
counting()
   ↓
averagingDouble()
   ↓
maxBy() / minBy()
   ↓
partitioningBy()
   ↓
anyMatch() / allMatch()
```
