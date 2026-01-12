# Spring Data JPA with PostgreSQL
## Complete 8-Hour Course

---

# Course Overview

**Duration:** 8 Hours (4 Modules × 2 Hours Each)

**Prerequisites:** 
- Basic Java knowledge
- PostgreSQL fundamentals (8 hours completed)
- Understanding of SQL queries

**What You'll Build:**
A complete Student Management System with REST API

---

# Course Schedule

| Module | Topic | Duration |
|--------|-------|----------|
| 1 | Introduction to Spring & JPA Fundamentals | 2 hours |
| 2 | Entity Mapping & Relationships | 2 hours |
| 3 | Spring Data JPA Repositories & Queries | 2 hours |
| 4 | Advanced Topics & Best Practices | 2 hours |

---

---

# MODULE 1: Introduction to Spring & JPA Fundamentals

**Duration: 2 Hours**

---

# 1.1 What is Spring Framework?

Spring is a comprehensive framework for enterprise Java development.

**Core Concepts:**
- Inversion of Control (IoC)
- Dependency Injection (DI)
- Aspect-Oriented Programming (AOP)

**Why Spring?**
- Simplifies Java development
- Promotes loose coupling
- Extensive ecosystem

---

# 1.2 Spring Boot Overview

Spring Boot makes it easy to create stand-alone, production-grade Spring applications.

**Key Features:**
- Auto-configuration
- Embedded servers (Tomcat, Jetty)
- Starter dependencies
- Production-ready features

```xml
<!-- No more 100 lines of XML configuration! -->
```

---

# 1.3 What is JPA?

**Java Persistence API (JPA)** is a specification for Object-Relational Mapping (ORM).

```
┌─────────────┐         ┌─────────────┐
│   Java      │   JPA   │  Database   │
│   Objects   │ ◄─────► │   Tables    │
└─────────────┘         └─────────────┘
```

**JPA is NOT an implementation** — it's a specification!

---

# 1.4 JPA vs Hibernate vs Spring Data JPA

| Layer | Description |
|-------|-------------|
| **JPA** | Specification (interfaces & annotations) |
| **Hibernate** | JPA Implementation (the actual ORM engine) |
| **Spring Data JPA** | Abstraction over JPA (reduces boilerplate) |

```
┌──────────────────────────┐
│    Spring Data JPA       │  ← You write this
├──────────────────────────┤
│       Hibernate          │  ← Does the heavy lifting
├──────────────────────────┤
│         JPA              │  ← The contract
├──────────────────────────┤
│      PostgreSQL          │  ← Your database
└──────────────────────────┘
```

---

# 1.5 Project Setup

**Step 1: Go to https://start.spring.io**

**Dependencies to add:**
- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Spring Boot DevTools (optional)
- Lombok (optional, but recommended)

---

# 1.6 Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/example/demo/
│   │       ├── DemoApplication.java
│   │       ├── entity/
│   │       ├── repository/
│   │       ├── service/
│   │       └── controller/
│   └── resources/
│       ├── application.properties
│       └── application.yml
└── test/
```

---

# 1.7 Configuring PostgreSQL Connection

**application.properties:**

```properties
# Database Configuration
spring.datasource.url=jdbc:postgresql://localhost:5432/studentdb
spring.datasource.username=postgres
spring.datasource.password=your_password
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA/Hibernate Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

---

# 1.8 Understanding ddl-auto Options

| Value | Description | Use Case |
|-------|-------------|----------|
| `none` | No schema management | Production |
| `validate` | Validates schema, no changes | Production |
| `update` | Updates schema (additive only) | Development |
| `create` | Creates schema, destroys previous | Testing |
| `create-drop` | Create on start, drop on stop | Unit tests |

⚠️ **Never use `create` or `create-drop` in production!**

---

# 1.9 Your First Entity

```java
package com.example.demo.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "students")
public class Student {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "first_name", nullable = false)
    private String firstName;
    
    @Column(name = "last_name", nullable = false)
    private String lastName;
    
    @Column(unique = true)
    private String email;
    
    // Constructors, getters, setters
}
```

---

# 1.10 Key JPA Annotations

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Marks class as JPA entity |
| `@Table` | Specifies table name |
| `@Id` | Marks primary key field |
| `@GeneratedValue` | Auto-generate ID values |
| `@Column` | Customize column mapping |
| `@Transient` | Field not persisted |

---

# 1.11 ID Generation Strategies

```java
// AUTO - Let Hibernate decide
@GeneratedValue(strategy = GenerationType.AUTO)

// IDENTITY - Database auto-increment (recommended for PostgreSQL)
@GeneratedValue(strategy = GenerationType.IDENTITY)

// SEQUENCE - Database sequence
@GeneratedValue(strategy = GenerationType.SEQUENCE, 
                generator = "student_seq")
@SequenceGenerator(name = "student_seq", 
                   sequenceName = "student_sequence",
                   allocationSize = 1)

// TABLE - Separate table for ID generation
@GeneratedValue(strategy = GenerationType.TABLE)
```

