# Building REST APIs

## Complete Application Architecture

---

# Layered Architecture

```
┌─────────────────────────────────────┐
│           Controller                │  ← HTTP handling
├─────────────────────────────────────┤
│            Service                  │  ← Business logic
├─────────────────────────────────────┤
│           Repository                │  ← Data access
├─────────────────────────────────────┤
│            Database                 │  ← PostgreSQL
└─────────────────────────────────────┘
```

---

# DTO Pattern

```java
// Request DTO
public record CreateStudentRequest(
    @NotBlank String firstName,
    @NotBlank String lastName,
    @Email @NotBlank String email,
    Long departmentId
) {}

// Response DTO
public record StudentResponse(
    Long id,
    String firstName,
    String lastName,
    String email,
    String departmentName,
    LocalDateTime createdAt
) {
    public static StudentResponse from(Student student) {
        return new StudentResponse(
            student.getId(),
            student.getFirstName(),
            student.getLastName(),
            student.getEmail(),
            student.getDepartment() != null ?
                student.getDepartment().getName() : null,
            student.getCreatedAt()
        );
    }
}
```

---

# Service Layer

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class StudentService {

    private final StudentRepository studentRepository;
    private final DepartmentRepository departmentRepository;

    public List<StudentResponse> findAll() {
        return studentRepository.findAll().stream()
            .map(StudentResponse::from)
            .toList();
    }

    public StudentResponse findById(Long id) {
        return studentRepository.findById(id)
            .map(StudentResponse::from)
            .orElseThrow(() -> new StudentNotFoundException(id));
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

    @Transactional
    public StudentResponse create(CreateStudentRequest request) {
        if (studentRepository.existsByEmail(request.email())) {
            throw new EmailAlreadyExistsException(request.email());
        }

        Student student = new Student();
        student.setFirstName(request.firstName());
        student.setLastName(request.lastName());
        student.setEmail(request.email());

        if (request.departmentId() != null) {
            Department dept = departmentRepository.findById(request.departmentId())
                .orElseThrow(() -> new DepartmentNotFoundException(request.departmentId()));
            student.setDepartment(dept);
        }

        return StudentResponse.from(studentRepository.save(student));
    }
}
```

---

# Controller - GET Operations

```java
@RestController
@RequestMapping("/api/students")
@RequiredArgsConstructor
public class StudentController {

    private final StudentService studentService;

    @GetMapping
    public List<StudentResponse> getAllStudents() {
        return studentService.findAll();
    }

    @GetMapping("/{id}")
    public StudentResponse getStudent(@PathVariable Long id) {
        return studentService.findById(id);
    }

    @GetMapping("/search")
    public List<StudentResponse> searchStudents(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) StudentStatus status
    ) {
        return studentService.search(name, status);
    }
}
```

---

# Controller - Write Operations

```java
@RestController
@RequestMapping("/api/students")
@RequiredArgsConstructor
public class StudentController {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public StudentResponse createStudent(
        @Valid @RequestBody CreateStudentRequest request
    ) {
        return studentService.create(request);
    }

    @PutMapping("/{id}")
    public StudentResponse updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody UpdateStudentRequest request
    ) {
        return studentService.update(id, request);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteStudent(@PathVariable Long id) {
        studentService.delete(id);
    }
}
```

---

# Paginated Endpoint

```java
@GetMapping
public Page<StudentResponse> getStudents(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy,
    @RequestParam(defaultValue = "asc") String direction
) {
    Sort sort = direction.equalsIgnoreCase("desc")
        ? Sort.by(sortBy).descending()
        : Sort.by(sortBy).ascending();

    Pageable pageable = PageRequest.of(page, size, sort);
    return studentService.findAll(pageable);
}
```

---

# Exception Handling

```java
// Custom exception
public class StudentNotFoundException extends RuntimeException {
    public StudentNotFoundException(Long id) {
        super("Student not found with id: " + id);
    }
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(StudentNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(StudentNotFoundException ex) {
        return new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return new ErrorResponse(HttpStatus.BAD_REQUEST.value(), message);
    }
}

public record ErrorResponse(int status, String message) {}
```

---

# Validation

```java
public record CreateStudentRequest(
    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50)
    String firstName,

    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 50)
    String lastName,

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    String email,

    @Positive(message = "Department ID must be positive")
    Long departmentId
) {}
```

---

# Complete CRUD Example

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class StudentService {

    public Page<StudentResponse> findAll(Pageable pageable) {
        return studentRepository.findAll(pageable).map(StudentResponse::from);
    }

    public StudentResponse findById(Long id) { /* ... */ }

    @Transactional
    public StudentResponse create(CreateStudentRequest req) { /* ... */ }

    @Transactional
    public StudentResponse update(Long id, UpdateStudentRequest req) {
        Student student = studentRepository.findById(id)
            .orElseThrow(() -> new StudentNotFoundException(id));

        student.setFirstName(req.firstName());
        student.setLastName(req.lastName());
        // ... update other fields

        return StudentResponse.from(studentRepository.save(student));
    }

    @Transactional
    public void delete(Long id) {
        if (!studentRepository.existsById(id)) {
            throw new StudentNotFoundException(id);
        }
        studentRepository.deleteById(id);
    }
}
```

---

# Lab Exercise: REST API

**Tasks (45 minutes):**

<v-clicks>

1. Create `StudentController` with all CRUD endpoints
2. Create request/response DTOs
3. Implement `StudentService` with proper transactions
4. Add global exception handling
5. Add validation to request DTOs
6. Test all endpoints with Postman or curl

</v-clicks>

---

# Key Takeaways

<v-clicks>

- Use layered architecture: Controller → Service → Repository
- DTOs separate API from entities
- Validation with `@Valid` and Bean Validation annotations
- `@RestControllerAdvice` for global exception handling
- `@Transactional` on service methods
- Use `@ResponseStatus` for HTTP status codes
- Map entities to DTOs in service layer

</v-clicks>
