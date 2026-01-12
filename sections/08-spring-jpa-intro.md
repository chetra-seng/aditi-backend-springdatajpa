# Introduction to Spring & JPA

## Framework Fundamentals

---

# What is Spring Framework?

Spring is a comprehensive framework for enterprise Java development.

<v-clicks>

**Core Concepts:**
- Inversion of Control (IoC)
- Dependency Injection (DI)
- Aspect-Oriented Programming (AOP)

**Why Spring?**
- Simplifies Java development
- Promotes loose coupling
- Extensive ecosystem

</v-clicks>

---

# What is Spring Boot?

Spring Boot makes it easy to create stand-alone, production-grade Spring applications.

<v-clicks>

- **Convention over Configuration** - Sensible defaults
- **Embedded Server** - No external server needed
- **Auto-configuration** - Smart defaults
- **Production-ready** - Health checks, metrics out of the box
- **No XML** - Java-based configuration

</v-clicks>

---

# What is JPA?

**Java Persistence API (JPA)** is a specification for Object-Relational Mapping (ORM).

```
┌─────────────┐         ┌─────────────┐
│   Java      │   JPA   │  Database   │
│   Objects   │ ◄─────► │   Tables    │
└─────────────┘         └─────────────┘
```

<v-click>

**JPA is NOT an implementation** — it's a specification!

</v-click>

---

# JPA vs Hibernate vs Spring Data JPA

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

# Project Setup

**Step 1: Go to https://start.spring.io**

**Dependencies to add:**
- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Spring Boot DevTools (optional)
- Lombok (optional, but recommended)

---

# Project Structure

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
│       └── application.properties
└── test/
```

---

# Configuring PostgreSQL Connection

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

# Understanding ddl-auto Options

| Value | Description | Use Case |
|-------|-------------|----------|
| `none` | No schema management | Production |
| `validate` | Validates schema, no changes | Production |
| `update` | Updates schema (additive only) | Development |
| `create` | Creates schema, destroys previous | Testing |
| `create-drop` | Create on start, drop on stop | Unit tests |

<v-click>

**Never use `create` or `create-drop` in production!**

</v-click>

---

# Your First Entity

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

# Key JPA Annotations

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Marks class as JPA entity |
| `@Table` | Specifies table name |
| `@Id` | Marks primary key field |
| `@GeneratedValue` | Auto-generate ID values |
| `@Column` | Customize column mapping |
| `@Transient` | Field not persisted |

---

# ID Generation Strategies

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
```

---

# Using Lombok

```java
import lombok.*;
import jakarta.persistence.*;

@Entity
@Table(name = "students")
@Data                    // Getters, setters, toString, equals, hashCode
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

# Creating Your First Repository

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

<v-click>

**What you get automatically:**
- `save()`, `saveAll()`
- `findById()`, `findAll()`
- `deleteById()`, `delete()`, `deleteAll()`
- `count()`, `existsById()`

</v-click>

---

# Testing Your Setup

```java
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

# Lab Exercise: Setup & First Entity

**Tasks (30 minutes):**

<v-clicks>

1. Create a new Spring Boot project with required dependencies
2. Configure PostgreSQL connection
3. Create a `Student` entity with fields:
   - id (Long, auto-generated)
   - firstName, lastName (String)
   - email (String, unique)
   - enrollmentDate (LocalDate)
4. Create `StudentRepository` interface
5. Run the application and verify table creation

</v-clicks>

---

# Module 8 Summary

<v-clicks>

- Spring Boot simplifies Java application development
- JPA is a specification, Hibernate is the implementation
- Spring Data JPA reduces boilerplate code
- Entities map Java classes to database tables
- Repositories provide automatic CRUD operations
- Use `@Entity`, `@Id`, `@GeneratedValue` annotations

</v-clicks>
