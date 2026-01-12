# Pagination & Sorting

## Handling Large Datasets

---

# Why Pagination?

<v-clicks>

- **Performance** - Don't load millions of rows
- **Memory** - Prevent OutOfMemoryError
- **User Experience** - Show data in manageable chunks
- **Network** - Reduce data transfer

</v-clicks>

---

# Basic Sorting

```java
// Repository method
List<Student> findByStatus(StudentStatus status, Sort sort);

// Usage
Sort sort = Sort.by("lastName").ascending();
Sort sort = Sort.by("lastName").descending();
Sort sort = Sort.by("lastName").ascending()
                .and(Sort.by("firstName").ascending());

// Direction enum
Sort.by(Sort.Direction.ASC, "lastName");
Sort.by(Sort.Direction.DESC, "enrollmentDate");

// Call repository
List<Student> students = studentRepository.findByStatus(
    StudentStatus.ACTIVE,
    Sort.by("lastName").ascending()
);
```

---

# Pagination with Pageable

```java
// Repository method
Page<Student> findByStatus(StudentStatus status, Pageable pageable);

// Create Pageable
Pageable pageable = PageRequest.of(0, 10);  // page 0, size 10

// With sorting
Pageable pageable = PageRequest.of(0, 10, Sort.by("lastName").ascending());

// Call repository
Page<Student> page = studentRepository.findByStatus(
    StudentStatus.ACTIVE,
    pageable
);
```

---

# Page Object

```java
Page<Student> page = repository.findAll(PageRequest.of(0, 10));

// Get data
List<Student> students = page.getContent();

// Pagination info
int currentPage = page.getNumber();          // Current page (0-based)
int pageSize = page.getSize();               // Page size
long totalElements = page.getTotalElements(); // Total records
int totalPages = page.getTotalPages();        // Total pages

// Navigation
boolean hasNext = page.hasNext();
boolean hasPrevious = page.hasPrevious();
boolean isFirst = page.isFirst();
boolean isLast = page.isLast();
```

---

# Pagination in Service Layer

```java
@Service
@Transactional(readOnly = true)
public class StudentService {

    private final StudentRepository studentRepository;

    public Page<Student> getStudents(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return studentRepository.findAll(pageable);
    }

    public Page<Student> getActiveStudents(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return studentRepository.findByStatus(StudentStatus.ACTIVE, pageable);
    }
}
```

---

# Pagination in Controller

```java
@RestController
@RequestMapping("/api/students")
public class StudentController {

    private final StudentService studentService;

    @GetMapping
    public Page<Student> getStudents(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy
    ) {
        return studentService.getStudents(page, size, sortBy);
    }
}
```

---

# Page Response JSON

```json
{
  "content": [
    { "id": 1, "firstName": "John", "lastName": "Doe" },
    { "id": 2, "firstName": "Jane", "lastName": "Smith" }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "sort": { "sorted": true, "direction": "ASC" }
  },
  "totalElements": 100,
  "totalPages": 10,
  "first": true,
  "last": false,
  "numberOfElements": 10
}
```

---

# Slice vs Page

```java
// Page - knows total count (extra COUNT query)
Page<Student> findByStatus(StudentStatus status, Pageable pageable);

// Slice - doesn't know total (more efficient)
Slice<Student> findByStatus(StudentStatus status, Pageable pageable);
```

<v-click>

**When to use:**
- `Page` - When you need total count (pagination UI with page numbers)
- `Slice` - When you only need "Load More" (infinite scroll)

</v-click>

---

# Pagination with Custom Query

```java
public interface StudentRepository extends JpaRepository<Student, Long> {

    @Query("SELECT s FROM Student s WHERE s.department.id = :deptId")
    Page<Student> findByDepartmentId(
        @Param("deptId") Long departmentId,
        Pageable pageable
    );

    @Query("SELECT s FROM Student s JOIN FETCH s.department " +
           "WHERE s.status = :status",
           countQuery = "SELECT COUNT(s) FROM Student s WHERE s.status = :status")
    Page<Student> findByStatusWithDepartment(
        @Param("status") StudentStatus status,
        Pageable pageable
    );
}
```

---

# Sorting with Derived Queries

```java
public interface StudentRepository extends JpaRepository<Student, Long> {

    // Sort in method name
    List<Student> findByStatusOrderByLastNameAsc(StudentStatus status);

    List<Student> findByStatusOrderByEnrollmentDateDesc(StudentStatus status);

    // Multiple sort fields
    List<Student> findByStatusOrderByLastNameAscFirstNameAsc(StudentStatus status);

    // With limit
    List<Student> findTop10ByStatusOrderByEnrollmentDateDesc(StudentStatus status);
}
```

---

# Custom DTO with Pagination

```java
// DTO
public record StudentDTO(Long id, String fullName, String email) {
    public static StudentDTO from(Student student) {
        return new StudentDTO(
            student.getId(),
            student.getFirstName() + " " + student.getLastName(),
            student.getEmail()
        );
    }
}

// Service
public Page<StudentDTO> getStudentDTOs(int page, int size) {
    Page<Student> students = studentRepository.findAll(
        PageRequest.of(page, size)
    );
    return students.map(StudentDTO::from);
}
```

---

# Lab Exercise: Pagination & Sorting

**Tasks (30 minutes):**

<v-clicks>

1. Add paginated find methods to repository
2. Create service method for paginated students
3. Create REST endpoint with pagination parameters
4. Add sorting to the endpoint
5. Test with different page sizes and sort fields
6. Compare Page vs Slice performance

</v-clicks>

---

# Module 13 Summary

<v-clicks>

- Use `Pageable` for pagination
- `PageRequest.of(page, size, sort)` creates Pageable
- `Page<T>` contains data and metadata
- `Sort.by()` creates sort specification
- `Slice<T>` is more efficient when total count not needed
- Use `countQuery` with JOIN FETCH
- Map Page to DTOs with `.map()`

</v-clicks>
