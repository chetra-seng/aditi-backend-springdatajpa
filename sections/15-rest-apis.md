# Building REST APIs with Spring Data JPA

## Repository and Service Layer Integration

---

# Layered Architecture Review

```
┌─────────────────────────────────────┐
│           Controller                │  ← You've seen this
├─────────────────────────────────────┤
│            Service                  │  ← Business logic + transactions
├─────────────────────────────────────┤
│           Repository                │  ← Spring Data JPA
├─────────────────────────────────────┤
│            Database                 │  ← PostgreSQL
└─────────────────────────────────────┘
```

Now we connect repositories to controllers through services

---

# DTOs with MapStruct (Review)

```java
// You've seen this - MapStruct generates the mapping
@Mapper(componentModel = "spring")
public interface StudentMapper {
    StudentResponse toResponse(Student student);
    Student toEntity(CreateStudentRequest request);
}
```

```java
// Service uses the mapper
@Service
@RequiredArgsConstructor
public class StudentService {
    private final StudentRepository studentRepository;
    private final StudentMapper studentMapper;

    public StudentResponse findById(Long id) {
        return studentRepository.findById(id)
            .map(studentMapper::toResponse)
            .orElseThrow(() -> new RuntimeException("Not found"));
    }
}
```

---

# Alternative: Spring Data Projections

Instead of fetching entity + mapping to DTO, fetch only what you need:

```java
// Interface projection - Spring Data implements this
public interface StudentSummary {
    Long getId();
    String getFirstName();
    String getLastName();
    String getDepartmentName();  // Nested property!
}

// Repository returns projection directly
public interface StudentRepository extends JpaRepository<Student, Long> {
    List<StudentSummary> findByDepartmentId(Long deptId);
    Optional<StudentSummary> findSummaryById(Long id);
}
```

**Benefit:** Only SELECTs the columns you need

---

# Projection vs DTO

| Approach | When to Use |
|----------|-------------|
| **DTO + MapStruct** | Complex transformations, reusable across layers |
| **Projection** | Simple read-only views, performance-critical queries |

```java
// Projection query only selects needed columns
SELECT s.id, s.first_name, s.last_name, d.name
FROM student s JOIN department d ...

// Entity query selects everything, then maps
SELECT * FROM student s JOIN department d ...
```

---

# Service Layer - Read Operations

```java
@Service
@Transactional(readOnly = true)  // Default: read-only for safety
@RequiredArgsConstructor
public class StudentService {

    private final StudentRepository studentRepository;
    private final StudentMapper studentMapper;

    public List<StudentResponse> findAll() {
        return studentRepository.findAll().stream()
            .map(studentMapper::toResponse)
            .toList();
    }

    public StudentResponse findById(Long id) {
        return studentRepository.findById(id)
            .map(studentMapper::toResponse)
            .orElseThrow(() -> new RuntimeException("Not found"));
    }
}
```

---

# Service Layer - Write Operations

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class StudentService {

    private final StudentRepository studentRepository;
    private final DepartmentRepository departmentRepository;
    private final StudentMapper studentMapper;

    @Transactional  // Override: this method writes
    public StudentResponse create(CreateStudentRequest request) {
        Student student = studentMapper.toEntity(request);

        if (request.departmentId() != null) {
            Department dept = departmentRepository
                .findById(request.departmentId())
                .orElseThrow(() -> new RuntimeException("Dept not found"));
            student.setDepartment(dept);
        }

        return studentMapper.toResponse(studentRepository.save(student));
    }
}
```

---

# Why @Transactional on Service Layer?

```java
@Service
@Transactional(readOnly = true)  // Class level: all methods read-only
public class StudentService {

    // Uses class-level @Transactional(readOnly = true)
    public StudentResponse findById(Long id) { ... }

    @Transactional  // Override: this writes to DB
    public StudentResponse create(CreateStudentRequest req) { ... }

    @Transactional
    public void delete(Long id) { ... }
}
```

**Benefits:**
- Read-only = performance optimization (no dirty checking)
- Write methods explicitly marked
- Automatic rollback on exceptions

---

# Pagination with Spring Data

```java
// Repository - already supports pagination!
public interface StudentRepository extends JpaRepository<Student, Long> {
    Page<Student> findByDepartmentId(Long deptId, Pageable pageable);
    Page<Student> findByStatus(StudentStatus status, Pageable pageable);
}
```

```java
// Service
public Page<StudentResponse> findAll(Pageable pageable) {
    return studentRepository.findAll(pageable)
        .map(studentMapper::toResponse);  // Page has map()!
}

