name: inverse
layout: true
class: center, middle, inverse

---
# Adding a GraphQL API to Django
.pink[With Vue.js Frontend]

Updated June 2019, PyCon Thailand 2019

---
layout: false
.left-column[
  ## About Me
]
.right-column[
### Businesses and Startups
- First online startup: 2008 (tailor-made shirts)
- iViveLabs agency since 2012
- Founder of [BorgBase.com](https://www.borgbase.com) – backup hosting platform
- Django & Vue.js mostly

### Open Source/Community Projects
- **invoice2data**.red[a] – a Python library for extracting structured data from PDF invoices, used in Odoo ERP
- **Vorta**.red[b] – a Borg Backup desktop client (lightning talk coming!!)
- **Group 868** – WhatsApp support group for startup founders, born from YC Startup School

.footnote[
  .red[a] [https://github.com/invoice-x/invoice2data](https://github.com/invoice-x/invoice2data)

  .red[b] [https://github.com/borgbase/vorta/](https://github.com/borgbase/vorta/)
  ]
]

---
.left-column[
  ## What to Expect
]
.right-column[
### Goals of this Talk
- Targetted at web developers already using Django
- Introduce GraphQL concepts
- See how it looks like in code (Python and JavaScript)

### Contents
1. Comparison with REST – Issues GraphQL solves
2. Schema Definition
3. Backend Example: Graphene & Django
4. Frontend Example: Vue.js & Graphql-request
5. Authentication and Security
6. Summary
7. Questions/Discussion
]

---
.left-column[
## Issues with REST.red[a]
]
.right-column[
### Loose Standardization of REST
- GET, POST, PUT, DELETE – how to map them to your operations?
- My first REST API was always POST+JSON :-)

### Hard to Model Relationships
- In reality we have complex nested objects (BorgBase has ca. 20 models)
- REST is one endpoint per model `acme.com/repos/:id`
- Populating a complete UI would need many fetches.

### Over-/Under Fetching
- Client has no control over which attributes to fetch.
- Often we only want some attributes of a model.

.footnote[
  .red[a] [REST vs. GraphQL](https://medium.com/codingthesmartway-com-blog/rest-vs-graphql-418eac2e3083) by Sebastian Eschweiler
  ]
]
---
![eat](assets/graphql/rest-fetches.png)
.footnote[
  From: [How To GraphQL](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/)]
---
![eat](assets/graphql/graphql-fetch.png)
.footnote[
  From: [How To GraphQL](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/)]
---
.left-column[
## Core Concepts
### Types

]
.right-column[
GraphQL has its own Schema Definition Language.

### Type
- Usually map to your database models
- Types consist of primitives (`String`, `Int`, ..) or other Types

```graphql
type Post {
  title: String!
  author: Person!
}
```

```graphql
type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}
```

.footnote[
  Examples from [How To GraphQL](https://www.howtographql.com/basics/2-core-concepts/)]
]
---
.left-column[
## Core Concepts
### Types
### Queries
]
.right-column[
## Query
The default operation (`query` keyword can be ommitted)
```graphql
[query] {
  allPersons {
    name
    age
  }
}
```

```json
{
  "allPersons": [
    { "name": "Johnny", "age": 23 },
    { "name": "Sarah", "age": 32 },
    { "name": "Alice", "age": 45 }
  ]
}
```

]
---
.left-column[
## Core Concepts
### Types
### Queries
### Mutations
]
.right-column[
## Mutation
Make changes to backend data
```graphql
mutation {
  createPerson(name: "Alice", age: 36) {
    id
  }
}
```

```json
{
  "data": {
    "createPerson": {
      "id": "cjrfslcnx084e0172fpijdgqg"
    }
  }
}
```

]
---
.left-column[
## Core Concepts
### Types
### Queries
### Mutations
### Arguments
]
.right-column[
## Arguments
Reuse queries and pass data
```graphql
mutation newPerson($name:String!, $age: Int!) {
    createPerson(name: $name, age: $age) {
      person { id }
  }
} 
```

```json
{
  "data": {
    "createPerson": {
      "person": {
        "id": "7"
      }
    }
  }
}
```

]
---
.left-column[
## Core Concepts
### Types
### Queries
### Mutations
### Arguments
### Nesting
]
.right-column[
## Nested Queries
You can request nested attributes of result instances.
```graphql
mutation {
    createPost(title: "My GraphQL Musings", author: 1) {
      post { 
        id,
        title,
        author {id, name, age }
      }
  }
} 
```

```json
{
  "data": {
    "createPost": {
      "post": {
        "id": "3",
        "author": {
          "name": "Lisa"
        }
      }
    }
  }
}
```

]
---
.left-column[
## Core Concepts
### Types
### Queries
### Mutations
### Arguments
### Nesting
### Subscriptions
]
.right-column[
### Realtime Updates
- Client subscribes to steady updates an receives e.g. new chat messages
- Not widely implmented yet.red[a]
- WebSockets are often used for transport
- I see WebSockets used *next to* GraphQL more often

```graphql
subscription {
  newPerson {
    name
    age
  }
}
```
.footnote[
  .red[a] [Apollo for NodeJS](https://blog.apollographql.com/tutorial-graphql-subscriptions-server-side-e51c32dc2951)

]
]
---
.left-column[
## Core Concepts
### Types
### Queries
### Mutations
### Arguments
### Nesting
### Subscriptions
### Fragments
]
.right-column[
### Fragments
- Often you want to query the same set of fields
- Keep it DRY with Fragments

```graphql
fragment addressDetails on User {
  name
  street
  zipcode
  city
}
```

Use fragment in query:
```graphql
{
  allUsers {
    ... addressDetails
  }
}
```
]
---
.left-column[
## Demo
### Backend
]
.right-column[
### Demo Implementation.red[a] in Graphene + Django
- Graphene.red[b] is a server-side Python-implementation of GraphQL.
- Allows to quickly link Django models to GraphQL types, queries and mutations.

### `Person`

```python
class Person(models.Model):
    name = models.CharField(max_length=128)
    age = models.IntegerField()
```

### `allPerson`
```python
class Query(graphene.ObjectType):
    all_persons = graphene.List(PersonType)

    def resolve_all_persons(self, info):
        return Person.objects.all()
```

.footnote[
  .red[a] Minimal demo app: [https://github.com/m3nu/graphql-demo-backend](https://github.com/m3nu/graphql-demo-backend)

  .red[b] [https://github.com/graphql-python/graphene-django](https://github.com/graphql-python/graphene-django)
  ]
]
---
.left-column[
## Demo
### Backend
### Frontend
]
.right-column[
Some popular JavaScript client libraries:

- Relay.red[a]: original Facebook client
- Apollo.red[b]: most comprehensive client, lots of features
- Graphql-request.red[c]: minimal client

### Minimal Vue.js Implementation.red[d]
```javascript
getPersons () {
  const query = `{
                    allPersons {
                        id, name, age, postSet { title }
                    }
                 }`

  request(this.graphqlEndpoint, query).then(data =>
    this.persons = data.allPersons
  )
```
]
.footnote[
  .red[a] [https://facebook.github.io/relay/](https://facebook.github.io/relay/)

  .red[b] [https://www.apollographql.com/docs/react/](https://www.apollographql.com/docs/react/)

  .red[c] [https://github.com/prisma/graphql-request](https://github.com/prisma/graphql-request)

  .red[d] [https://github.com/m3nu/graphql-demo-frontend](https://github.com/m3nu/graphql-demo-frontend)
  ]
---
.left-column[
## Auth
]
.right-column[
### How to Control Access in GraphQL? 
- Same endpoint/URL for all queries
- Restrict per-query/mutation
- e.g. `@is_authenticated` decorator

### Authentication
- Cookie (httpOnly)
- Header token (e.g. JWT)

### Security.red[a]
- Query depth
- Query complexity
- Throttle
.footnote[
  .red[a] [https://www.howtographql.com/advanced/4-security/](https://www.howtographql.com/advanced/4-security/)
]
]
---
.left-column[
## Auth
### Backend
]
.right-column[
### Django
- Graphene works well with standard Django sessions
- JWT middleware is available

#### Login and Authentication Example

```python
class Login(graphene.Mutation):
    ok = graphene.Boolean()

    class Arguments:
        username = graphene.String(required=True)
        password = graphene.String(required=True)

    def mutate(self, info, **input):
        user = authenticate(username=input['username'], password=input['password'])
        if user is not None:
            login(info.context, user)
            return Login(True)
        else:
            raise Exception('Invalid username or password!')

```
```python
class CreatePerson(graphene.Mutation):
...
    def mutate(self, info, **kwargs):
        if not info.context.user.is_authenticated:
            raise Exception('Please authenticate first.')
```
]
---
.left-column[
## Summary
]
.right-column[
### Compared to REST
- Solves problems, like over/underfetching, nesting, connections
- Minimizes number of HTTP calls

### GraphQL Schema
- Uses its own Schema Definition Language
- First define entities: `PersonType`, `PostType`
- Then define operations: Query `allPersons`, Mutation `createPerson`
- Decide on the fields to fetch, can be nested

### Implementations
- Graphene is the default for Django
- Apollo is the default JavaScript client, but alternatives are available.
- Most Django concepts can be reused (sessions, authentication)
]
---
name: inverse
class: center, middle, inverse
## Questions/Discussion
---
name: inverse
class: center, middle, inverse
## Thanks for coming

Twitter: .pink[@_m3nu]

Slides: .pink[github.com/m3nu/talks]
