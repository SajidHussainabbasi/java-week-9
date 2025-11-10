
# JPA Repository with PostgreSQL

## 🧩 What is JPA?

**JPA (Java Persistence API)** is a specification that defines how Java objects (entities) map to database tables.  
Spring Data JPA provides an abstraction layer over JPA, making database access simple and powerful.

---

## 🗄️ What is a JPA Repository?

A **JPA repository** is an interface that allows you to perform CRUD (Create, Read, Update, Delete) operations directly on your entities.

It extends:

- `CrudRepository` — basic CRUD methods.
- `PagingAndSortingRepository` — adds pagination and sorting.
- `JpaRepository` — adds extra JPA-related features.

Example:

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

## ⚙️ Database Configuration (PostgreSQL)

In your `application.properties` (or `application.yml`):

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydatabase
spring.datasource.username=postgres
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

---

## 🧱 Entity Example

```java
import jakarta.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Getters and Setters
}
```

---

## 🧰 Repository Example

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    // You can add custom query methods here
    User findByEmail(String email);
}
```

---

## 🚀 Common JPA Repository Methods

| Method                    | Description                                                      | Example                               |
|---------------------------|------------------------------------------------------------------|---------------------------------------|
| `findAll()`               | Returns all entities from the table.                             | `userRepository.findAll()`            |
| `findById(ID id)`         | Returns an entity by its ID (wrapped in `Optional`).             | `userRepository.findById(1L)`         |
| `save(Entity e)`          | Saves a new entity or updates an existing one.                   | `userRepository.save(new User(...))`  |
| `deleteById(ID id)`       | Deletes an entity by its ID.                                     | `userRepository.deleteById(1L)`       |
| `delete(Entity e)`        | Deletes a specific entity instance.                              | `userRepository.delete(user)`         |
| `count()`                 | Returns the total number of entities.                            | `userRepository.count()`              |
| `existsById(ID id)`       | Checks if an entity exists by ID.                                | `userRepository.existsById(1L)`       |
| `flush()`                 | Forces the persistence context to synchronize with the database. | `userRepository.flush()`              |
| `getReferenceById(ID id)` | Returns a reference to the entity (lazy loading).                | `userRepository.getReferenceById(1L)` |

---

## 🧠 Custom Finder Methods

Spring Data JPA can automatically generate queries based on method names.

Examples:

```java
List<User> findByName(String name);
List<User> findByEmailContaining(String keyword);
List<User> findByNameAndEmail(String name, String email);
List<User> findByIdBetween(Long start, Long end);
```

---

## 🧾 Example Controller

```java
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/users")
public class UserController {
    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userRepository.save(user);
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found"));
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userRepository.deleteById(id);
    }
}
```

---