---

# 1.12 Using Lombok (Optional but Recommended)

```java
import lombok.*;
import jakarta.persistence.*;

@Entity
@Table(name = "students")
@Data                    // Generates getters, setters, toString, equals, hashCode
@NoArgsConstructor       // No-args constructor (required by JPA)
@AllArgsConstructor      // All-args constructor
@Builder                 // Builder pattern
public class Student {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String firstName;
    private String lastName;
    private String email;
}
```

---

# 1.13 Creating Your First Repository

```java
package com.example.demo.repository;

import com.example.demo.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    // That's it! You get CRUD operations for free!
}
```

**What you get automatically:**
- `save()`, `saveAll()`
- `findById()`, `findAll()`
- `deleteById()`, `delete()`, `deleteAll()`
- `count()`, `existsById()`

---

# 1.14 Lab Exercise 1: Setup & First Entity

**Tasks (30 minutes):**

1. Create a new Spring Boot project with required dependencies
2. Configure PostgreSQL connection
3. Create a `Student` entity with fields:
   - id (Long, auto-generated)
   - firstName (String)
   - lastName (String)
   - email (String, unique)
   - enrollmentDate (LocalDate)
4. Create `StudentRepository` interface
5. Run the application and verify table creation in PostgreSQL

---

# 1.15 Testing Your Setup

```java
package com.example.demo;

import com.example.demo.entity.Student;
import com.example.demo.repository.StudentRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class DataLoader implements CommandLineRunner {
    
    private final StudentRepository repository;
    
    public DataLoader(StudentRepository repository) {
        this.repository = repository;
    }
    
    @Override
    public void run(String... args) {
        Student student = new Student();
        student.setFirstName("John");
        student.setLastName("Doe");
        student.setEmail("john.doe@email.com");
        
        repository.save(student);
        System.out.println("Student saved: " + student);
    }
}
```

---

# Module 1 Summary

**What We Covered:**
- Spring Framework & Spring Boot basics
- JPA specification vs Hibernate implementation
- Spring Data JPA abstraction layer
- Project setup with PostgreSQL
- Entity creation with annotations
- Basic repository pattern

**Key Takeaways:**
- Spring Data JPA dramatically reduces boilerplate
- Entities map Java classes to database tables
- Repositories provide automatic CRUD operations

---

---

# MODULE 2: Entity Mapping & Relationships

**Duration: 2 Hours**

---

# 2.1 Entity Lifecycle States

```
┌───────────┐    persist()    ┌───────────┐
│           │ ─────────────►  │           │
│  TRANSIENT│                 │  MANAGED  │
│  (New)    │ ◄─────────────  │           │
└───────────┘    remove()     └─────┬─────┘
                                    │
                              detach() / clear()
                                    │
                                    ▼
                              ┌───────────┐
                              │ DETACHED  │
                              └─────┬─────┘
                                    │
                                merge()
                                    │
                                    ▼
                              ┌───────────┐
                              │  MANAGED  │
                              └───────────┘
```

---

# 2.2 Column Mapping Options

```java
@Column(
    name = "email_address",      // Column name in DB
    nullable = false,            // NOT NULL constraint
    unique = true,               // UNIQUE constraint
    length = 100,                // VARCHAR(100)
    columnDefinition = "TEXT",   // Override type
    insertable = true,           // Include in INSERT
    updatable = true             // Include in UPDATE
)
private String email;
```

---

# 2.3 Data Types Mapping

| Java Type | PostgreSQL Type |
|-----------|-----------------|
| `String` | VARCHAR / TEXT |
| `Integer`, `int` | INTEGER |
| `Long`, `long` | BIGINT |
| `Double`, `double` | DOUBLE PRECISION |
| `BigDecimal` | NUMERIC |
| `Boolean`, `boolean` | BOOLEAN |
| `LocalDate` | DATE |
| `LocalDateTime` | TIMESTAMP |
| `byte[]` | BYTEA |

---

# 2.4 Temporal Types

```java
import java.time.*;

@Entity
public class Event {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private LocalDate eventDate;           // DATE
    
    private LocalTime eventTime;           // TIME
    
    private LocalDateTime createdAt;       // TIMESTAMP
    
    private Instant timestamp;             // TIMESTAMP WITH TIME ZONE
    
    private ZonedDateTime zonedDateTime;   // TIMESTAMP WITH TIME ZONE
}
```

---

# 2.5 Enum Mapping

```java
public enum StudentStatus {
    ACTIVE, INACTIVE, GRADUATED, SUSPENDED
}

@Entity
public class Student {
    
    // Stored as INTEGER (0, 1, 2, 3) - NOT recommended
    @Enumerated(EnumType.ORDINAL)
    private StudentStatus status;
    
    // Stored as STRING ('ACTIVE', 'INACTIVE', ...) - Recommended!
    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private StudentStatus status;
}
```

⚠️ **Always use `EnumType.STRING`** — ordinal breaks if enum order changes!

