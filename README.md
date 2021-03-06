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
| bool     | boolean field             |
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
### Internationalization

Internationalization can be part of your application. It's specifically intended to translate models data.

Here is how you would specify i18n for model in service builder:

```
<Model name (singular)> {
  i18n {
    default: "<lang>"
    languages: ["<lang>", "<lang>", ...]
    fields: ["<field name>", "<field name>", ...]
    mode: <missing|default>
  }
}
```

In `default` field you need to specify default language. Then in `languages`, specify rest of supported languages. `fields` should contain all the fields that are supposed to be translated to other languages. `mode` has two possible options - `missing` (record is considered missing in language that has no translation) or `default` (if translation is missing for the record, use default language).

### Slugs

You might require slugs for your model records. Here is how you would create them in service builder:

```
<Model name (singular)> {
  slug {
    fields: [<filed name>, <field name>, ...]
  }
}
```

`fields` specify an array of fields that will be used to generate record slug.

### Authentication

Authentication mechanism can be added to selected model. Authentication will happen with login_name and password. Login name can be configured to be one of the specified fields (let's say user can use login or email field as login_name).

Here is what the syntax looks like:

```
<Model name (singular)> {
  authentication {
    login_name: [<field name>, <field name>, ...]
  }
}
```

Example:

```
User {
  authentication {
    login_name: [email]
  }
}
```

In specified example user will have to log in using email/password.

Authentication adds `password_digest` string field to the model as well as login name string fields (unless they have been already been specified in `fields` section).

### Custom actions

By default each model gets 5 REST actions: `index`, `show`, `create`, `update`, `destroy`. This actions can be customized by selecting only the ones that will be activated:

```
<Model name (singular)> {
  actions {
    rest_actions: [<action name>, <action name>...]
  }
}
```

Example:

```
User {
  fields {
    name: string
    email: string
  }
  actions {
    rest_actions: [index, show]
  }
}
```

This example will have only `index` and `show` actions for user.

Also custom actions for model can be created.

```
<Model name (singular)> {
  actions {
    <action name> {
      <code>
    }
  }
}
```

Custom action should contain service builder code. There should be separate section on code. Action can return json, html or xml. In this case content will be rendered "as is". Or it can return data that eventually can be acted on (application will determine what format has user requested and render it appropriately).

### Scopes

Scopes allow you to write more specific queries that retrieve only data you want. Scopes can be used in custom actions as well as for permissions restriction. Here is scope syntax:

```
  <Model name (singular)> {
    scopes {
      <scope name>[(param1, param2, ...)] {
        <query clause>[.<query clause>...]
      }
    }
  }
```

Example:

```
User {
  fields {
    height: int
    admin: bool
  }
  scopes {
    admin { where('admin = ?', true) }
    taller_than(height) {
      where('height > ?', height)
    }
  }
}
```

Here are possible query clauses:

| Query cause |
|-------------|
| where       |
| limit       |
| offset      |
| order       |


### Permissions

Authenticated users have roles. For this roles certain actions might be allowed or not. Or action to resource can be restricted based on current user and query parameters.

Synatx:

```
  <Model name (singular)> {
    permissions {
      <authenticated model name (downcase singular)>{
        [action|[action1, action2, ...]] {
          <role>: [true|false|<scope>],
          <role>: [true|false|<scope>],
          ...
        }
      }
    }
  }
```

Example:

```
Article {
  fields {
    creator_id: int
  }
  scopes {
    for_user(user) {
      where('creator_id = ?', user.id)
    }
  }
  permissions {
    user {
      [update, destroy] {
        admin { true }
        editor {
          for_user(current_user.id)
        }
      }
      show {
        for_user(current_user.id)
      }
    }
  }
} 
```

In this example user is allowed to update and destroy article if user is an admin. If user is editor, he can ony update and destroy article if he is creator. Also show action only works for article creators.

### Routes

Custom routes can be set on model. By default model will have 5 REST actions: `index`, `show`, `create`, `update`, `destroy`. If actions list has been customized, then routes only to selected ones will be present (this can be made in `rest_actions` parameter under `actions`).

Syntax:

```
<Model name (singular)> {
  routes {
    [default: <preserve|override>]
    <method> '<path>', <action|[action1, action2, ...]>[, <format>: '<format handler>']
  }
}
```

Example:

```
Album {
  routes {
    default: override
    get '/great_albums/:id', show, html: 'album/show', json: 'customize_album'
    put '/great_albums/:id', update
  }
}
```

In this example couple of actions will be changed. Instead of using '/albums/:id', `show` and `update` actions will use '/great_albums/:id'. Also show got html as possible output format (what will happen is that all generated json parameters will be send to 'album/show' action of remote server responsible for rendering html and then recieved html will be returned back). Json output for album show action got additional filter. Filters are normally used to transform json into different representation. For example, with filter you can transform following json:

```
  {users: [{id: 1, name: 'John Doe'}, {id: 2, name: 'Bill Clinton'}]}
```

Into following json:

```
  {count: 2,  users: ['John Doe', 'Bill Clinton']}
```

Filters can be applied one on top of the other. In this case you would use `|` pipe to chain them together.

Example:

```
customize_album | simple_album_representation
```

Filters will be applied left to right.

In above example there is also default parameter. If default is not specified, specified routes will replace default REST routes that would be automatically created. `override` mode will override action routes that have been specify in routes section, but the ones that have not been mentioned, will be preserved. Preserve will preserve default REST routes and routes specified in routes section will just be appended.

Filters can also be applied to before json parameters will be send to remote server to render json.

Example:

```
Album {
  routes {
    default: override
    get '/great_albums/:id', show, html: 'album/show | customize_album'
    put '/great_albums/:id', update
  }
}
```

In this case `customize_album` filter will be applied to json before data will be send over to remote server to render html.

