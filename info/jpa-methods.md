# 📘 Spring Data JPA — Derived Query Methods Guide

Spring Data JPA can auto-generate SQL queries **based on method names**.
You don’t write SQL — instead, you follow specific naming patterns.

> ✨ If you follow the **right keywords + entity field names**, Spring implements the query for you!

---

## ✅ Basic Structure

```java
findBy + FieldName + Condition
readBy + FieldName + Condition
getBy + FieldName + Condition
```

**Examples:**

```java
findByTitle(String title)
findByAuthor(String author)
```

---

## 🎯 Supported Keywords

### 1️⃣ **Equality & Comparison**

| Keyword                | Meaning   | Example                           |
|------------------------|-----------|-----------------------------------|
| `findBy`               | Equals    | `findByTitle(String title)`       |
| `findBy...Not`         | Not equal | `findByAuthorNot(String author)`  |
| `findBy...LessThan`    | `<`       | `findByYearLessThan(int year)`    |
| `findBy...GreaterThan` | `>`       | `findByYearGreaterThan(int year)` |
| `findBy...Between`     | Range     | `findByYearBetween(1990, 2020)`   |

---

### 2️⃣ **String Matching**

| Keyword      | Meaning          | Example                          |
|--------------|------------------|----------------------------------|
| `Containing` | LIKE %val%       | `findByTitleContaining("code")`  |
| `StartsWith` | LIKE val%        | `findByAuthorStartsWith("Rob")`  |
| `EndsWith`   | LIKE %val        | `findByAuthorEndsWith("Martin")` |
| `IgnoreCase` | Case-insensitive | `findByGenreIgnoreCase("Drama")` |

---

### 3️⃣ **Boolean Operators**

| Keyword | Example                                             |
|---------|-----------------------------------------------------|
| `And`   | `findByAuthorAndGenre(String author, String genre)` |
| `Or`    | `findByTitleOrAuthor(String title, String author)`  |

---

### 4️⃣ **Sorting**

| Keyword              | Example                         |
|----------------------|---------------------------------|
| `OrderBy<Field>Asc`  | `findByAuthorOrderByYearAsc()`  |
| `OrderBy<Field>Desc` | `findByAuthorOrderByYearDesc()` |

---

### 5️⃣ **Null / Not Null**

| Keyword     | Example                  |
|-------------|--------------------------|
| `IsNull`    | `findByGenreIsNull()`    |
| `IsNotNull` | `findByGenreIsNotNull()` |

---

### 6️⃣ **Check Existence**

| Keyword    | Example                       |
|------------|-------------------------------|
| `existsBy` | `existsByTitle(String title)` |

---

## 🧠 IMPORTANT RULES

### ✅ Field names must match Entity fields

Entity:

```java
private String author;
```

Method must match casing:

```java
findByAuthor() ✅
findByauthor() ❌
findByWriter() ❌ (no such field)
```

---

### ✅ You can chain keywords

```java
findByAuthorIgnoreCaseAndGenreIgnoreCase(String a, String g)
```

---

### ✅ Use `List<...>` or Optional return types

```java
List<Book> findByGenre(String genre);
Optional<Book> findByTitle(String title);
```

---

## 🚫 When JPA WON’T Implement the Method

❌ Wrong field name
❌ Wrong keyword order
❌ Complex logic (use `@Query` instead)

Example (invalid):

```java
findBookBySomethingFancyThatDoesNotExist()
```

---

## 🧩 If method name gets too big — use `@Query`

```java
@Query("SELECT b FROM Book b WHERE LOWER(b.author) LIKE LOWER(CONCAT('%', :author, '%'))")
List<Book> searchAuthor(@Param("author") String author);
```

---

## 🎬 Example Repository

```java
public interface BookRepository extends JpaRepository<Book, Long> {

    List<Book> findByAuthorContainingIgnoreCase(String author);
    List<Book> findByGenreIgnoreCase(String genre);

    List<Book> findByYearGreaterThan(int year);
    List<Book> findByYearBetween(int start, int end);

    boolean existsByTitle(String title);

    List<Book> findAllByOrderByYearAsc();
    List<Book> findAllByOrderByYearDesc();
}
```

---

## 🏁 Summary

| Feature                          | Power |
|----------------------------------|-------|
| No SQL needed                    | ✅     |
| Query from method name           | ✅     |
| Very fast for simple logic       | ✅     |
| Use `@Query` for complex queries | ✅     |

---
