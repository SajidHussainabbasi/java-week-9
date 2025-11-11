# 📦 Understanding DTOs (Data Transfer Objects)

## 🎯 What is a DTO?

A **DTO (Data Transfer Object)** is a simple Java class used to **transfer data** between layers of your application, especially between:

```
Client → Controller → Service → Database
```

DTOs allow us to **control exactly what data is received and returned** in the API.

---

## ❓ Why Not Use Entities Directly?

Your **Entity** classes are tied to the **database**. They often include:

* JPA annotations (`@Entity`, `@OneToMany`, etc.)
* Internal identifiers
* Fields you don’t want exposed
* Relationships & DB-specific details

If you expose them directly in your API:

| Problem                                 | Example                                         |
|-----------------------------------------|-------------------------------------------------|
| Sensitive data can leak                 | Password, internal ID                           |
| Frontend becomes dependent on DB schema | Harder to change DB later                       |
| Cannot control API structure            | Fields sent to/from client cannot be restricted |
| Harder to validate input                | User could update fields they shouldn’t         |

✅ Using **DTOs prevents these issues.**

---

## ✅ Two Types of DTOs

| Type             | Purpose                                            |
|------------------|----------------------------------------------------|
| **Request DTO**  | Used when the **client sends data** (POST/PUT)     |
| **Response DTO** | Used when the **API returns data** (GET endpoints) |

---

## 📝 Example: Contact API

### Entity (Database Model)

```java
@Entity
public class Contact 
{
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;
    private String email;

    // getters & setters
}
```

### Request DTO (Client → Server)

```java
public class ContactRequestDTO 
{
    @NotBlank(message = "Name is required")
    private String name;

    @NotBlank(message = "Phone is required")
    private String phone;

    @Email @NotBlank(message = "Email is required")
    private String email;

    // getters & setters
}
```

### Response DTO (Server → Client)

```java
public class ContactResponseDTO 
{
    private Long id;
    private String name;
    private String phone;
    private String email;

    // getters & setters
}
```

---

## 🔄 Converting Entity ↔ DTO (Inside Service Layer)

```java
@Service
public class ContactService 
{

    public ContactResponseDTO toDTO(Contact c) 
    {
        ContactResponseDTO dto = new ContactResponseDTO();
        dto.setId(c.getId());
        dto.setName(c.getName());
        dto.setPhone(c.getPhone());
        dto.setEmail(c.getEmail());
        return dto;
    }

    public Contact toEntity(ContactRequestDTO dto) 
    {
        Contact c = new Contact();
        c.setName(dto.getName());
        c.setPhone(dto.getPhone());
        c.setEmail(dto.getEmail());
        return c;
    }
}
```

---

## 🌍 Controller Example Using DTOs

```java
@PostMapping("/contacts")
public ResponseEntity<ContactResponseDTO> create(@Valid @RequestBody ContactRequestDTO dto) 
{
    Contact saved = service.save(service.toEntity(dto));
    return ResponseEntity.status(201).body(service.toDTO(saved));
}
```

---

## 🍽 Example: Recipe API DTOs

### Request DTO

```java
public class RecipeRequestDTO 
{
    @NotBlank private String title;
    @NotBlank private String ingredients;
    @NotBlank private String category;
}
```

### Response DTO

```java
public class RecipeResponseDTO
{
    private Long id;
    private String title;
    private String ingredients;
    private String category;
}
```

### Convert inside Service

```java
private RecipeResponseDTO toDTO(Recipe r) 
{
    RecipeResponseDTO dto = new RecipeResponseDTO();
    dto.setId(r.getId());
    dto.setTitle(r.getTitle());
    dto.setIngredients(r.getIngredients());
    dto.setCategory(r.getCategory());
    return dto;
}
```

---

## 🧠 Key Benefits of DTOs

| Benefit                    | Explanation                                        |
|----------------------------|----------------------------------------------------|
| **Security**               | You choose what data leaves your system            |
| **Validation**             | Apply `@NotBlank`, `@Email`, etc. on DTOs          |
| **Clean API**              | Your API models are separate from database models  |
| **Flexibility**            | Change database structure without breaking the API |
| **Separation of Concerns** | Controllers deal with API data, not DB data        |

---

## ✅ Summary

| Term             | Meaning                              |
|------------------|--------------------------------------|
| **Entity**       | Represents database tables           |
| **Request DTO**  | Data the client sends                |
| **Response DTO** | Data returned to client              |
| **Conversion**   | Entity ↔ DTO (done in service layer) |

DTOs make your API **safer, cleaner, and easier to maintain**.

---