---

# 2.6 Embedded Objects

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    private String country;
}

@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @Embedded
    private Address address;  // All address fields in same table
}
```

**Result:** Single `students` table with `street`, `city`, `state`, `zip_code`, `country` columns.

---

# 2.7 Relationship Types Overview

```
┌─────────────────────────────────────────────────────────┐
│                    JPA Relationships                     │
├─────────────────────────────────────────────────────────┤
│  @OneToOne    │  Student ←──────► StudentProfile        │
│  @OneToMany   │  Department ←────► Many Employees       │
│  @ManyToOne   │  Many Students ────► One Course         │
│  @ManyToMany  │  Students ◄──────► Courses              │
└─────────────────────────────────────────────────────────┘
```

---

# 2.8 @OneToOne Relationship

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private StudentProfile profile;
}

@Entity
public class StudentProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String bio;
    private String linkedinUrl;
    
    @OneToOne(mappedBy = "profile")
    private Student student;
}
```

---

# 2.9 @OneToMany / @ManyToOne Relationship

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Student> students = new ArrayList<>();
}

@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
}
```

---

# 2.10 Understanding Owning Side

**The owning side:**
- Contains the foreign key in the database
- Does NOT have `mappedBy` attribute
- Changes to this side are persisted

**The inverse side:**
- Has `mappedBy` attribute
- Changes are NOT automatically persisted
- Must update owning side!

```java
// ✅ Correct - updating owning side
student.setDepartment(department);

// ❌ Wrong - inverse side only, won't persist
department.getStudents().add(student);

// ✅ Best practice - update both sides
student.setDepartment(department);
department.getStudents().add(student);
```

---

# 2.11 @ManyToMany Relationship

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @ManyToMany
    @JoinTable(
        name = "student_courses",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

---

# 2.12 ManyToMany with Extra Columns

When the join table needs additional columns, create an entity for it:

```java
@Entity
@Table(name = "enrollments")
public class Enrollment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "student_id")
    private Student student;
    
    @ManyToOne
    @JoinColumn(name = "course_id")
    private Course course;
    
    private LocalDate enrollmentDate;
    private String grade;
    private Boolean completed;
}
```

---

# 2.13 Cascade Types

```java
@OneToMany(cascade = CascadeType.ALL)
```

| Cascade Type | Description |
|--------------|-------------|
| `PERSIST` | Save child when parent is saved |
| `MERGE` | Update child when parent is updated |
| `REMOVE` | Delete child when parent is deleted |
| `REFRESH` | Refresh child when parent is refreshed |
| `DETACH` | Detach child when parent is detached |
| `ALL` | All of the above |

⚠️ **Be careful with `REMOVE`** — can cause unintended deletions!

---

# 2.14 Fetch Types

```java
// EAGER - Load immediately with parent
@ManyToOne(fetch = FetchType.EAGER)  // Default for @ManyToOne, @OneToOne

// LAZY - Load on demand (when accessed)
@OneToMany(fetch = FetchType.LAZY)   // Default for @OneToMany, @ManyToMany
```

**Best Practice:**
- Use `LAZY` for collections (`@OneToMany`, `@ManyToMany`)
- Use `LAZY` for `@ManyToOne` when not always needed
- Never use `EAGER` on multiple collections

---

# 2.15 Orphan Removal

```java
@Entity
public class Department {
    
    @OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,
        orphanRemoval = true  // Delete students without department
    )
    private List<Student> students = new ArrayList<>();
}
```

```java
// This will DELETE the student from database
department.getStudents().remove(student);
```

---

# 2.16 Complete Entity Example

```java
@Entity
@Table(name = "students")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Student {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 50)
    private String firstName;
    
    @Column(nullable = false, length = 50)
    private String lastName;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private StudentStatus status;
    
    private LocalDate enrollmentDate;
    
    @Embedded
    private Address address;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    @ManyToMany
    @JoinTable(
        name = "student_courses",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

---

# 2.17 Lab Exercise 2: Entity Relationships

**Tasks (45 minutes):**

1. Create the following entities with relationships:
   - `Department` (id, name, description)
   - `Course` (id, title, credits, description)
   - `Student` (extend from Module 1)
   - `Enrollment` (student, course, enrollmentDate, grade)

2. Implement relationships:
   - Department → Students (OneToMany)
   - Student → Courses (ManyToMany via Enrollment)

3. Add proper cascade and fetch types

4. Test by creating sample data

---

# 2.18 Helper Methods for Bidirectional Relationships

```java
@Entity
public class Department {
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Student> students = new ArrayList<>();
    
    // Helper method to maintain both sides
    public void addStudent(Student student) {
        students.add(student);
        student.setDepartment(this);
    }
    
    public void removeStudent(Student student) {
        students.remove(student);
        student.setDepartment(null);
    }
}
```

---

# Module 2 Summary

**What We Covered:**
- Entity lifecycle states
- Column mapping and data types
- Enum and embedded objects mapping
- All relationship types (@OneToOne, @OneToMany, @ManyToOne, @ManyToMany)
- Owning vs inverse side
- Cascade types and fetch strategies
- Orphan removal

**Key Takeaways:**
- Always use `EnumType.STRING` for enums
- Prefer `LAZY` fetching for collections
- Always update the owning side of relationships
- Use helper methods for bidirectional relationships

---

---

# MODULE 3: Spring Data JPA Repositories & Queries

**Duration: 2 Hours**

---

# 3.1 Repository Hierarchy

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
│   ListCrudRepository<T, ID>         │  ← Returns List instead of Iterable
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

# 3.2 JpaRepository Methods

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
    
    // Sorting & Paging
    List<T> findAll(Sort sort);
    Page<T> findAll(Pageable pageable);
}
```

---

# 3.3 Derived Query Methods

Spring Data JPA creates queries from method names automatically!

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // SELECT * FROM students WHERE email = ?
    Optional<Student> findByEmail(String email);
    
    // SELECT * FROM students WHERE first_name = ? AND last_name = ?
    List<Student> findByFirstNameAndLastName(String firstName, String lastName);
    
    // SELECT * FROM students WHERE last_name = ? OR email = ?
    List<Student> findByLastNameOrEmail(String lastName, String email);
}
```