public Page<StudentResponse> findByDepartment(Long deptId, Pageable pageable) {
    return studentRepository.findByDepartmentId(deptId, pageable)
        .map(studentMapper::toResponse);
}
```

---

# Controller with Pagination - Individual Params

```java
@GetMapping
public PagedResponse<StudentResponse> getStudents(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy,
    @RequestParam(defaultValue = "asc") String direction
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
    return PagedResponse.from(studentService.findAll(pageable));
}
```

---

# Controller with Pagination - Request Object

```java
@Getter @Setter
public class PaginationRequest {
    private int page = 0;
    private int size = 10;
    private String sortBy = "id";

    public Pageable toPageable() {
        return PageRequest.of(page, size, Sort.by(sortBy));
    }
}

@GetMapping
public PagedResponse<StudentResponse> getStudents(PaginationRequest request) {
    return PagedResponse.from(studentService.findAll(request.toPageable()));
}
```

---

# Default Page Response (Verbose)

```json
{
  "content": [...],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "sort": { "empty": false, "sorted": true, "unsorted": false },
    "offset": 0,
    "paged": true,
    "unpaged": false
  },
  "totalElements": 45,
  "totalPages": 5,
  "last": false,
  "first": true,
  "size": 10,
  "number": 0,
  "sort": { "empty": false, "sorted": true, "unsorted": false },
  "numberOfElements": 10,
  "empty": false
}
```

Too much redundant info!

---

# Custom Paginated Response

```java
@Getter @Setter
@AllArgsConstructor
public class PagedResponse<T> {
    private List<T> content;
    private Pagination pagination;

    public static <T> PagedResponse<T> from(Page<T> page) {
        return new PagedResponse<>(
            page.getContent(),
            new Pagination(page.getNumber(), page.getSize(),
                           page.getTotalElements(), page.getTotalPages())
        );
    }
}

@Getter @Setter
@AllArgsConstructor
public class Pagination {
    private int page;
    private int size;
    private long total;
    private int totalPages;
}
```

---

# Custom Response Output

```json
{
  "content": [ ... ],
  "pagination": { "page": 0, "size": 10, "total": 45, "totalPages": 5 }
}
```

Much cleaner for frontend!

---

# Complete Service - Setup & Reads

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class StudentService {

    private final StudentRepository studentRepository;
    private final StudentMapper studentMapper;

    public Page<StudentResponse> findAll(Pageable pageable) {
        return studentRepository.findAll(pageable)
            .map(studentMapper::toResponse);
    }

    public StudentResponse findById(Long id) {
        return studentRepository.findById(id)
            .map(studentMapper::toResponse)
            .orElseThrow(() -> new RuntimeException("Not found"));
    }
}
```

---

# Complete Service - Writes

*Same `StudentService` class, continued:*

```java
@Transactional
public StudentResponse create(CreateStudentRequest req) {
    Student student = studentMapper.toEntity(req);
    return studentMapper.toResponse(studentRepository.save(student));
}

@Transactional
public StudentResponse update(Long id, UpdateStudentRequest req) {
    Student student = studentRepository.findById(id)
        .orElseThrow(() -> new RuntimeException("Not found"));
    studentMapper.updateEntity(req, student);
    return studentMapper.toResponse(studentRepository.save(student));
}

@Transactional
public void delete(Long id) {
    studentRepository.deleteById(id);
}
```

---

# Practice: Connect the Layers

**Tasks (30 minutes):**

<v-clicks>

1. Create service layer with `@Transactional`
2. Use your MapStruct mapper in the service
3. Add pagination to `findAll()` method
4. Wire service to controller
5. Test endpoints with pagination: `?page=0&size=5`

</v-clicks>

---

# Key Takeaways

<v-clicks>

- Service layer owns business logic and transactions
- `@Transactional(readOnly = true)` for reads, `@Transactional` for writes
- Use MapStruct for DTO mapping, Projections for simple read-only views
- Spring's `Page<T>` is verbose - wrap in custom `PagedResponse`
- `Page.map()` transforms entities to DTOs

</v-clicks>
