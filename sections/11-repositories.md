---
layout: center
---
# Spring Data Repositories

## The Repository Pattern

---

# Repository Hierarchy

```
┌─────────────────────────────────────┐
│           Repository<T, ID>         │  ← Marker interface
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│      CrudRepository<T, ID>          │  ← Basic CRUD
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│   PagingAndSortingRepository<T, ID> │  ← + Pagination & Sorting
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│        JpaRepository<T, ID>         │  ← + JPA specific methods
└─────────────────────────────────────┘
```

---

# JpaRepository Methods

```java
public interface JpaRepository<T, ID> {

    // Save operations
    <S extends T> S save(S entity);
    <S extends T> List<S> saveAll(Iterable<S> entities);

    // Find operations
    Optional<T> findById(ID id);
    List<T> findAll();
    List<T> findAllById(Iterable<ID> ids);

    // Delete operations
    void deleteById(ID id);
    void delete(T entity);
    void deleteAll();

    // Utility
    long count();
    boolean existsById(ID id);
    void flush();
}
```

---

# Creating a Repository

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    // You get all CRUD operations for free!
}
```

<v-click>

```java
// Using the repository
@Service
public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    public Student createStudent(Student student) {
        return studentRepository.save(student);
    }

    public List<Student> getAllStudents() {
        return studentRepository.findAll();
    }
}
```

</v-click>

---

# Save Operations

```java
// Save single entity
Student student = new Student();
student.setFirstName("John");
student.setLastName("Doe");
student = studentRepository.save(student);  // Returns saved entity with ID

// Save multiple entities
List<Student> students = Arrays.asList(student1, student2, student3);
List<Student> savedStudents = studentRepository.saveAll(students);

// Save and flush immediately
Student saved = studentRepository.saveAndFlush(student);
```

---

# Find Operations

```java
// Find by ID
Optional<Student> student = studentRepository.findById(1L);
student.ifPresent(s -> System.out.println(s.getName()));

// Find by ID or throw
Student student = studentRepository.findById(1L)
    .orElseThrow(() -> new StudentNotFoundException(1L));

// Find all
List<Student> allStudents = studentRepository.findAll();

// Find multiple by IDs
List<Student> students = studentRepository.findAllById(Arrays.asList(1L, 2L, 3L));

// Check existence
boolean exists = studentRepository.existsById(1L);

// Count
long count = studentRepository.count();
```

---

# Delete Operations

```java
// Delete by ID
studentRepository.deleteById(1L);

// Delete entity
studentRepository.delete(student);

// Delete multiple
studentRepository.deleteAll(studentsToDelete);

// Delete all
studentRepository.deleteAll();

// Delete by ID if exists
if (studentRepository.existsById(1L)) {
    studentRepository.deleteById(1L);
}
```

---

# Update Operations

```java
// JPA doesn't have explicit update
// Just modify and save - Hibernate tracks changes

// Find, modify, save
Student student = studentRepository.findById(1L)
    .orElseThrow(() -> new StudentNotFoundException(1L));

student.setEmail("new.email@example.com");
student.setStatus(StudentStatus.ACTIVE);

studentRepository.save(student);  // Updates existing record
```

---

# Working with Optional

```java
// Pattern 1: orElseThrow
Student student = studentRepository.findById(id)
    .orElseThrow(() -> new StudentNotFoundException(id));

// Pattern 2: orElse
Student student = studentRepository.findById(id)
    .orElse(new Student());

// Pattern 3: ifPresent
studentRepository.findById(id)
    .ifPresent(student -> {
        student.setStatus(StudentStatus.ACTIVE);
        studentRepository.save(student);
    });

// Pattern 4: map and orElse
String name = studentRepository.findById(id)
    .map(Student::getFirstName)
    .orElse("Unknown");
```

---

# Service Layer Pattern

```java
@Service
@Transactional(readOnly = true)  // Default: read-only
public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    public Student findById(Long id) {
        return studentRepository.findById(id)
            .orElseThrow(() -> new StudentNotFoundException(id));
    }

    @Transactional  // Override for write operations
    public Student create(Student student) {
        return studentRepository.save(student);
    }
}
```

**Key Points:** Constructor injection, `@Transactional` for write operations

---

# Practice: Repositories

 

<v-clicks>

1. Create `DepartmentRepository` interface
2. Create `CourseRepository` interface
3. Create `StudentService` with CRUD operations
4. Implement proper exception handling
5. Test all CRUD operations
6. Use Optional correctly in find methods

</v-clicks>

---

# Key Takeaways

<v-clicks>

- `JpaRepository` provides CRUD operations automatically
- `save()` handles both insert and update
- `findById()` returns `Optional<T>`
- Always handle `Optional` properly
- Use service layer for business logic
- Constructor injection for dependencies
- `@Transactional` manages transactions

</v-clicks>
