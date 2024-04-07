---
published: true
title: Finding Which Columns Value Has Actually Changed in PostgreSQL Trigger Functions
description: 'In PostgreSQL, triggers are powerful tools used to execute custom actions when certain events occur on a specified table.'
tags:
  - sql
  - tutorial
  - programming
cover_image: 'https://i.imgur.com/JK2j6mT.png'
canonical_url: null
---

### Finding Which Columns Value Has Actually Changed in PostgreSQL Trigger Functions

In PostgreSQL, triggers are powerful tools used to execute custom actions when certain events occur on a specified table. One common use case for triggers is auditing changes to data. When implementing an audit trail, it's essential to identify which columns have been modified in an update operation. This article explores how to efficiently determine which columns have had their values changed within a trigger function.

### The Challenge

When an update operation occurs on a table, it's necessary to identify which columns have been modified and record these changes. However, PostgreSQL does not provide built-in functionality to directly identify the changed columns within a trigger function. Instead, developers need to devise a method to achieve this.

### Solution

To identify the changed columns within a PostgreSQL trigger function, we can compare the old and new values of each column using dynamic SQL and JSON functions. Let's illustrate this with an example trigger function:

```sql
CREATE OR REPLACE FUNCTION audit_changes()
RETURNS TRIGGER AS
$$
DECLARE
    _changed_columns TEXT[];
BEGIN
    SELECT array_agg(pre.key)
    INTO _changed_columns
    FROM jsonb_each(to_jsonb(old)) AS pre
    CROSS JOIN jsonb_each(to_jsonb(new)) AS post
    WHERE pre.key = post.key
      AND pre.value IS DISTINCT FROM post.value
    ;

    -- Perform actions based on the changed columns
    -- For example, log the changes to an audit table
    INSERT INTO audit_table (table_name, changed_columns)
    VALUES (TG_TABLE_NAME, _changed_columns);
    
    RETURN NEW;
END;
$$
LANGUAGE plpgsql;
```

#### Trigger Definition

Once the trigger function `audit_changes()` is defined, it can be associated with a specific table and event (e.g., `BEFORE UPDATE`):

sql
```sql
CREATE TRIGGER audit_table_changes
BEFORE UPDATE ON your_table
FOR EACH ROW
EXECUTE FUNCTION audit_changes();
```

This trigger will execute the `audit_changes()` function before each `UPDATE` operation on `your_table`. It will identify the changed columns, log them into an audit table, and then allow the original update operation to proceed.

### Conclusion

In PostgreSQL trigger functions, identifying changed columns is a common requirement for various purposes such as auditing and conditional logic. By leveraging PostgreSQL's JSONB functions, we can efficiently compare old and new row states to determine which columns have been modified. Implementing this logic within trigger functions allows for automated tracking of changes, enhancing data integrity and auditability in database systems.
