---
published: true
title: Preventing Empty Updates In PostgreSQL Tables Using A Simple Trigger Function
description: 'We will explore the purpose, mechanics, and advantages of a PostgreSQL trigger designed to safeguard against null or empty updates.'
tags:
  - sql
  - tutorial
  - programming
cover_image: 'https://upload.wikimedia.org/wikipedia/commons/2/29/Postgresql_elephant.svg'
canonical_url: null
id: 1646381
date: '2023-10-26T14:51:50Z'
---

### The Need for Preventing Empty Updates

Empty updates are silent problems in PostgreSQL, occurring when an `UPDATE` makes no real changes to data. They're wasteful and can lead to loops in scenarios with recursive triggers.

Some common causes of empty updates include:

- Triggers meant to maintain data integrity can trigger loops if they update the same table. This leads to inefficiencies.
- Two tables need to stay in sync. Changes in one table trigger updates in the other, and vice versa. Mismanagement can lead to recursive circular updates, causing empty updates and performance issues.

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