---

# 3.4 Query Method Keywords

| Keyword | Sample | JPQL Snippet |
|---------|--------|--------------|
| `And` | `findByFirstNameAndLastName` | `WHERE x.firstName = ?1 AND x.lastName = ?2` |
| `Or` | `findByFirstNameOrLastName` | `WHERE x.firstName = ?1 OR x.lastName = ?2` |
| `Is`, `Equals` | `findByFirstName` | `WHERE x.firstName = ?1` |
| `Between` | `findByAgeBetween` | `WHERE x.age BETWEEN ?1 AND ?2` |
| `LessThan` | `findByAgeLessThan` | `WHERE x.age < ?1` |
| `GreaterThan` | `findByAgeGreaterThan` | `WHERE x.age > ?1` |
| `Like` | `findByFirstNameLike` | `WHERE x.firstName LIKE ?1` |
| `Containing` | `findByFirstNameContaining` | `WHERE x.firstName LIKE %?1%` |

---

# 3.5 More Query Keywords

| Keyword | Sample | JPQL Snippet |
|---------|--------|--------------|
| `StartingWith` | `findByFirstNameStartingWith` | `WHERE x.firstName LIKE ?1%` |
| `EndingWith` | `findByFirstNameEndingWith` | `WHERE x.firstName LIKE %?1` |
| `IsNull` | `findByEmailIsNull` | `WHERE x.email IS NULL` |
| `IsNotNull` | `findByEmailIsNotNull` | `WHERE x.email IS NOT NULL` |
| `In` | `findByStatusIn` | `WHERE x.status IN ?1` |
| `NotIn` | `findByStatusNotIn` | `WHERE x.status NOT IN ?1` |
| `True` | `findByActiveTrue` | `WHERE x.active = TRUE` |
| `False` | `findByActiveFalse` | `WHERE x.active = FALSE` |
| `OrderBy` | `findByLastNameOrderByFirstNameAsc` | `ORDER BY x.firstName ASC` |

---

# 3.6 Derived Query Examples

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // Count queries
    long countByStatus(StudentStatus status);
    
    // Exists queries
    boolean existsByEmail(String email);
    
    // Delete queries
    void deleteByStatus(StudentStatus status);
    
    // Distinct
    List<Student> findDistinctByLastName(String lastName);
    
    // Top/First
    List<Student> findTop3ByOrderByEnrollmentDateDesc();
    Student findFirstByOrderByEnrollmentDateAsc();
    
    // Nested properties
    List<Student> findByDepartmentName(String departmentName);
    List<Student> findByAddressCity(String city);
}
```

---

# 3.7 @Query Annotation - JPQL

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // JPQL with positional parameters
    @Query("SELECT s FROM Student s WHERE s.status = ?1")
    List<Student> findByStatus(StudentStatus status);
    
    // JPQL with named parameters
    @Query("SELECT s FROM Student s WHERE s.firstName = :firstName " +
           "AND s.lastName = :lastName")
    Optional<Student> findByFullName(
        @Param("firstName") String firstName,
        @Param("lastName") String lastName
    );
    
    // JPQL with LIKE
    @Query("SELECT s FROM Student s WHERE s.email LIKE %:domain")
    List<Student> findByEmailDomain(@Param("domain") String domain);
}
```

---

