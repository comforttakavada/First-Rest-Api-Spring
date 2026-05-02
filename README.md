# First REST API – Spring Boot Product Management

A fully functional REST API backend built with Java and Spring Boot.  
It supports full CRUD operations, Swagger UI documentation, global exception handling, and an H2 in-memory database connected via Spring Data JPA.

---

## Technologies Used

- Java 17
- Spring Boot
- Spring Web (REST controllers)
- Spring Data JPA (database access)
- Hibernate (ORM framework)
- H2 Database (in-memory, for development)
- Springdoc OpenAPI / Swagger UI
- Maven

---

## How to Run

1. Clone or download the project
2. Open in IntelliJ IDEA
3. Run Maven reload (right-click project → Maven → Reload Project)
4. Run `FirstRestApiSpringApplication.java`
5. Application starts at: `http://localhost:8080`

---

## Project Structure

```
src/main/java/pl/edu/vistula/firstrestapispring/
│
├── product/
│   ├── api/
│   │   ├── request/
│   │   │   ├── ProductRequest.java          # Body for POST requests
│   │   │   └── UpdateProductRequest.java    # Body for PUT requests
│   │   ├── response/
│   │   │   └── ProductResponse.java         # Response sent back to client
│   │   └── ProductController.java           # Handles all HTTP requests
│   ├── domain/
│   │   └── Product.java                     # JPA entity (maps to DB table)
│   ├── repository/
│   │   └── ProductRepository.java           # Extends JpaRepository
│   ├── service/
│   │   └── ProductService.java              # Business logic
│   └── support/
│       ├── exception/
│       │   └── ProductNotFoundException.java
│       ├── ProductExceptionAdvisor.java      # Global error handler
│       ├── ProductExceptionSupplier.java     # Supplies typed exceptions
│       └── ProductMapper.java               # Maps objects between layers
│
├── shared/
│   └── api/
│       └── response/
│           └── ErrorMessageResponse.java    # Wraps error messages
│
└── FirstRestApiSpringApplication.java        # Main entry point
```

---

## application.properties

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/console/
spring.datasource.url=jdbc:h2:mem:testdb
logging.level.org.hibernate.SQL=DEBUG
```

---

## API Endpoints

| Method | URL | Description | Response Status |
|--------|-----|-------------|-----------------|
| POST | `/api/v1/products` | Create a new product | 201 Created |
| GET | `/api/v1/products` | Get all products | 200 OK |
| GET | `/api/v1/products/{id}` | Get product by ID | 200 OK |
| PUT | `/api/v1/products/{id}` | Update a product | 200 OK |
| DELETE | `/api/v1/products/{id}` | Delete a product | 204 No Content |

---

## Endpoint Details

### POST /api/v1/products — Create a Product

Accepts a JSON body and saves a new product to the database.

**Request body:**
```json
{
  "name": "First product"
}
```

**Response — 201 Created:**
```json
{
  "id": 1,
  "name": "First product"
}
```
<img width="779" height="435" alt="image" src="https://github.com/user-attachments/assets/279ab6f0-9b40-4c4d-abf2-fc8bcec8ad8d" />


**What happens in the code:**
1. The request JSON is mapped to a `ProductRequest` object via `@RequestBody`
2. The controller calls `ProductService.create()`
3. The service uses `ProductMapper.toProduct()` to convert it to a `Product` entity
4. The entity is saved via `ProductRepository.save()`
5. The saved entity is mapped to `ProductResponse` and returned with status `201 Created`

---

### GET /api/v1/products/{id} — Get Product by ID

Fetches a single product using its ID from the URL path.

**Request:** `GET http://localhost:8080/api/v1/products/1`

**Response — 200 OK:**
```json
{
  "id": 1,
  "name": "First product"
}
```

**What happens in the code:**
1. `@PathVariable` extracts the `id` from the URL
2. The controller calls `ProductService.find(id)`
3. The service calls `ProductRepository.findById(id)`
4. If found, the product is mapped to `ProductResponse` and returned
5. If not found, `ProductNotFoundException` is thrown (see Error Handling)

---

### GET /api/v1/products — Get All Products

Returns a list of all products in the database.

**Request:** `GET http://localhost:8080/api/v1/products`

**Response — 200 OK:**
```json
[
  { "id": 1, "name": "First product" },
  { "id": 2, "name": "Second product" }
]
```

If no products exist, returns an empty array `[]` — no exception is thrown.

**What happens in the code:**
1. `ProductRepository.findAll()` returns all records
2. The list is streamed and each `Product` is mapped to `ProductResponse`
3. The result list is returned with status `200 OK`

