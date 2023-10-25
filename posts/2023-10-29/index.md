---
published: false
title: Preventing Empty Updates In PostgreSQL Tables Using A Simple Trigger Function
description: We will explore the purpose, mechanics, and advantages of a PostgreSQL trigger designed to safeguard against null or empty updates.
tags:
  - sql
  - tutorial
  - programming
cover_image: 'https://upload.wikimedia.org/wikipedia/commons/2/29/Postgresql_elephant.svg'
canonical_url: null
---

### The Need for Preventing Empty Updates

Empty updates occur when an `UPDATE` statement is executed, but the new values being set are identical to the old values. In other words, the update does not result in any changes to the data. These empty updates can waste resources and potentially lead to infinite loops in cases where table maintenance triggers recursively update their own table or multiple tables update each other in a circular fashion.

Empty updates can occur in a variety of situations, but they are often most pronounced in scenarios involving recursive triggers. Consider two common cases where empty updates can be problematic:

**Table Maintenance Triggers**: In many databases, you'll find triggers responsible for maintaining a table's data integrity. If these triggers inadvertently update the same table they are meant to maintain, you could inadvertently trigger a recursive loop. Without a mechanism to detect and halt empty updates, these loops could lead to database inefficiency and even stack overflows.

**Two-Way Data Synchronization**: In complex database environments, you may have two tables that need to stay synchronized. Changes in one table trigger updates in the other, and vice versa. Without careful handling, this can lead to recursive circular updates, causing empty updates and performance issues.

### The Solution: A Simple Trigger Function

```sql
CREATE OR REPLACE FUNCTION prevent_empty_update()
RETURNS trigger
LANGUAGE plpgsql
AS $function$
DECLARE
BEGIN
    if old is not distinct from new then
        return null;
    end if;

    return new;
END;
$function$;
```

- The `if old is not distinct from new then` condition evaluates if the old and new data are the same. If they are identical, it means that the update does not change any values in the row, and the function returns `null`.
- If the condition is not met, indicating that the update will make changes to the row, the function returns `new`, allowing the update to proceed.

### Application Of The Trigger Function

To put this trigger function into action, you need to apply it to the specific tables where you want to prevent empty updates.
Use the `BEFORE UPDATE` trigger to execute the `prevent_empty_update()` function for each row before the update is performed.

```sql
CREATE TRIGGER prevent_empty_updates
BEFORE UPDATE ON your_table
FOR EACH ROW
EXECUTE FUNCTION prevent_empty_update();
```

By implementing this trigger function, you can improve the efficiency and reliability of your PostgreSQL database, reducing the risk of unnecessary data modifications and recursive loops.
