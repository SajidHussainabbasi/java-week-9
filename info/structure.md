# 🧭 Project Structure Overview

In a well-structured Spring Boot project, your code is usually organized into **4 main packages**:

```txt
com.example.projectname
│
├── model
├── repository
├── service
└── controller
```

Each layer has a **specific responsibility**, making your application easier to maintain, test, and scale.

---

## 🧱 1. `model` (or `entity`)

**Purpose:**  
This package contains all your **data models**, which are mapped to database tables using **JPA annotations**.

**Common annotations:**

- `@Entity` → marks the class as a database entity.
- `@Table(name = "table_name")` → (optional) specifies the table name.
- `@Id` → marks the primary key.
- `@GeneratedValue` → defines how the ID is auto-generated.

**Example:**

```java
package com.example.projectname.model;

import jakarta.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Getters, setters, constructors
}
```

🧠 Think of the **model** as the *blueprint* of your database tables.

---

### 🧰 2. `repository`

**Purpose:**
This layer interacts **directly with the database**.
You use **Spring Data JPA** interfaces like `JpaRepository` to perform CRUD operations.

**Example:**

```java
package com.example.projectname.repository;

import com.example.projectname.model.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

✅ The repository layer should only handle **data access**, not business logic.

---

### ⚙️ 3. `service`

**Purpose:**
This layer contains the **business logic** of your application.
It connects the **controller** and **repository** layers.

**Common annotations:**

* `@Service` → marks the class as a service component (managed by Spring).
* `@Transactional` → ensures that database operations are executed as a single transaction.

**Example:**

```java
package com.example.projectname.service;

import com.example.projectname.model.User;
import com.example.projectname.repository.UserRepository;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User getUserById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found"));
    }

    public User saveUser(User user) {
        return userRepository.save(user);
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

🧠 The service layer helps **separate business logic** from database logic and makes your code reusable.

---

### 🌐 4. `controller`

**Purpose:**
This is the **entry point** for API requests.
It handles HTTP requests (`GET`, `POST`, `PUT`, `DELETE`) and calls the service layer.

**Common annotations:**

* `@RestController` → marks the class as a REST API controller.
* `@RequestMapping("/path")` → sets the base URL for the controller.
* `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` → handle HTTP methods.

**Example:**

```java
package com.example.projectname.controller;

import com.example.projectname.model.User;
import com.example.projectname.service.UserService;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.saveUser(user);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

🧠 The controller’s job is to **receive requests**, **delegate logic to the service**, and **return responses**.

---

## 🧩 How the Layers Work Together

```txt
[Client / Frontend] 
       ↓
[Controller] → handles HTTP request (API endpoint)
       ↓
[Service] → contains business logic
       ↓
[Repository] → communicates with database
       ↓
[Database] → PostgreSQL
```

---

## ✅ Summary

| Package      | Role                               | Example Class         |
|--------------|------------------------------------|-----------------------|
| `model`      | Defines entities / database tables | `User.java`           |
| `repository` | Handles data access (CRUD)         | `UserRepository.java` |
| `service`    | Contains business logic            | `UserService.java`    |
| `controller` | Handles HTTP endpoints             | `UserController.java` |

---
