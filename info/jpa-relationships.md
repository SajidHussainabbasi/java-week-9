# 📚 JPA Relationships in Spring Boot

When working with databases in a Spring Boot application using **JPA / Hibernate**, we use **annotations** to define how tables relate to each other.  
These annotations describe **entity relationships** — how data in one table connects to another.

---

## 🧱 Types of Relationships

| Relationship                  | Meaning                                                      | Example                                                    |
|-------------------------------|--------------------------------------------------------------|------------------------------------------------------------|
| **One-to-One**                | One record relates to exactly one other record               | A User has one Profile                                     |
| **One-to-Many / Many-to-One** | One record relates to many, but each of those relates to one | A Movie has many Reviews; each Review belongs to one Movie |
| **Many-to-Many**              | Many records relate to many other records                    | Students enrolled in Courses                               |

---

## 1) **One-to-Many / Many-to-One (Most Common)**

### Example Scenario

- A **Movie** can have **many Reviews**
- A **Review** belongs to **one Movie**

### 🏛 Database Structure

| Movie Table | Review Table              |
|-------------|---------------------------|
| id (PK)     | id (PK)                   |
| title       | rating                    |
| ...         | comment                   |
|             | movie_id (FK to Movie.id) |

---

### ✅ Movie Entity (Parent)

```java
@Entity
public class Movie {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    // One movie has many reviews
    @OneToMany(mappedBy = "movie", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonManagedReference
    private List<Review> reviews = new ArrayList<>();
}
````

### ✅ Review Entity (Child)

```java
@Entity
public class Review {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int rating;
    private String comment;

    // Many reviews belong to one movie
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "movie_id")
    @JsonBackReference
    private Movie movie;
}
```

---

### 🧠 Explanation of Important Annotations

| Annotation                  | Used On                  | Meaning                                              |
|-----------------------------|--------------------------|------------------------------------------------------|
| `@OneToMany`                | Parent                   | A single parent has multiple children                |
| `mappedBy`                  | Parent side              | Points to the field name in the child entity         |
| `@ManyToOne`                | Child                    | Many children belong to one parent                   |
| `@JoinColumn("movie_id")`   | Child table              | Creates the foreign key column                       |
| `@JsonManagedReference`     | Parent side              | Indicates forward direction for JSON serialization   |
| `@JsonBackReference`        | Child side               | Prevents infinite recursion in JSON responses        |
| `fetch = FetchType.LAZY`    | Default for `@ManyToOne` | Load related data only when needed                   |
| `cascade = CascadeType.ALL` | Parent                   | When parent is saved/removed, children follow        |
| `orphanRemoval = true`      | Parent                   | If a child is removed from the list → delete from DB |

---

## 2) **One-to-One**

### Example: Each User has One Profile

```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private Profile profile;
}
```

```java
@Entity
public class Profile {
    @Id @GeneratedValue
    private Long id;

    @OneToOne(mappedBy = "profile")
    private User user;
}
```

---

## 3) **Many-to-Many**

### Example: Students enrolled in Courses

```java
@Entity
public class Student {
    @Id @GeneratedValue
    private Long id;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses;
}
```

```java
@Entity
public class Course {
    @Id @GeneratedValue
    private Long id;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students;
}
```

---

## 🚀 Why JSON Serialization Needs Special Handling

Bidirectional relationships can cause infinite loops in JSON:

```txt
Movie → Review → Movie → Review → ...
```

To prevent this, we use:

| Annotation              | Effect                                   |
|-------------------------|------------------------------------------|
| `@JsonManagedReference` | Serialize this side normally             |
| `@JsonBackReference`    | Do not serialize this side (breaks loop) |

**Alternative:** You can replace both with `@JsonIgnore` on the child side if you don’t need the parent inside responses.

---

## 🎯 Best Practice for Production (Recommended)

Use **DTOs** instead of returning Entity classes directly.

Example:

```java
public record ReviewDTO(Long id, int rating, String comment) {}
```

This keeps API responses clean and avoids leaking DB structure.

---

## ✅ Summary

| Use Case                  | Best Relationship                              | Important Notes                                     |
|---------------------------|------------------------------------------------|-----------------------------------------------------|
| One parent, many children | `@OneToMany` + `@ManyToOne`                    | Remember `mappedBy` + `@JoinColumn`                 |
| One-to-one mapping        | `@OneToOne`                                    | Put FK on the owning side                           |
| Many-to-many              | `@ManyToMany`                                  | Often better to convert into two One-to-Many tables |
| JSON recursion issues     | `@JsonManagedReference` / `@JsonBackReference` | Or use DTOs                                         |

---
