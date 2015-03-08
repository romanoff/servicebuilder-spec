ServiceBuilder
==============

ServiceBuilder is intended to help developers create common web applications using DSL that will help boost their productivity. It's not intended to cover all possible usecases, but hopefully will cover quite a few.

Models
------

Here is how you define a model in service builder:

```
<Model name (singular)> {
}
```

Example:

```
User {
}
```

### Fields

To store data in model we will need fields. Here are field types that can be represented in service builder:

|Field type| Description               |
|----------|---------------------------|
| int      | integer field             |
| double   | double field              |
| string   | text field                |
| date     | date field                |
| datetime | field with date and time  |
| file     | field that contains file  |
| image    | field that contains image |

Here is how we define a model with couple of fields:

```
<Model name (singular)> {
  fields {
    <field name>: <field type>
    <field name>: <field type>
    ...
  }
}
```

Example:


```
User {
  fields {
    name: string
    email: string
    birth_date: date
  }
}
```
Couple of fields will be automatically added to your model. Such fields as id (unique identifier), created (datetime) and updated (datetime). This fields will be populated automatically for you whenever record is created/updated.

### Pagination

Pagination section is normally used to set per_page and max_per_page for pagination (specifically for index action). Here is syntax:

```
<Model name (singular)> {
  pagination {
    per_page: <int>
    max_per_page: <int>
  }
}
```

Both per_page and max_per_page are optional. If not specified, defaults will be used. (by default per_page=10, max_per_page=50)

Example:

```
User {
  fields {
    name: string
  }
  pagination {
    per_page: 20
    max_per_page: 100
  }
}
```

### Json

Model record has json representation. By default json will return all model fields. But sometimes that's wasteful. In this case you can customize json representations that model has. To do that, you just need to specify what fields will each json representation type have:

```
<Model name (singular)> {
  json {
    <json representation type>: [<field name>, <field name> ...]
    <json representation type>: [<field name>, <field name> ...]
    ...
  }
}
```

Example:

```
User {
  fields {
    name: string
    email: string
    birth_date: date
  }
  json {
    default: [id, name]
    full: [id, name, email, birth_date, created, updated]
  }
}
```

Latest example shows that you can also specify id, created and updated fields in json representation type. Also no matter how you named first json represenatation, it will be considered default one.

####Files and images

You can also return file and image fields in json representation. By default it will give you file url and image url. For images specifically you can customize image size and it will be reized accordingly.

Example of using image:

```
User {
  fields {
    name: string
    email: string
    logo: image
  }
  json {
    default: [id, name, logo#60x60]
    full: [id, name, email, logo, created, updated]
  }
}
```

### Associations

Associations between models can be defined in association block. There are four types of associations:

| Association type        |
|-------------------------|
| has_one                 |
| belongs_to              |
| has_many                |
| has_and_belongs_to_many |

If you specified association on a model, you don't have to worry about foreign keys, they will be automatically created for you.

Here is how you define associations in service builder:

```
<Model name (singular)> {
  associations {
    <assocaition type> <model name (lowercase underscore)>
    <association type> <model name (lowercase underscore)>
   }
  }
```

Model name in association block also depends on association type. If that's `has_one` or `belongs_to`, you have to use singular form. If that's `has_many` or `has_and_belongs_to_many`, you have to use plural. You can also create association with custom name and specify associatied model name manually. Here is how it can be done:

```
<Model name (singular)> {
  associations {
    <assocaition type> <custom name>(<Model name (singular)>)
    <association type> <custom name>(<Model name (singular)>)
   }
  }
```

Example:

```
User {
  fields {
    name: string  
  }
  association {
    has_one address
    has_many email_addresses(Email)
  }
}

Address {
  fields {
    country: string
    city: string
    street: string
  }
  associations {
    belongs_to user
  }
}

Email {
  fields {
    address: string
  }
  associations {
    belongs_to user
  }
}
```

### Json and associations

If you want, you can add association json to your model json representation. 

Here is how you would do that:n

```
<Model name (singular)> {
  json {
    <json representation type>: [<field name>|<association_name>[#<json type>] ...]
    <json representation type>: [<field name>|<association_name>[#<json type>] ..].
    ...
  }
}
```

Example:

```
User {
  fields {
    name: string  
  }
  json {
    default: [id, name, address#full]
  }
  association {
    has_one address
  }
}

Address {
  fields {
    country: string
    city: string
    street: string
  }
  associations {
    belongs_to user
  }
  json {
    default: [id, street]
    full: [id, country, city, street]
  }
}
```
### Validations

Validations for fields can be added to the model. Here are possible validation types:

|Validation| Description                                         | Parameters      |
|----------|-----------------------------------------------------|-----------------|
| present  | validates field content presence                    | none            |
| min      | validates minimum (details dependant on field type) | int             |
| max      | validates minimum (details dependant on field type) | int             |
| max_size | validates maximum file size                         | size (20Mb)     |
| one_of   | validates that field value is one of enum values    | enum            |
| uniq     | validates that field is uniq                        | optional scopes |

Here is how validations are defined in service builder:

```
<Model name (singular)> {
  validations {
    <field name>: [<validation>, <validation>, ...]
    <field name>: [<validation>, <validation>, ...]
  }
}
```

Example:

```
Document {
 enum Type {
    Work
    Business
    Travel
  }
  fields {
    name: string
    description: string
    document: file
    type: int
  }
  validations {
    name: [max(20), uniq(type)]
    description: [min(10])
    document: [present, max_size(20Mb)]
    type: [one_of(Document::Type)]
  }
}
```