# 3.8 @Query Annotation - Native SQL

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // Native PostgreSQL query
    @Query(value = "SELECT * FROM students WHERE email LIKE %?1%", 
           nativeQuery = true)
    List<Student> findByEmailContainingNative(String email);
    
    // Native query with named parameter
    @Query(value = "SELECT * FROM students s " +
                   "JOIN departments d ON s.department_id = d.id " +
                   "WHERE d.name = :deptName",
           nativeQuery = true)
    List<Student> findByDepartmentNameNative(@Param("deptName") String deptName);
    
    // Using PostgreSQL specific features
    @Query(value = "SELECT * FROM students " +
                   "WHERE created_at > NOW() - INTERVAL '30 days'",
           nativeQuery = true)
    List<Student> findRecentStudents();
}
```

---

# 3.9 Modifying Queries

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // Update query
    @Modifying
    @Transactional
    @Query("UPDATE Student s SET s.status = :status WHERE s.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") StudentStatus status);
    
    // Bulk update
    @Modifying
    @Transactional
    @Query("UPDATE Student s SET s.status = :status " +
           "WHERE s.enrollmentDate < :date")
    int updateStatusForOldStudents(
        @Param("status") StudentStatus status,
        @Param("date") LocalDate date
    );
    
    // Delete query
    @Modifying
    @Transactional
    @Query("DELETE FROM Student s WHERE s.status = :status")
    int deleteByStatusQuery(@Param("status") StudentStatus status);
}
```

---

# 3.10 Pagination

```java
// Repository method
Page<Student> findByStatus(StudentStatus status, Pageable pageable);

// Service usage
@Service
public class StudentService {
    
    public Page<Student> getActiveStudents(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return studentRepository.findByStatus(StudentStatus.ACTIVE, pageable);
    }
}

// Controller usage
@GetMapping("/students")
public Page<Student> getStudents(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size
) {
    return studentService.getActiveStudents(page, size);
}
```

---

# 3.11 Page Object Structure

```java
Page<Student> page = repository.findAll(PageRequest.of(0, 10));

page.getContent();        // List<Student> - the actual data
page.getTotalElements();  // Total number of elements
page.getTotalPages();     // Total number of pages
page.getNumber();         // Current page number (0-based)
page.getSize();           // Page size
page.hasNext();           // Has next page?
page.hasPrevious();       // Has previous page?
page.isFirst();           // Is first page?
page.isLast();            // Is last page?
```

---

# 3.12 Sorting

```java
// Simple sorting
List<Student> findByStatus(StudentStatus status, Sort sort);

// Usage
Sort sort = Sort.by("lastName").ascending();
Sort sort = Sort.by("lastName").descending();
Sort sort = Sort.by("lastName").ascending()
                .and(Sort.by("firstName").ascending());

// Combine with pagination
PageRequest.of(0, 10, Sort.by("lastName").ascending());

// Direction enum
Sort.by(Sort.Direction.ASC, "lastName");
Sort.by(Sort.Direction.DESC, "enrollmentDate");
```

---

# 3.13 Slice vs Page

```java
// Page - knows total count (extra COUNT query)
Page<Student> findByStatus(StudentStatus status, Pageable pageable);

// Slice - doesn't know total (more efficient)
Slice<Student> findByStatus(StudentStatus status, Pageable pageable);
```

**When to use:**
- `Page` - When you need total count (pagination UI with page numbers)
- `Slice` - When you only need "Load More" functionality (infinite scroll)

---

# 3.14 Projections - Interface-Based

```java
// Define projection interface
public interface StudentNameProjection {
    String getFirstName();
    String getLastName();
    String getEmail();
}

// Use in repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    List<StudentNameProjection> findByStatus(StudentStatus status);
    
    @Query("SELECT s FROM Student s WHERE s.department.id = :deptId")
    List<StudentNameProjection> findNamesByDepartment(@Param("deptId") Long deptId);
}
```

Only selected fields are fetched from database!

---

# 3.15 Projections - Class-Based (DTO)

```java
// DTO class
public record StudentDTO(
    String firstName,
    String lastName,
    String email,
    String departmentName
) {}

// Repository with constructor expression
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    @Query("SELECT new com.example.demo.dto.StudentDTO(" +
           "s.firstName, s.lastName, s.email, s.department.name) " +
           "FROM Student s WHERE s.status = :status")
    List<StudentDTO> findStudentDTOsByStatus(@Param("status") StudentStatus status);
}
```

---

# 3.16 Specifications (Dynamic Queries)

```java
// Enable specifications
public interface StudentRepository extends 
    JpaRepository<Student, Long>,
    JpaSpecificationExecutor<Student> {
}

// Create specifications
public class StudentSpecs {
    
    public static Specification<Student> hasStatus(StudentStatus status) {
        return (root, query, cb) -> cb.equal(root.get("status"), status);
    }
    
    public static Specification<Student> nameLike(String name) {
        return (root, query, cb) -> 
            cb.like(cb.lower(root.get("firstName")), "%" + name.toLowerCase() + "%");
    }
    
    public static Specification<Student> inDepartment(Long deptId) {
        return (root, query, cb) -> 
            cb.equal(root.get("department").get("id"), deptId);
    }
}
```

---

# 3.17 Using Specifications

