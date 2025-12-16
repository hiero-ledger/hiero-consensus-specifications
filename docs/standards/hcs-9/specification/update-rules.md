---
title: update-rules module
sidebar_position: 8
---


#### [ hcs-9 - Poll Metadata Schema ]

The update-rules module defines the rules around the update action. The [update action](../../hcs-8/operations.md) is defined under hcs-8.

```
{
    "schema" - the schema that defines the structure of this module
    "permissions" - array of permissions-module which defines who is allowed to use the update action
    "rules" - an array of rules that govern the update action
}
```

![update-rules diagram](../../../../static/polls/update-rules.png)

The JSON Schema file can be found on here: [update-rules.json](/assets/schema/update-rules.json)

## Fields

### schema

**Tags:** required, case-insensitive

` "schema" : { "type" : "string" }`

The *schema* field defines the schema that is being used. It is any identifiable string about the standard or platform that the data follows. A developer who wishes to implement the schema will use the schema to filter data that they support and process it appropriately.

For the hcs-9 standard, the schema is defined as hcs-9.

### permissions

**Tags:** optional, module

Permissions is an array of modules that define who is allowed to perform the update action.

Permissions are used for this purpose for every action. This specification implements a number of default permission modules. This is covered in more detail on the [Permissions](./permissions.md) page.

**Default Behaviour**

If permissions is not defined, no user is allowed to manage.

### rules

**Tags:** optional

Rules is an array of [rules](./rules.md) that define additional behaviours. 

####WRITING IN PROGRESS

For hcs-9 the update rules define the ability of the creator (or permitted users) to modify the poll details after registration.

For example, updateTitle , updateDescription, updateOption.