---

### PUT /api/v1/products/{id} — Update a Product

Updates the name of an existing product.

**Request:** `PUT http://localhost:8080/api/v1/products/1`

**Request body:**
```json
{
  "name": "Updated product name"
}
```

**Response — 200 OK:**
```json
{
  "id": 1,
  "name": "Updated product name"
}
```

**What happens in the code:**
1. The `id` is extracted from the URL via `@PathVariable`
2. The body is mapped to `UpdateProductRequest` via `@RequestBody`
3. The service finds the existing product or throws `ProductNotFoundException`
4. `ProductMapper.toProduct(product, updateRequest)` updates the entity fields
5. The updated entity is saved and returned as `ProductResponse`

---

### DELETE /api/v1/products/{id} — Delete a Product

Deletes a product by ID. Returns no body on success.

**Request:** `DELETE http://localhost:8080/api/v1/products/1`

**Response — 204 No Content** (empty body)

**What happens in the code:**
1. The service first checks if the product exists (throws exception if not)
2. `ProductRepository.deleteById()` removes the record
3. The controller returns `ResponseEntity.status(HttpStatus.NO_CONTENT).build()`

---

## Error Handling

If a product with the given ID does not exist, the API returns a `404` response instead of a generic `500` error.

**Example:** `GET http://localhost:8080/api/v1/products/999`

**Response — 404 Not Found:**
```json
{
  "message": "Product with 999 not found"
}
```

**How it works:**
- `ProductNotFoundException` extends `RuntimeException` and formats the message
- `ProductExceptionSupplier` provides a reusable supplier of this exception
- `ProductExceptionAdvisor` is annotated with `@ControllerAdvice` and catches all `ProductNotFoundException` errors, returning an `ErrorMessageResponse` with status `404`

---

## Swagger UI

Swagger UI is available while the application is running.

**URL:** `http://localhost:8080/swagger-ui/index.html`

From Swagger UI you can:
- See all endpoints with their descriptions
- Send requests and view responses directly in the browser
- Inspect `ProductRequest` and `ProductResponse` schemas

**Raw OpenAPI JSON:** `http://localhost:8080/v3/api-docs`

To enable Swagger, add this dependency to `pom.xml`:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.0</version>
</dependency>
```

---

## H2 Database Console

View the in-memory database directly in the browser.

**URL:** `http://localhost:8080/console`

**Login details:**

| Field | Value |
|-------|-------|
| JDBC URL | `jdbc:h2:mem:testdb` |
| User Name | `sa` |
| Password | *(leave empty)* |

**Useful SQL queries:**

```sql
-- View all products
SELECT * FROM PRODUCTS;

-- Find product by ID
SELECT * FROM PRODUCTS WHERE ID = 1;

-- Count all products
SELECT COUNT(*) FROM PRODUCTS;

-- Delete a product
DELETE FROM PRODUCTS WHERE ID = 1;
```

> The H2 database is in-memory only. All data is lost when the application stops. This is intentional for development and testing purposes.

---

## Database Entity

The `Product` class is a JPA entity mapped to the `products` table in H2.

```java
@Entity(name = "products")
public class Product {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
}
```

Hibernate automatically creates the `products` table on startup with columns `ID` (BIGINT, primary key) and `NAME` (VARCHAR).

---
<img width="784" height="405" alt="image" src="https://github.com/user-attachments/assets/daca9aaf-60ce-433b-9a78-8839f055c592" />



## Key Spring Annotations Used

| Annotation | Used In | Purpose |
|---|---|---|
| `@RestController` | ProductController | Marks class as REST controller |
| `@RequestMapping` | ProductController | Sets base URL `/api/v1/products` |
| `@PostMapping` | create() | Handles POST requests |
| `@GetMapping` | find(), findAll() | Handles GET requests |
| `@PutMapping` | update() | Handles PUT requests |
| `@DeleteMapping` | delete() | Handles DELETE requests |
| `@RequestBody` | method params | Maps JSON body to Java object |
| `@PathVariable` | method params | Extracts `{id}` from URL |
| `@Service` | ProductService | Marks as Spring service bean |
| `@Repository` | ProductRepository | Marks as Spring data repository |
| `@Component` | ProductMapper | Marks as Spring-managed component |
| `@ControllerAdvice` | ProductExceptionAdvisor | Global exception handler |
| `@ExceptionHandler` | advisor methods | Catches specific exception types |
| `@Entity` | Product | Maps class to database table |
| `@Id` | Product.id | Marks primary key |
| `@GeneratedValue` | Product.id | Auto-generates primary key value |
