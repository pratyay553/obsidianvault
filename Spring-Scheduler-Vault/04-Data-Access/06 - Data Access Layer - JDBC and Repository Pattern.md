---
tags: [data-access, jdbc, repository-pattern, interview]
---

# Data Access Layer — JDBC & Repository Pattern

Related: [[18 - Database and Transactions]] · [[02 - Configuration Classes Overview|03 - Configuration Classes Overview]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
The **Repository Pattern** hides *how* data is fetched/stored behind a simple interface, so the rest of the app (schedulers, services) doesn't care whether data comes from JDBC, JPA, or a REST call.

## 🔑 Core Pieces

| Component | Role |
|---|---|
| `DataSource` | Manages actual DB connections (usually pooled, e.g., HikariCP) |
| `JdbcTemplate` | Spring's helper that removes JDBC boilerplate (opening/closing connections, statements, result sets) |
| `RowMapper<T>` | Converts one SQL result row into a Java object |
| `Repository` interface | The *contract* — "what can be done" (e.g., `findById`, `save`) |
| `RepositoryImpl` | The *implementation* — the actual SQL/JdbcTemplate code |
| `@Repository` | Marks the implementation as a Spring-managed bean; also enables automatic translation of SQL exceptions into Spring's `DataAccessException` hierarchy |

## 🛠️ Code Example

```java
public interface BirthdayDBOperation {
    List<Employee> findEmployeesWithBirthdayToday();
}

@Repository
public class BirthdayDBOperationImpl implements BirthdayDBOperation {

    private final JdbcTemplate jdbcTemplate;

    public BirthdayDBOperationImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public List<Employee> findEmployeesWithBirthdayToday() {
        String sql = "SELECT id, name, email FROM employee WHERE MONTH(dob)=MONTH(CURRENT_DATE) AND DAY(dob)=DAY(CURRENT_DATE)";
        return jdbcTemplate.query(sql, new EmployeeRowMapper());
    }
}

class EmployeeRowMapper implements RowMapper<Employee> {
    @Override
    public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Employee(rs.getLong("id"), rs.getString("name"), rs.getString("email"));
    }
}
```

## 🏭 Why a Generic `DBOperation` / `DBOperationImpl`?
Looking at the module's classes — `DBOperation`/`DBOperationImpl` (generic) plus `BirthdayDBOperation`, `TSLDBOperation`, `FleetedgeDBOperation`, etc. — this is a common pattern: a **generic base** with common CRUD-ish helpers, and **feature-specific repositories** that extend or compose it for their own queries. This avoids re-implementing basic query execution/error handling in every specific repository.

```java
public interface DBOperation<T> {
    List<T> executeQuery(String sql, RowMapper<T> mapper, Object... params);
    int executeUpdate(String sql, Object... params);
}
```

## ❓ Interview Q&A

**Q: Why use `JdbcTemplate` instead of raw JDBC?**
A: Raw JDBC requires manually opening/closing `Connection`, `Statement`, `ResultSet` in try/finally blocks and translating `SQLException`. `JdbcTemplate` handles all resource management and converts checked `SQLException` into Spring's unchecked `DataAccessException` hierarchy, making error handling much simpler.

**Q: `JdbcTemplate` vs Spring Data JPA — when would you pick JDBC here?**
A: `JdbcTemplate` gives full control over SQL — useful for complex/legacy queries, reporting, or when working with an existing schema that doesn't map cleanly to JPA entities. It's also lighter weight for a small scheduler module that doesn't need a full ORM.

**Q: What does `@Repository` do beyond marking a bean?**
A: It enables **exception translation** — Spring wraps the underlying persistence exception (SQL, Hibernate, etc.) into a consistent, unchecked `DataAccessException` subtype, so calling code doesn't need to catch vendor-specific exceptions.

**Q: What is `RowMapper` for, and how is it different from `ResultSetExtractor`?**
A: `RowMapper` maps **one row at a time** into an object (called once per row). `ResultSetExtractor` gives you the entire `ResultSet` at once, useful when you need to build one object out of multiple rows (e.g., master/detail aggregation).
