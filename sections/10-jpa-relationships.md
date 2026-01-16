# JPA Relationships

## OneToOne, OneToMany, ManyToMany

---

# Relationship Types Overview

```
┌─────────────────────────────────────────────────────────┐
│                    JPA Relationships                     │
├─────────────────────────────────────────────────────────┤
│  @OneToOne    │  Student ←──────► StudentProfile        │
│  @OneToMany   │  Department ←────► Many Students        │
│  @ManyToOne   │  Many Students ────► One Department     │
│  @ManyToMany  │  Students ◄──────► Courses              │
└─────────────────────────────────────────────────────────┘
```

---

# @ManyToOne / @OneToMany

The most common relationship type.

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "department")
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

# Understanding Owning Side

**The owning side:**
- Contains the foreign key in the database
- Does NOT have `mappedBy` attribute
- Changes to this side are persisted

**The inverse side:**
- Has `mappedBy` attribute
- Changes are NOT automatically persisted

```java
// ✅ Correct - updating owning side
student.setDepartment(department);

// ❌ Wrong - inverse side only, won't persist relationship
department.getStudents().add(student);

// ✅ Best practice - update both sides
student.setDepartment(department);
department.getStudents().add(student);
```

---

# @OneToOne Relationship

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id")
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

# @ManyToMany Relationship

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

# ManyToMany with Extra Columns

When the join table needs additional columns, create an entity:

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
}
```

---

# Cascade Types

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

<v-click>

**Be careful with `REMOVE`** — can cause unintended deletions!

</v-click>

---

# Fetch Types

```java
// EAGER - Load immediately with parent
@ManyToOne(fetch = FetchType.EAGER)  // Default for @ManyToOne, @OneToOne

// LAZY - Load on demand (when accessed)
@OneToMany(fetch = FetchType.LAZY)   // Default for @OneToMany, @ManyToMany
```

<v-click>

**Best Practice:**
- Use `LAZY` for collections (`@OneToMany`, `@ManyToMany`)
- Use `LAZY` for `@ManyToOne` when not always needed
- Never use `EAGER` on multiple collections

</v-click>

---

# Orphan Removal

```java
@Entity
public class Department {

    @OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,
        orphanRemoval = true  // Delete students when removed from list
    )
    private List<Student> students = new ArrayList<>();
}

// This will DELETE the student from database
department.getStudents().remove(student);
```

---

# Helper Methods for Bidirectional

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

# Practice: JPA Relationships

**Tasks (45 minutes):**

<v-clicks>

1. Create `Department` entity with OneToMany to Student
2. Add ManyToOne from Student to Department
3. Create `Course` entity
4. Create `Enrollment` entity for Student-Course relationship
5. Add helper methods for bidirectional relationships
6. Test creating students with departments and enrollments

</v-clicks>

---

# Key Takeaways

<v-clicks>

- `@ManyToOne` / `@OneToMany` - Most common relationship
- `@OneToOne` - Single related entity
- `@ManyToMany` - Many-to-many with join table
- **Owning side** has the foreign key (no mappedBy)
- **Inverse side** has mappedBy
- Always update the owning side
- Use `LAZY` fetching for collections
- Use helper methods for bidirectional relationships

</v-clicks>
