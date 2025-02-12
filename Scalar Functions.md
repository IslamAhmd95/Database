#scalar_functions #COALESCE #concat #length #upper #lower #round #floor #ceil #abs #now #date_add #date_format #user #database #if #case #COALESCE #ifnull

---

**Scalar functions** in SQL are functions that operate on a single value and return a single value. They are used to perform operations like formatting, calculations, or transformations on individual values in a query.

#### Common Categories of Scalar Functions

- **String Functions**: such as `CONCAT()`, `LENGTH()`, `UPPER()`, `LOWER()`

```sql
SELECT CONCAT('Hello', ' ', 'World') AS Greeting;
```

- **Numeric Functions**: such as `ROUND()`, `FLOOR()`, `CEIL()`, `ABS()`
  
```sql
SELECT ROUND(12.345, 2) AS RoundedValue;
```

- **Date and Time Functions**: such as `NOW()`, `DATE_ADD()`, `DATE_FORMAT()`
  
```sql
SELECT NOW() AS CurrentTime, DATE_FORMAT(NOW(), '%Y-%m-%d') AS FormattedDate;
%% FormattedDate = current date without time %%
```

- **System Functions**: such as `USER()`, `DATABASE()`
  
```sql
SELECT DATABASE() AS CurrentDatabase;
```

- **Control Flow Functions**: such as `IF()`, `CASE`, `COALESCE()`
  
  
#### `COALESCE()` Function
The `COALESCE()` function is a **control flow function** used to handle `NULL` values by returning the first non-NULL value in a list of arguments.

```sql
Syntax: COALESCE(value1, value2, ..., valueN)
- **Returns**: The first non-NULL value from the arguments provided.
- If all arguments are `NULL`, it returns `NULL`.
```

```sql
SELECT coalesce(null, "default value");
%% `Result` = `Default Value` (because the first argument is `NULL`, so it uses the second argument). %%
```

```sql
SELECT 
    id, 
    COALESCE(middle_name, 'N/A') AS MiddleName
FROM students;
%% This replaces `NULL` values in the `middle_name` column with `"N/A"`. %%
```

```sql
SELECT 
    COALESCE(phone_number, mobile_number, 'No Contact') AS Contact
FROM employees;
%% This checks multiple columns (`phone_number` and `mobile_number`) and uses the first non-NULL value. If both are `NULL`, it defaults to `"No Contact"`. %%
```

##### Why Use `COALESCE()` Instead of `IFNULL()`?
- **`COALESCE()`** can take **multiple arguments**, returning the first non-NULL value.
- **`IFNULL()`** works with only **two arguments**.
  
```sql
SELECT 
    COALESCE(NULL, NULL, 'Fallback Value') AS Result;  -- Works
SELECT 
    IFNULL(NULL, 'Fallback Value') AS Result;  -- Works
SELECT 
    IFNULL(NULL, NULL) AS Result;  -- Fails to provide a fallback beyond two arguments, means can't use more that these 2 arguments
```

- **`COALESCE()`** is ANSI SQL compliant, making it more portable across databases compared to `IFNULL()`, which is specific to MySQL.
- Can be used in conjunction with other functions, like `CONCAT()` or `CASE`.

```sql
SELECT 
    CONCAT('Contact: ', COALESCE(phone, 'No phone')) AS ContactDetails 
FROM employees;
```
