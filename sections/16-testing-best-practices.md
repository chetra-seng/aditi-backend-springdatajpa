# Testing & Best Practices

## Quality and Production Readiness

---

# Testing Layers

```
┌─────────────────────────────────────┐
│     Integration Tests               │  ← Full stack
├─────────────────────────────────────┤
│       Service Tests                 │  ← Business logic
├─────────────────────────────────────┤
│      Repository Tests               │  ← Data access
└─────────────────────────────────────┘
```

---

# Testing Repository with @DataJpaTest

```java
@DataJpaTest
class StudentRepositoryTest {

    @Autowired
    private StudentRepository repository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void shouldFindByEmail() {
        // Given
        Student student = new Student();
        student.setFirstName("John");
        student.setLastName("Doe");
        student.setEmail("john@test.com");
        entityManager.persistAndFlush(student);

        // When
        Optional<Student> found = repository.findByEmail("john@test.com");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getFirstName()).isEqualTo("John");
    }
}
```

---

# @DataJpaTest Features

<v-clicks>

- Configures in-memory database (H2) by default
- Scans for `@Entity` classes
- Configures Spring Data JPA repositories
- Each test runs in a transaction (rolled back after)
- Use `@AutoConfigureTestDatabase(replace = NONE)` for real DB

</v-clicks>

---

# Testing with Real PostgreSQL

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class StudentRepositoryTest {

    @Autowired
    private StudentRepository repository;

    @Test
    void shouldCountByStatus() {
        // Given
        createStudentWithStatus(StudentStatus.ACTIVE);
        createStudentWithStatus(StudentStatus.ACTIVE);
        createStudentWithStatus(StudentStatus.INACTIVE);

        // When
        long count = repository.countByStatus(StudentStatus.ACTIVE);

        // Then
        assertThat(count).isEqualTo(2);
    }
}
```

---

# Testing with Testcontainers

```java
@SpringBootTest
@Testcontainers
class StudentServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private StudentService studentService;

    @Test
    void shouldCreateStudent() {
        // Test with real PostgreSQL in container
    }
}
```

---

# Testing Service Layer

```java
@ExtendWith(MockitoExtension.class)
class StudentServiceTest {

    @Mock
    private StudentRepository studentRepository;

    @InjectMocks
    private StudentService studentService;

    @Test
    void shouldFindById() {
        // Given
        Student student = new Student();
        student.setId(1L);
        student.setFirstName("John");

        when(studentRepository.findById(1L))
            .thenReturn(Optional.of(student));

        // When
        StudentResponse response = studentService.findById(1L);

        // Then
        assertThat(response.firstName()).isEqualTo("John");
        verify(studentRepository).findById(1L);
    }
}
```

---

# Testing Controller

```java
@WebMvcTest(StudentController.class)
class StudentControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private StudentService studentService;

    @Test
    void shouldGetStudent() throws Exception {
        StudentResponse response = new StudentResponse(
            1L, "John", "Doe", "john@test.com", "IT", null
        );

        when(studentService.findById(1L)).thenReturn(response);

        mockMvc.perform(get("/api/students/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.firstName").value("John"))
            .andExpect(jsonPath("$.email").value("john@test.com"));
    }
}
```

---

# Testing POST Endpoints

```java
@Test
void shouldCreateStudent() throws Exception {
    CreateStudentRequest request = new CreateStudentRequest(
        "John", "Doe", "john@test.com", null
    );

    StudentResponse response = new StudentResponse(
        1L, "John", "Doe", "john@test.com", null, null
    );

    when(studentService.create(any())).thenReturn(response);

    mockMvc.perform(post("/api/students")
            .contentType(MediaType.APPLICATION_JSON)
            .content("""
                {
                    "firstName": "John",
                    "lastName": "Doe",
                    "email": "john@test.com"
                }
                """))
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.id").value(1));
}
```

---

# Key Takeaways

<v-clicks>

**Architecture:**
- Use layered architecture (Controller → Service → Repository)
- Keep controllers thin, services contain business logic
- Use DTOs for API contracts

**JPA:**
- Use `LAZY` fetching for collections
- Always handle N+1 problems
- Use projections for read-only queries

**Transactions:**
- `@Transactional(readOnly = true)` at class level
- Override with `@Transactional` for write methods

</v-clicks>

---

# More Best Practices

<v-clicks>

**Naming:**
- Tables: lowercase, plural, snake_case (`students`)
- Columns: snake_case (`first_name`)
- Entities: PascalCase, singular (`Student`)

**Performance:**
- Index foreign keys
- Use pagination for lists
- Monitor SQL queries

**Security:**
- Never expose entities directly
- Validate all input
- Use prepared statements (JPA does this)

</v-clicks>

---

# Common Mistakes to Avoid

<v-clicks>

- Using `EAGER` fetch on collections
- Not handling `Optional` properly
- Missing `@Transactional` on write operations
- Exposing entities in API responses
- Ignoring N+1 queries
- Not validating input
- Not writing tests

</v-clicks>

---

# Quick Reference

```java
// Essential Annotations
@Entity @Table @Id @GeneratedValue @Column
@OneToOne @OneToMany @ManyToOne @ManyToMany
@JoinColumn @JoinTable
@Query @Modifying @Transactional

// Repository Methods
save() findById() findAll() delete() count()

// Query Keywords
findBy... And Or Like Containing Between
OrderBy LessThan GreaterThan IsNull

// Fetch Strategies
FetchType.LAZY   // Default for collections
FetchType.EAGER  // Avoid for collections
JOIN FETCH       // JPQL eager loading
@EntityGraph     // Declarative eager loading
```

---

# Course Complete!

**What you've learned:**

<v-clicks>

- PostgreSQL fundamentals (CRUD, Joins, Subqueries)
- Spring Data JPA setup and configuration
- Entity mapping and relationships
- Repository pattern and query methods
- Pagination, sorting, transactions
- Building REST APIs
- Testing strategies

</v-clicks>

---

# Next Steps

<v-clicks>

- Build a complete project from scratch
- Explore Spring Security for authentication
- Learn about caching with Spring Cache
- Study database migrations with Flyway
- Practice with more complex queries
- Deploy to cloud platforms

</v-clicks>

---
layout: center
class: text-center
---

# Thank You!

**Resources:**
- PostgreSQL: postgresql.org/docs
- Spring Data JPA: spring.io/projects/spring-data-jpa
- Hibernate: hibernate.org/orm/documentation

Happy Coding!
