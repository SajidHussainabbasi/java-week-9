# 📘 Spring Data Pagination and Sorting

Spring Data JPA provides built-in support for **pagination** and **sorting**, allowing you to efficiently query and organize large datasets.

This document explains the concepts behind:

- `Page<T>`
- `Pageable`
- `PageRequest`
- `Sort`

using the following example method:

```java
public Page<Product> getProducts(Integer page, Integer limit, String category, String sortBy)
{
    Pageable pageable;

    if (sortBy != null && !sortBy.isEmpty())
    {
        pageable = PageRequest.of(page, limit, Sort.by(sortBy).ascending());
    }
    else
    {
        pageable = PageRequest.of(page, limit);
    }

    if (category != null && !category.isEmpty())
    {
        return repo.findByCategoryIgnoreCase(category, pageable);
    }
    else
    {
        return repo.findAll(pageable);
    }
}
```

---

## 🧩 1. Overview of the Method

This method fetches a **paginated and optionally sorted list of `Product` entities**, optionally filtered by a given category.

It demonstrates:

- How to **create pageable queries**.
- How to **conditionally apply sorting**.
- How to **return a Page object** containing both data and pagination metadata.

---

## ⚙️ 2. Components Involved

### **1. `Page<T>`**

#### **What `Page<T>` is**

- A Spring Data container for **a single page of results**.
- Includes both the **content** (list of entities) and **metadata** (total pages, total elements, etc.).

#### **In this method**

The return type `Page<Product>` gives:

- The list of products (`getContent()`).
- Information about pagination (`getNumber()`, `getTotalPages()`, etc.).

#### **Example Usage**

```java
Page<Product> products = productService.getProducts(0, 10, "Electronics", "price");

System.out.println("Total Pages: " + products.getTotalPages());
System.out.println("Total Elements: " + products.getTotalElements());
System.out.println("Products on this page: " + products.getContent());
```

---

### **2. `Pageable`**

#### **What `Pageable` is**

An **interface** that defines the pagination and sorting configuration.
It tells the repository:

- Which page number to fetch
- How many records to fetch per page
- How to sort the results

#### **Usage in this method**

`Pageable pageable` is dynamically created based on user input:

```java
if (sortBy != null && !sortBy.isEmpty()) {
    pageable = PageRequest.of(page, limit, Sort.by(sortBy).ascending());
} else {
    pageable = PageRequest.of(page, limit);
}
```

This means:

- If a `sortBy` field is provided → sort ascending by that field.
- Otherwise → fetch data unsorted.

---

### **3. `PageRequest`**

#### **What `PageRequest` is**

A **concrete implementation** of the `Pageable` interface.
Used to specify:

- Page index (zero-based)
- Page size (number of records per page)
- Sort order (optional)

#### **Example in this method**

`PageRequest.of(page, limit, Sort.by(sortBy).ascending())`
creates a pageable object with both pagination and sorting.

For example:

```java
PageRequest.of(0, 10, Sort.by("price").ascending());
```

➡ means: fetch **page 0**, **10 items per page**, **sorted by price ascending**.

---

### **4. `Sort`**

#### **What `Sort` is**

Defines **how the data should be ordered** — ascending or descending, by one or more fields.

#### **Applied in this method**

```java
Sort.by(sortBy).ascending()
```

tells Spring Data to sort the products by the field name given in the `sortBy` parameter.

You could also extend it:

```java
Sort.by(Sort.Order.desc("price"), Sort.Order.asc("name"));
```

---

## 🧠 3. How It All Works Together

Here’s how the method operates step by step:

1. **Receive client input**
   The user specifies page, limit, category, and optionally a sort field.

2. **Build a Pageable object**

   - If `sortBy` is provided → create a pageable with sorting.
   - If not → create a pageable with just pagination.

3. **Run the repository query**

   - If `category` is provided → call `findByCategoryIgnoreCase(category, pageable)`.
   - Otherwise → call `findAll(pageable)`.

4. **Return a `Page<Product>`**
   The repository automatically handles pagination and sorting at the database level.

---

## 🧾 4. Example Scenarios

### **Example 1 — Paginated only**

```java
getProducts(0, 5, null, null);
```

➡ Returns page 0 (first 5 products), no sorting, all categories.

---

### **Example 2 — Paginated + Sorted**

```java
getProducts(0, 5, null, "price");
```

➡ Returns first 5 products sorted by price ascending.

---

### **Example 3 — Paginated + Filtered + Sorted**

```java
getProducts(1, 10, "Electronics", "name");
```

➡ Returns page 1 (10 products per page) from the "Electronics" category, sorted by name.

---

## 📊 5. Example Output (for illustration)

| Property       | Value |
| -------------- | ----- |
| Page Number    | 1     |
| Page Size      | 10    |
| Total Pages    | 3     |
| Total Elements | 25    |
| Has Next       | true  |
| Has Previous   | true  |

---

## 🧾 6. Repository Integration Example

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    Page<Product> findByCategoryIgnoreCase(String category, Pageable pageable);
}
```

Spring Data automatically interprets the method name and applies pagination + sorting according to the `Pageable` parameter.

---

## ✅ 7. Summary

| Concept                 | Class         | Purpose                                              |
| ----------------------- | ------------- | ---------------------------------------------------- |
| **Page**                | `Page<T>`     | Represents a single page of data + metadata          |
| **Pagination Info**     | `Pageable`    | Interface defining page and size configuration       |
| **Page Implementation** | `PageRequest` | Concrete `Pageable` implementation with sort options |
| **Sorting**             | `Sort`        | Defines field(s) and order for sorting results       |

---

## 🏁 Conclusion

This example demonstrates how to:

- Dynamically create pageable and sortable queries
- Filter data conditionally
- Return results in a structured, paginated `Page<T>` format

These abstractions make Spring Data JPA highly flexible for building REST APIs that handle **large datasets efficiently**.

---
