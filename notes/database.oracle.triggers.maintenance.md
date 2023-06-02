---
id: f9h0y53ru4ys8q5reixec9a
title: Maintenance
desc: ''
updated: 1685446554072
created: 1685426913400
---
## Disabling A Trigger
Sometimes you might need to disable a trigger temporarily.

```sql
ALTER TRIGGER trigger_name DISABLE;
```

## Enablig A Trigger
Enabling a trigger is very similar to the process of disabling.

```sql
ALTER TRIGGER trigger_name ENABLE;
```

## Specify Schema When Alterting Triggers
You can add the name of the schema in the sql statement to disable a trigger that is not in your schema. **Or you can just always add it as mentioning own schema in anything won't give you error.**

```sql
ALTER TRIGGER schema_name.trigger_name DISABLE;
```


