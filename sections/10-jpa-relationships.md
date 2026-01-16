---
layout: center
---
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

Students can enroll in many courses, courses have many students.

```
students                student_courses           courses
┌────┬──────┐          ┌────────┬─────────┐     ┌────┬─────────┐
│ 1  │ John │─────────▶│ 1      │ 101     │────▶│101 │ Math    │
│ 2  │ Jane │─────┐    │ 1      │ 102     │  ┌─▶│102 │ Physics │
└────┴──────┘     │    │ 2      │ 102     │──┘  └────┴─────────┘
                  └───▶│ 2      │ 103     │
                       └────────┴─────────┘
                       Join Table
```

---

# @ManyToMany - Owner Side

**Student** owns the relationship, defines the join table:

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
```

---

# @ManyToMany - Inverse Side

**Course** references back with `mappedBy`:

```java
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

<v-click>

**Key Point:** `mappedBy` means "I don't own this, Student does"

</v-click>

---

# ManyToMany with Extra Columns - Option 1

**Using Generated ID:**

```java
@Entity
@Table(name = "enrollments")
public class Enrollment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;  // ← Separate ID

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

# ManyToMany with Extra Columns - Option 2

**Using Composite Key - Step 1:**

Create an `@Embeddable` composite key class:

```java
@Embeddable
public class EnrollmentId implements Serializable {
    private Long studentId;
    private Long courseId;

    // Constructor, equals, and hashCode required
    public EnrollmentId() {}

    public EnrollmentId(Long studentId, Long courseId) {
        this.studentId = studentId;
        this.courseId = courseId;
    }

    // equals() and hashCode() must be implemented
}
```
---

# Composite Key

<v-click>

**Why Composite Key?**
- Natural key from both foreign keys
- No extra surrogate ID needed
- Enforces uniqueness at database level

</v-click>

---

# ManyToMany with Extra Columns - Option 2 (cont.)

**Using Composite Key - Step 2:**

Use the composite key in the entity with `@MapsId`:

```java
@Entity
public class Enrollment {
    @EmbeddedId
    private EnrollmentId id;  // ← Composite key

    @ManyToOne
    @MapsId("studentId")
    @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne
    @MapsId("courseId")
    @JoinColumn(name = "course_id")
    private Course course;

    private LocalDate enrollmentDate;
    private String grade;
}
```

**`@MapsId` links the composite key fields to the entity relationships**

---

# Practice: JPA Relationships



<v-clicks>

1. Create `Department` entity with OneToMany to Student
2. Add ManyToOne from Student to Department
3. Create `Course` entity
4. Create `Enrollment` entity for Student-Course relationship
5. Test creating students with departments and enrollments

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
- Use helper methods for bidirectional relationships

</v-clicks>
