# Repository: Retrieving Data

Vuex ORM provides a handful of methods to retrieve inserted data. In this section, we'll walk through various ways to query the Vuex Store.

When querying the data from the store, in most cases, you want to query data inside the computed property. This way, when any data changes, the part of your application where you use those data gets updated reactively. It works as same as Vuex Getters.

```html
<template>
  <ul>
    <li v-for="user in users" :key="user.id">
      {{ user.name }}
    </li>
  </ul>
</template>

<script>
import { mapRepos } from '@vuex-orm/core'
import User from '@/models/User'

export default {
  computed: {
    ...mapRepos({
      userRepo: User
    }),

    users () {
      return this.userRepo.all()
    }
  }
}
</script>
```

::: tip NOTE
To retrieve records with its relationships, you must explicitly specify which relationships to fetch by with the query chain. For example, if you want to fetch users with posts, you must do `userRepo.with('posts').get()`. See [Relationships](../relationships/getting-started) page for more detail.
:::

## Retrieving Models

Once you have created the repository, you are ready to start retrieving data from the store. Think of each repository as a powerful query builder allowing you to fluently query the store associated with the repository model. For example:

```js
// Get all users from the store.
const users = store.$repo(User).all()
```

The `all` method will return all of the results in the model's store. Since each repository serves as a query builder, you may also add constraints to queries, and then use the `get` method to retrieve the results.

```js
// Get all active users.
const users = store.$repo(User).where('active', true).get()
```

Don't worry, we'll cover all of the available query methods in upcoming sections.

Note that the retrieved models are indeed the model instances. Not the plain object. For example, when you call `store.$repo(User).get()`, you'll get a list of `User` model instances as a result.

## Retrieving Single Models

In addition to retrieving all of the records for the given store, you may also retrieve single records using `find` or `first`. Instead of returning a collection of models, these methods return a single model instance.

```js
// Retrieve a model by its primary key.
const user = store.$repo(User).find(1)

// Retrieve the first model matching the query constraints.
const user = store.$repo(User).where('active', true).first()
```

## Where Clauses

You may use the `where` method to add `where` clauses to the query. The most basic call to `where` requires 2 arguments. The first argument is the name of the field. The second argument is the value to evaluate against the field.

For example, here is a query that verifies the value of the "votes" column is equal to 100:

```js
const users = store.$repo(User).where('votes', 100).get()
```

You may also define the second argument as a closure to perform advanced checks.

```js
const users = store.$repo(User).where('votes', (value) => {
  return value >= 100
}).get()
```

Finally, the first argument can be a closure to perform more powerful querying.

```js
const users = store.$repo(User).where((user) => {
  return user.vote >= 100 && user.active
}).get()
```

### Or Statements

You may chain where constraints together as well as add "or" clauses to the query. The `orWhere` method accepts the same arguments as the where method.

```js
const users = store.$repo(User)
  .where('votes', 100)
  .orWhere('name', 'John Doe')
  .get()
```

If you need to group an "or" condition within parentheses, you should simply use closure to do so.

```js
const users = store.$repo(User)
  .where('votes', 100)
  .orWhere((user) => {
    return user.votes > 50 && user.name === 'Jane Doe'
  })
  .get()
```

## Ordering

The `orderBy` method allows you to sort the result of the query by a given field. The first argument to the `orderBy` method should be the field you wish to sort by, while the second argument controls the direction of the sort, and maybe either `asc` or `desc`. If there is no second argument, the direction is going to defaults to `asc`.

```js
// Order users by name.
const users = store.$repo(User).orderBy('name').get()

// You may also chain orderBy.
const users = store.$repo(User)
  .orderBy('name')
  .orderBy('age', 'desc')
  .get()
```

You may also pass a function as the first argument. The function will accept a record that is being sorted, and it should return value to be sorted by.

```js
// Sort user name by its third character.
const users = store.$repo(User).orderBy(user => user.name[2]).get()
```

## Limit & Offset

To limit the number of results returned from the query, or to skip a given number of results in the query, you may use the `limit` and `offset` methods.

```js
const user = store.$repo(User).limit(30).offset(10).get()
```