```java
@Service
public class StudentService {
    
    private final StudentRepository repository;
    
    public List<Student> searchStudents(
        StudentStatus status,
        String name,
        Long departmentId
    ) {
        Specification<Student> spec = Specification.where(null);
        
        if (status != null) {
            spec = spec.and(StudentSpecs.hasStatus(status));
        }
        if (name != null) {
            spec = spec.and(StudentSpecs.nameLike(name));
        }
        if (departmentId != null) {
            spec = spec.and(StudentSpecs.inDepartment(departmentId));
        }
        
        return repository.findAll(spec);
    }
}
```

---

# 3.18 Lab Exercise 3: Queries

**Tasks (45 minutes):**

1. Add derived query methods to `StudentRepository`:
   - Find by email
   - Find by status
   - Find by department name
   - Count students by status
   - Find top 5 by enrollment date

2. Add custom JPQL queries:
   - Find students enrolled after a date
   - Find students with specific email domain
   - Update status for multiple students

3. Implement pagination for listing students

4. Create a search endpoint with optional filters (name, status, department)

---

# 3.19 Query By Example

```java
// Create probe (example entity)
Student probe = new Student();
probe.setStatus(StudentStatus.ACTIVE);
probe.setLastName("Smith");

// Create matcher
ExampleMatcher matcher = ExampleMatcher.matching()
    .withIgnoreCase()
    .withStringMatcher(ExampleMatcher.StringMatcher.CONTAINING);

// Create example
Example<Student> example = Example.of(probe, matcher);

// Execute query
List<Student> results = repository.findAll(example);
```

---

# Module 3 Summary

**What We Covered:**
- Repository hierarchy
- Derived query methods
- @Query with JPQL and native SQL
- Modifying queries
- Pagination and sorting
- Projections (interface & class-based)
- Specifications for dynamic queries
- Query By Example

**Key Takeaways:**
- Method naming conventions auto-generate queries
- Use native queries for PostgreSQL-specific features
- Projections improve performance
- Specifications enable type-safe dynamic queries

---

---

# MODULE 4: Advanced Topics & Best Practices

**Duration: 2 Hours**

---

# 4.1 Transaction Management

```java
@Service
@Transactional(readOnly = true)  // Default for class
public class StudentService {
    
    private final StudentRepository repository;
    
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
        Student student = repository.findById(studentId).orElseThrow();
        // ... enrollment logic
    }
}
```

---

# 4.2 Transaction Attributes

```java
@Transactional(
    readOnly = false,                    // Read-write transaction
    timeout = 30,                        // 30 seconds timeout
    isolation = Isolation.READ_COMMITTED,// Isolation level
    propagation = Propagation.REQUIRED,  // Propagation behavior
    rollbackFor = Exception.class,       // Rollback on exception
    noRollbackFor = NotFoundException.class
)
```

**Common Propagation Types:**
- `REQUIRED` (default) - Use existing or create new
- `REQUIRES_NEW` - Always create new (suspends current)
- `MANDATORY` - Must have existing transaction
- `NEVER` - Must NOT have transaction

---

# 4.3 The N+1 Problem

```java
// BAD: N+1 queries!
List<Student> students = studentRepository.findAll();
for (Student student : students) {
    System.out.println(student.getDepartment().getName());  // Extra query each time!
}

// Query 1: SELECT * FROM students
// Query 2: SELECT * FROM departments WHERE id = 1
// Query 3: SELECT * FROM departments WHERE id = 2
// ... N more queries!
```

---

# 4.4 Solving N+1 with Join Fetch

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // JPQL Join Fetch
    @Query("SELECT s FROM Student s JOIN FETCH s.department")
    List<Student> findAllWithDepartment();
    
    // With condition
    @Query("SELECT s FROM Student s " +
           "JOIN FETCH s.department d " +
           "WHERE d.name = :deptName")
    List<Student> findByDepartmentWithFetch(@Param("deptName") String deptName);
}
```

**Result:** Single query with JOIN

---

# 4.5 Entity Graphs

```java
@Entity
@NamedEntityGraph(
    name = "Student.withDepartmentAndCourses",
    attributeNodes = {
        @NamedAttributeNode("department"),
        @NamedAttributeNode("courses")
    }
)
public class Student {
    // ...
}

// Repository usage
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    @EntityGraph(value = "Student.withDepartmentAndCourses")
    List<Student> findByStatus(StudentStatus status);
    
    @EntityGraph(attributePaths = {"department", "courses"})
    Optional<Student> findById(Long id);
}
```

---

# 4.6 Auditing - Setup

```java
// 1. Enable auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of("system");  // Or get from security context
    }
}

// 2. Create base entity
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    @CreatedBy
    private String createdBy;
    
    @LastModifiedBy
    private String updatedBy;
}
```

---

# 4.7 Auditing - Usage

```java
@Entity
@Table(name = "students")
public class Student extends BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String firstName;
    private String lastName;
    // ...
}

