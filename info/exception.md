# 🚨 Exception Handling in Spring Boot

When something goes wrong in your API — invalid input, missing data, wrong format — we want to return
**clear and friendly error messages**, not long stack traces.
Spring Boot gives us a way to handle exceptions **globally** using:

```
@RestControllerAdvice + @ExceptionHandler
```

This keeps **controllers clean** and handles errors **in one place**.

---

## ✅ Why Do We Need Global Exception Handling?

Without a handler, validation or runtime errors return messy responses:

```
400 Bad Request
Full stack trace shown → confusing for clients
```

With a global handler, we return:

```json
{
  "email": "Email must be valid",
  "name": "Name cannot be blank"
}
```

Much cleaner and ready for frontend consumption ✅

---

## 🧱 What Exceptions Do We Usually Handle?

| Exception                         | When it Happens                                | Example                                                |
|-----------------------------------|------------------------------------------------|--------------------------------------------------------|
| `MethodArgumentNotValidException` | When `@Valid` fails on **RequestBody**         | Missing/invalid fields in JSON                         |
| `ConstraintViolationException`    | When parameter validation fails (`@Validated`) | `@Min`, `@Email` on `@RequestParam` or `@PathVariable` |
| `RuntimeException` (Optional)     | For custom errors                              | "Product not found"                                    |

---

## ✋ Step 1 — Ensure Controllers Use `@Valid`

```java
@PostMapping
public ResponseEntity<ContactResponseDTO> create(@Valid @RequestBody ContactRequestDTO dto) 
{
    return ResponseEntity.status(201).body(service.create(dto));
}
```

---

## 🛡️ Step 2 — Create Global Exception Handler

```
src/main/java/.../exception/GlobalExceptionHandler.java
```

```java
package com.example.recipeapi.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import jakarta.validation.ConstraintViolationException;
import org.springframework.web.bind.annotation.*;

import java.util.*;

@RestControllerAdvice
public class GlobalExceptionHandler 
{

    // Handles @Valid failures (RequestBody validation)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handleValidationErrors(MethodArgumentNotValidException ex) 
    {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(err ->
                errors.put(err.getField(), err.getDefaultMessage()));
        return errors;
    }

    // Handles @Validated failures (PathVariable / RequestParam validation)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(ConstraintViolationException.class)
    public Map<String, String> handleConstraintViolation(ConstraintViolationException ex) 
    {
        Map<String, String> errors = new HashMap<>();
        ex.getConstraintViolations().forEach(v ->
                errors.put(v.getPropertyPath().toString(), v.getMessage()));
        return errors;
    }

    // Optional: Handle custom exceptions
    @ResponseStatus(HttpStatus.NOT_FOUND)
    @ExceptionHandler(RuntimeException.class)
    public Map<String, String> handleRuntime(RuntimeException ex) 
    {
        return Map.of("error", ex.getMessage());
    }
}
```

---

## 🧠 When to Use `@Valid` vs `@Validated`

| Annotation   | Used For                                                     | Example                                |
|--------------|--------------------------------------------------------------|----------------------------------------|
| `@Valid`     | Validating **RequestBody DTOs**                              | `@PostMapping` JSON input              |
| `@Validated` | Validating **URL inputs** (`@PathVariable`, `@RequestParam`) | `GET /products/low-stock?threshold=-1` |

### Example

```java
@RestController
@RequestMapping("/products")
@Validated // ✅ enables validation on request params
public class ProductController 
{

    @GetMapping("/low-stock")
    public ResponseEntity<?> lowStock(
        @RequestParam(defaultValue = "5") @Min(0) int threshold) 
    {
        ...
    }
}
```

---

## 🧪 Example Validation Error Response

### If client sends invalid JSON:

```json
POST /recipes
{
  "title": "",
  "ingredients": "",
  "category": ""
}
```

### API Response:

```json
{
  "title": "Title cannot be blank",
  "ingredients": "Ingredients cannot be blank",
  "category": "Category cannot be blank"
}
```

This is exactly what frontend developers want ✅

---

## ✅ Summary

| Concept                   | Meaning                           |
|---------------------------|-----------------------------------|
| `@RestControllerAdvice`   | Enables global exception handling |
| `@ExceptionHandler`       | Catches specific exception types  |
| `@Valid`                  | Validates request body (DTOs)     |
| `@Validated`              | Validates parameters in URLs      |
| Clean JSON error messages | Easy for frontend to display      |

---

## 🎉 Result

Your API now:

* Returns **clean and readable errors**
* Validates **both** request bodies and query parameters
* Has a **single centralized place** for error handling
* Is **professional and production-ready**

---


