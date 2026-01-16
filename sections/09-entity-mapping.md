# Entity Mapping Basics

## Java Objects to Database Tables

---

# Entity Lifecycle States

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

# Column Mapping Options

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

# Data Types Mapping

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

# Temporal Types

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
}
```

---

# Enum Mapping

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

<v-click>

**Always use `EnumType.STRING`** — ordinal breaks if enum order changes!

</v-click>

---

# Embedded Objects

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
}

@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Embedded
    private Address address;  // All fields in same table
}
```

**Result:** Single table with street, city, state, zip_code columns.

---

# Auditing Fields

```java
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;
    private String lastName;

    @Column(updatable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

---

# Using Spring Data Auditing

```java
// 1. Enable auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig {}

// 2. Use auditing annotations
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

---

# Complete Entity Example

```java
@Entity
@Table(name = "students")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
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
    private StudentStatus status;

    private LocalDate enrollmentDate;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

---

# Lab Exercise: Entity Mapping

**Tasks (30 minutes):**

<v-clicks>

1. Add `StudentStatus` enum to your Student entity
2. Add auditing fields (createdAt, updatedAt)
3. Create an `Address` embeddable and add to Student
4. Create a `Course` entity with:
   - id, title, description
   - credits (Integer)
   - status enum (ACTIVE, INACTIVE)
5. Test persistence of all new fields

</v-clicks>

---

# Key Takeaways

<v-clicks>

- Entity lifecycle: Transient → Managed → Detached
- `@Column` customizes column mapping
- Java types map automatically to PostgreSQL types
- Use `@Enumerated(EnumType.STRING)` for enums
- `@Embedded` / `@Embeddable` for value objects
- Spring Data Auditing auto-populates timestamps
- `@PrePersist` / `@PreUpdate` for custom callbacks

</v-clicks>