// Now all students automatically have:
// - createdAt (set on insert)
// - updatedAt (set on insert and update)
// - createdBy (set on insert)
// - updatedBy (set on insert and update)
```

---

# 4.8 Soft Delete

```java
@Entity
@Table(name = "students")
@Where(clause = "deleted = false")  // Hibernate filter
public class Student {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String firstName;
    
    private boolean deleted = false;
    
    private LocalDateTime deletedAt;
}

// Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // Automatically filters deleted records
    List<Student> findByStatus(StudentStatus status);
    
    // Include deleted records
    @Query("SELECT s FROM Student s WHERE s.id = :id")
    Optional<Student> findByIdIncludingDeleted(@Param("id") Long id);
}
```

---

# 4.9 Database Indexing

```java
@Entity
@Table(
    name = "students",
    indexes = {
        @Index(name = "idx_student_email", columnList = "email"),
        @Index(name = "idx_student_status", columnList = "status"),
        @Index(name = "idx_student_name", columnList = "lastName, firstName"),
        @Index(name = "idx_student_dept", columnList = "department_id")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uk_student_email", columnNames = "email")
    }
)
public class Student {
    // ...
}
```

---

# 4.10 Caching with Spring Cache

```java
// 1. Enable caching
@Configuration
@EnableCaching
public class CacheConfig {
}

// 2. Use in service
@Service
public class StudentService {
    
    @Cacheable(value = "students", key = "#id")
    public Student getById(Long id) {
        return repository.findById(id).orElseThrow();
    }
    
    @CacheEvict(value = "students", key = "#student.id")
    public Student update(Student student) {
        return repository.save(student);
    }
    
    @CacheEvict(value = "students", allEntries = true)
    public void clearCache() {
    }
}
```

---

# 4.11 Building REST API - Controller

```java
@RestController
@RequestMapping("/api/students")
@RequiredArgsConstructor
public class StudentController {
    
    private final StudentService studentService;
    
    @GetMapping
    public Page<StudentDTO> getAllStudents(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy
    ) {
        return studentService.getAllStudents(page, size, sortBy);
    }
    
    @GetMapping("/{id}")
    public StudentDTO getStudent(@PathVariable Long id) {
        return studentService.getById(id);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public StudentDTO createStudent(@Valid @RequestBody CreateStudentRequest request) {
        return studentService.create(request);
    }
}
```

---

# 4.12 Building REST API - Controller (Continued)

```java
@RestController
@RequestMapping("/api/students")
public class StudentController {
    
    // ... previous methods
    
