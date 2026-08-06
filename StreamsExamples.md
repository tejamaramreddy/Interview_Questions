For interviews (4–8 years experience), most Stream questions are based on a **List of Objects**, not primitive lists. Below are **15 additional interview-level examples** using an `Employee` list.

```java
class Employee {
    private int id;
    private String name;
    private String department;
    private double salary;
    private int age;

    public Employee(int id, String name, String department, double salary, int age) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
        this.age = age;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public int getAge() { return age; }

    @Override
    public String toString() {
        return name;
    }
}
```

```java
List<Employee> employees = List.of(
    new Employee(101, "John", "IT", 60000, 30),
    new Employee(102, "Alice", "HR", 45000, 28),
    new Employee(103, "Bob", "IT", 80000, 35),
    new Employee(104, "David", "Finance", 70000, 40),
    new Employee(105, "Emma", "HR", 55000, 32),
    new Employee(106, "Mike", "IT", 90000, 29)
);
```

---

# 1. Get All Employee Names

```java
List<String> names = employees.stream()
        .map(Employee::getName)
        .toList();

System.out.println(names);
```

---

# 2. Employees Older Than 30

```java
employees.stream()
        .filter(e -> e.getAge() > 30)
        .forEach(System.out::println);
```

---

# 3. Highest Salary

```java
Employee highest = employees.stream()
        .max(Comparator.comparing(Employee::getSalary))
        .orElse(null);

System.out.println(highest);
```

---

# 4. Lowest Salary

```java
Employee lowest = employees.stream()
        .min(Comparator.comparing(Employee::getSalary))
        .orElse(null);

System.out.println(lowest);
```

---

# 5. Average Salary

```java
double avg = employees.stream()
        .collect(Collectors.averagingDouble(Employee::getSalary));

System.out.println(avg);
```

---

# 6. Total Salary

```java
double total = employees.stream()
        .mapToDouble(Employee::getSalary)
        .sum();

System.out.println(total);
```

---

# 7. Employee Count per Department

```java
Map<String, Long> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()));

System.out.println(result);
```

Output

```text
{HR=2, IT=3, Finance=1}
```

---

# 8. Employees Grouped by Department

```java
Map<String, List<Employee>> result =
employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

---

# 9. Find Second Highest Salary

```java
Employee second = employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).reversed())
        .skip(1)
        .findFirst()
        .orElse(null);

System.out.println(second);
```

---

# 10. Sort by Name

```java
employees.stream()
        .sorted(Comparator.comparing(Employee::getName))
        .forEach(System.out::println);
```

---

# 11. Sort by Age then Salary

```java
employees.stream()
        .sorted(
            Comparator.comparing(Employee::getAge)
                      .thenComparing(Employee::getSalary))
        .forEach(System.out::println);
```

---

# 12. Convert List to Map

```java
Map<Integer, Employee> map =
employees.stream()
        .collect(Collectors.toMap(
                Employee::getId,
                e -> e));

System.out.println(map);
```

Output

```text
101 -> John
102 -> Alice
...
```

---

# 13. Department Names (Unique)

```java
List<String> departments =
employees.stream()
        .map(Employee::getDepartment)
        .distinct()
        .toList();

System.out.println(departments);
```

Output

```text
[IT, HR, Finance]
```

---

# 14. Highest Paid Employee in Each Department

```java
Map<String, Optional<Employee>> result =
employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.maxBy(
                        Comparator.comparing(Employee::getSalary)
                )));

System.out.println(result);
```

Output

```text
IT -> Mike
HR -> Emma
Finance -> David
```

---

# 15. Partition Employees by Salary > 60000

```java
Map<Boolean, List<Employee>> result =
employees.stream()
        .collect(Collectors.partitioningBy(
                e -> e.getSalary() > 60000));

System.out.println(result);
```

Output

```text
true -> [Bob, David, Mike]
false -> [John, Alice, Emma]
```

---

# 16. Find Employee by ID

```java
Employee emp = employees.stream()
        .filter(e -> e.getId() == 103)
        .findFirst()
        .orElse(null);

System.out.println(emp);
```

---

# 17. Check if All Employees Earn More Than 30K

```java
boolean result = employees.stream()
        .allMatch(e -> e.getSalary() > 30000);

System.out.println(result);
```

---

# 18. Check if Any Employee is in Finance

```java
boolean result = employees.stream()
        .anyMatch(e -> e.getDepartment().equals("Finance"));

System.out.println(result);
```

---

# 19. Get Top 3 Highest Salaries

```java
employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).reversed())
        .limit(3)
        .forEach(System.out::println);
```

---

# 20. Skip First Two Employees

```java
employees.stream()
        .skip(2)
        .forEach(System.out::println);
```

---

# 21. Join All Employee Names

```java
String names = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.joining(", "));

System.out.println(names);
```

---

# 22. Salary Statistics

```java
DoubleSummaryStatistics stats =
employees.stream()
        .collect(Collectors.summarizingDouble(Employee::getSalary));

System.out.println(stats);
```

Output

```text
Count
Sum
Average
Min
Max
```

---

# 23. Get Employee Names in Uppercase

```java
employees.stream()
        .map(Employee::getName)
        .map(String::toUpperCase)
        .forEach(System.out::println);
```

---

# 24. Collect IT Employees into a Set

```java
Set<Employee> itEmployees =
employees.stream()
        .filter(e -> e.getDepartment().equals("IT"))
        .collect(Collectors.toSet());

System.out.println(itEmployees);
```

---

# 25. Find the Youngest Employee

```java
Employee youngest = employees.stream()
        .min(Comparator.comparing(Employee::getAge))
        .orElse(null);

System.out.println(youngest);
```

---

# ⭐ Most Frequently Asked Stream Interview Questions

These are the ones that appear repeatedly in Java backend interviews:

| Question                                 | Concepts Tested              |
| ---------------------------------------- | ---------------------------- |
| Highest salary employee                  | `max()`, `Comparator`        |
| Second highest salary                    | `sorted()`, `skip()`         |
| Group employees by department            | `groupingBy()`               |
| Count employees in each department       | `groupingBy()`, `counting()` |
| Highest paid employee in each department | `groupingBy()`, `maxBy()`    |
| Convert List to Map                      | `toMap()`                    |
| Average salary                           | `averagingDouble()`          |
| Total salary                             | `sum()`                      |
| Top N highest salaries                   | `sorted()`, `limit()`        |
| Partition employees by condition         | `partitioningBy()`           |

Mastering these examples will prepare you for the majority of Java Stream questions asked in mid-level and senior Java backend interviews.
