# Transactions & Performance

## N+1 Problem and Solutions

---

# @Transactional Basics

```java
@Service
@Transactional(readOnly = true)  // Default for class
public class StudentService {

    public List<Student> getAllStudents() {
        return repository.findAll();  // Uses readOnly transaction
    }

    @Transactional  // Override: read-write transaction
    public Student createStudent(Student student) {
        return repository.save(student);
    }

    @Transactional(rollbackFor = Exception.class)
    public void enrollStudent(Long studentId, Long courseId) {
        // Multiple operations in single transaction
    }
}
```

---

# Transaction Attributes

```java
@Transactional(
    readOnly = false,                    // Read-write transaction
    timeout = 30,                        // 30 seconds timeout
    rollbackFor = Exception.class,       // Rollback on exception
    noRollbackFor = NotFoundException.class
)
```

<v-click>

**Best Practices:**
- Use `@Transactional(readOnly = true)` at class level
- Override with `@Transactional` for write methods
- Set `rollbackFor = Exception.class` for checked exceptions

</v-click>

---

# The N+1 Problem

```java
// BAD: N+1 queries!
List<Student> students = studentRepository.findAll();
for (Student student : students) {
    System.out.println(student.getDepartment().getName());
}

// Query 1: SELECT * FROM students
// Query 2: SELECT * FROM departments WHERE id = 1
// Query 3: SELECT * FROM departments WHERE id = 2
// ... N more queries!
```

<v-click>

With 100 students, you get **101 queries** instead of **1 or 2**!

</v-click>

---

# Solution 1: Join Fetch

```java
public interface StudentRepository extends JpaRepository<Student, Long> {

    @Query("SELECT s FROM Student s JOIN FETCH s.department")
    List<Student> findAllWithDepartment();

    @Query("SELECT s FROM Student s " +
           "JOIN FETCH s.department d " +
           "WHERE d.name = :deptName")
    List<Student> findByDepartmentWithFetch(@Param("deptName") String deptName);
}
```

**Result:** Single query with JOIN

---

# Solution 2: Entity Graphs

```java
@Entity
@NamedEntityGraph(
    name = "Student.withDepartment",
    attributeNodes = @NamedAttributeNode("department")
)
public class Student {
    // ...
}

// Repository usage
public interface StudentRepository extends JpaRepository<Student, Long> {

    @EntityGraph(value = "Student.withDepartment")
    List<Student> findByStatus(StudentStatus status);

    @EntityGraph(attributePaths = {"department", "courses"})
    Optional<Student> findById(Long id);
}
```

---

# Solution 3: Batch Fetching

```java
// application.properties
spring.jpa.properties.hibernate.default_batch_fetch_size=25
```

<v-click>

Or on specific relationship:

```java
@Entity
public class Student {

    @ManyToOne(fetch = FetchType.LAZY)
    @BatchSize(size = 25)
    private Department department;
}
```

Instead of N queries, Hibernate fetches in batches of 25.

</v-click>

---

# Detecting N+1 Problems

```properties
# application.properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Log statistics
logging.level.org.hibernate.stat=DEBUG
spring.jpa.properties.hibernate.generate_statistics=true
```

<v-click>

Watch for repeated queries in logs!

</v-click>

---

# Lazy Loading Outside Transaction

```java
// This will throw LazyInitializationException!
@GetMapping("/{id}")
public Student getStudent(@PathVariable Long id) {
    Student student = studentRepository.findById(id).orElseThrow();
    student.getDepartment().getName();  // LAZY - transaction already closed!
    return student;
}
```

<v-click>

**Solutions:**
1. Use `JOIN FETCH` or Entity Graph
2. Use DTOs (projections)
3. Open Session in View (not recommended)

</v-click>

---

# Using Projections

```java
// Interface projection
public interface StudentSummary {
    Long getId();
    String getFirstName();
    String getLastName();
    String getDepartmentName();  // Nested property
}

// Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    List<StudentSummary> findByStatus(StudentStatus status);
}

// Only selected fields are fetched!
```

---

# DTO Projection

```java
public record StudentDTO(
    Long id,
    String firstName,
    String lastName,
    String departmentName
) {}

@Query("SELECT new com.example.dto.StudentDTO(" +
       "s.id, s.firstName, s.lastName, s.department.name) " +
       "FROM Student s WHERE s.status = :status")
List<StudentDTO> findStudentDTOs(@Param("status") StudentStatus status);
```

---

# Performance Best Practices

<v-clicks>

**Do:**
- Use `LAZY` fetch for collections
- Use projections when you need subset of fields
- Use pagination for large datasets
- Use `JOIN FETCH` when you need related data
- Monitor query count in logs

**Don't:**
- Don't use `EAGER` fetch on collections
- Don't fetch entire entities when you need few fields
- Don't ignore N+1 warnings
- Don't load all data without pagination

</v-clicks>

---

# Lab Exercise: Performance

**Tasks (30 minutes):**

<v-clicks>

1. Enable SQL logging and statistics
2. Create a query that causes N+1
3. Fix it with JOIN FETCH
4. Create an interface projection
5. Create a DTO projection
6. Compare query counts before and after fixes

</v-clicks>

---

# Key Takeaways

<v-clicks>

- `@Transactional` manages database transactions
- Use `readOnly = true` for read operations
- N+1 problem: extra query per related entity
- Solutions: JOIN FETCH, Entity Graphs, Batch Fetching
- Projections fetch only needed columns
- Always monitor SQL queries during development
- Use DTOs to avoid lazy loading issues

</v-clicks>