    @PutMapping("/{id}")
    public StudentDTO updateStudent(
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
    
    @GetMapping("/search")
    public List<StudentDTO> searchStudents(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) StudentStatus status,
        @RequestParam(required = false) Long departmentId
    ) {
        return studentService.search(name, status, departmentId);
    }
}
```

---

# 4.13 DTO Pattern

```java
// Request DTO
public record CreateStudentRequest(
    @NotBlank String firstName,
    @NotBlank String lastName,
    @Email @NotBlank String email,
    Long departmentId
) {}

// Response DTO
public record StudentDTO(
    Long id,
    String firstName,
    String lastName,
    String email,
    StudentStatus status,
    String departmentName,
    LocalDateTime createdAt
) {
    public static StudentDTO fromEntity(Student student) {
        return new StudentDTO(
            student.getId(),
            student.getFirstName(),
            student.getLastName(),
            student.getEmail(),
            student.getStatus(),
            student.getDepartment() != null ? 
                student.getDepartment().getName() : null,
            student.getCreatedAt()
        );
    }
}
```

---

# 4.14 Service Layer

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class StudentService {
    
    private final StudentRepository studentRepository;
    private final DepartmentRepository departmentRepository;
    
    public Page<StudentDTO> getAllStudents(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return studentRepository.findAll(pageable).map(StudentDTO::fromEntity);
    }
    
    public StudentDTO getById(Long id) {
        return studentRepository.findById(id)
            .map(StudentDTO::fromEntity)
            .orElseThrow(() -> new StudentNotFoundException(id));
    }
    
    @Transactional
    public StudentDTO create(CreateStudentRequest request) {
        if (studentRepository.existsByEmail(request.email())) {
            throw new EmailAlreadyExistsException(request.email());
        }
        
        Student student = new Student();
        student.setFirstName(request.firstName());
        student.setLastName(request.lastName());
        student.setEmail(request.email());
        student.setStatus(StudentStatus.ACTIVE);
        
        if (request.departmentId() != null) {
            Department dept = departmentRepository.findById(request.departmentId())
                .orElseThrow(() -> new DepartmentNotFoundException(request.departmentId()));
            student.setDepartment(dept);
        }
        
        return StudentDTO.fromEntity(studentRepository.save(student));
    }
}
```

---

# 4.15 Exception Handling

```java
// Custom exceptions
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

# 4.16 Validation

```java
@Entity
public class Student {
    
    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50, message = "First name must be 2-50 characters")
    private String firstName;
    
    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 50, message = "Last name must be 2-50 characters")
    private String lastName;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;
    
    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;
    
    @Min(value = 0, message = "Credits must be non-negative")
    @Max(value = 200, message = "Credits cannot exceed 200")
    private Integer totalCredits;
}
```

---

# 4.17 Testing Repositories

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
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

# 4.18 Testing with Testcontainers

```java
@SpringBootTest
@Testcontainers
class StudentServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
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
    void shouldCreateAndRetrieveStudent() {
        CreateStudentRequest request = new CreateStudentRequest(
            "Jane", "Smith", "jane@test.com", null
        );
        
        StudentDTO created = studentService.create(request);
        StudentDTO retrieved = studentService.getById(created.id());
        
        assertThat(retrieved.firstName()).isEqualTo("Jane");
    }
}
```

---

# 4.19 Performance Best Practices

**Do:**
- Use `LAZY` fetch for collections
- Use projections when you need subset of fields
- Use pagination for large datasets
- Use batch operations for bulk inserts/updates
- Index frequently queried columns

**Don't:**
- Don't use `EAGER` fetch on multiple collections
- Don't fetch entire entities when you need few fields
- Don't load all data without pagination
- Don't ignore N+1 problems
- Don't use `IDENTITY` generator for batch inserts

---

# 4.20 Batch Insert Configuration

```properties
# application.properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
spring.jpa.properties.hibernate.batch_versioned_data=true
```

```java
@Transactional
public void saveAllInBatch(List<Student> students) {
    int batchSize = 50;
    for (int i = 0; i < students.size(); i++) {
        entityManager.persist(students.get(i));
        if (i > 0 && i % batchSize == 0) {
            entityManager.flush();
            entityManager.clear();
        }
    }
}
```

---

# 4.21 Lab Exercise 4: Complete Application

**Tasks (45 minutes):**

Build a complete Student Management REST API:

1. **Entities:** Student, Department, Course, Enrollment

2. **Features:**
   - CRUD operations for all entities
   - Search students with filters
   - Pagination and sorting
   - Enrollment management

3. **Best Practices:**
   - Use DTOs for requests/responses
   - Implement proper exception handling
   - Add auditing fields
   - Write at least 3 repository tests

---

# 4.22 Database Migration with Flyway

```properties
# application.properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.jpa.hibernate.ddl-auto=validate  # Don't auto-create, validate only
```

```sql
-- src/main/resources/db/migration/V1__create_students_table.sql
CREATE TABLE students (
    id BIGSERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    status VARCHAR(20),
    department_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_students_email ON students(email);
CREATE INDEX idx_students_status ON students(status);
```

---

# 4.23 Common Pitfalls to Avoid

| Pitfall | Solution |
|---------|----------|
| N+1 queries | Use JOIN FETCH or Entity Graphs |
| LazyInitializationException | Use DTOs or fetch within transaction |
| Detached entity passed to persist | Use merge() instead |
| Transaction not rolling back | Check exception type, use @Transactional properly |
| Slow queries | Add indexes, use pagination, optimize fetch strategy |
| Memory issues | Use pagination, avoid fetching large collections |

---

# Module 4 Summary

**What We Covered:**
- Transaction management
- N+1 problem and solutions
- Entity Graphs
- Auditing
- Soft delete pattern
- REST API design
- DTO pattern
- Exception handling
- Testing strategies
- Performance optimization

---

# Course Summary

**Module 1:** Spring Boot + JPA Basics
- Setup, configuration, first entity

**Module 2:** Entity Mapping & Relationships
- All relationship types, cascade, fetch strategies

**Module 3:** Repositories & Queries
- Derived queries, JPQL, native SQL, pagination

**Module 4:** Advanced Topics
- Transactions, N+1, caching, REST API, testing

---

# Quick Reference Card

```java
// Essential Annotations
@Entity @Table @Id @GeneratedValue @Column
@OneToOne @OneToMany @ManyToOne @ManyToMany
@JoinColumn @JoinTable
@Query @Modifying @Transactional
@CreatedDate @LastModifiedDate

// Repository Methods
save() findById() findAll() delete() count()

// Query Keywords
findBy... And Or Like Containing Between
OrderBy LessThan GreaterThan IsNull IsNotNull

// Fetch Strategies
FetchType.LAZY (default for collections)
FetchType.EAGER (avoid for collections)
JOIN FETCH (JPQL)
@EntityGraph (declarative)
```

---

# Resources for Further Learning

**Documentation:**
- Spring Data JPA: https://spring.io/projects/spring-data-jpa
- Hibernate: https://hibernate.org/orm/documentation
- PostgreSQL: https://www.postgresql.org/docs/

**Books:**
- "Spring in Action" by Craig Walls
- "Java Persistence with Hibernate" by Bauer & King

**Practice:**
- Build a blog application
- Build an e-commerce system
- Contribute to open source Spring projects

---

# Thank You!

**Questions?**

Remember:
- Practice is key
- Start simple, add complexity gradually
- Always consider performance implications
- Test your repositories!

**Happy Coding! 🚀**
