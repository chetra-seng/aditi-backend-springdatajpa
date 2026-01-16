---
layout: center
---
# Query Methods

## Derived Queries and JPQL

---

# Derived Query Methods

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

# Query Method Keywords

| Keyword | Sample | SQL Equivalent |
|---------|--------|----------------|
| `And` | `findByFirstNameAndLastName` | `WHERE first_name = ? AND last_name = ?` |
| `Or` | `findByFirstNameOrLastName` | `WHERE first_name = ? OR last_name = ?` |
| `Between` | `findByAgeBetween` | `WHERE age BETWEEN ? AND ?` |
| `LessThan` | `findByAgeLessThan` | `WHERE age < ?` |
| `GreaterThan` | `findByAgeGreaterThan` | `WHERE age > ?` |
| `Like` | `findByFirstNameLike` | `WHERE first_name LIKE ?` |
| `Containing` | `findByNameContaining` | `WHERE name LIKE %?%` |

---

# More Query Keywords

| Keyword | Sample | SQL Equivalent |
|---------|--------|----------------|
| `StartingWith` | `findByNameStartingWith` | `WHERE name LIKE ?%` |
| `EndingWith` | `findByNameEndingWith` | `WHERE name LIKE %?` |
| `IsNull` | `findByEmailIsNull` | `WHERE email IS NULL` |
| `IsNotNull` | `findByEmailIsNotNull` | `WHERE email IS NOT NULL` |
| `In` | `findByStatusIn` | `WHERE status IN (?)` |
| `OrderBy` | `findByStatusOrderByNameAsc` | `ORDER BY name ASC` |
| `True/False` | `findByActiveTrue` | `WHERE active = TRUE` |

---

# Derived Query Examples

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

    // Nested properties (joins automatically)
    List<Student> findByDepartmentName(String departmentName);
}
```

---

# @Query Annotation - JPQL

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

# @Query - Native SQL

```java
public interface StudentRepository extends JpaRepository<Student, Long> {

    // Native PostgreSQL query
    @Query(value = "SELECT * FROM students WHERE email LIKE %?1%",
           nativeQuery = true)
    List<Student> findByEmailContainingNative(String email);

    // Native query with join
    @Query(value = "SELECT * FROM students s " +
                   "JOIN departments d ON s.department_id = d.id " +
                   "WHERE d.name = :deptName",
           nativeQuery = true)
    List<Student> findByDepartmentNameNative(@Param("deptName") String deptName);
}
```

---

# Modifying Queries

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
    int deactivateOldStudents(
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

# Query with Joins

```java
public interface StudentRepository extends JpaRepository<Student, Long> {

    // Join fetch to avoid N+1
    @Query("SELECT s FROM Student s JOIN FETCH s.department")
    List<Student> findAllWithDepartment();

    // Join fetch with condition
    @Query("SELECT s FROM Student s " +
           "JOIN FETCH s.department d " +
           "WHERE d.name = :deptName")
    List<Student> findByDepartmentWithFetch(@Param("deptName") String deptName);

    // Multiple joins
    @Query("SELECT s FROM Student s " +
           "JOIN FETCH s.department " +
           "JOIN FETCH s.courses " +
           "WHERE s.id = :id")
    Optional<Student> findByIdWithDetails(@Param("id") Long id);
}
```

---

# Named Queries

```java
@Entity
@NamedQuery(
    name = "Student.findByStatus",
    query = "SELECT s FROM Student s WHERE s.status = :status"
)
@NamedQuery(
    name = "Student.findActiveStudents",
    query = "SELECT s FROM Student s WHERE s.status = 'ACTIVE'"
)
public class Student {
    // ...
}

// Repository automatically uses named query
public interface StudentRepository extends JpaRepository<Student, Long> {
    List<Student> findByStatus(@Param("status") StudentStatus status);
}
```

---

# Practice: Query Methods

**Tasks (45 minutes):**

<v-clicks>

1. Add derived query methods to `StudentRepository`:
   - Find by email
   - Find by status
   - Find by department name
   - Count by status
   - Find top 5 by enrollment date
2. Add custom JPQL queries:
   - Find students enrolled after a date
   - Find students with specific email domain
3. Add a modifying query to update status
4. Test all query methods

</v-clicks>

---

# Key Takeaways

<v-clicks>

- Method names generate queries automatically
- Keywords: `And`, `Or`, `Like`, `Between`, `OrderBy`
- `@Query` for custom JPQL or native SQL
- Named parameters with `@Param`
- `@Modifying` for UPDATE/DELETE queries
- Use `JOIN FETCH` to avoid N+1 problems
- Native queries for PostgreSQL-specific features

</v-clicks>
