# Recipes for @apollo/react-hooks to create scalable & maintainable applications 🚀
___

### Why?
By popular convention, mutation and query logic is co-located with our components.
With the increased amount of logic in our components due to more advanced usage of Mutations and Querys, our components quickly become bloated. Co-located logic causes unnecessary cognitive load for readers of the code and is harder to maintain. 

### How?

> “Building your own Hooks lets you extract component logic into reusable functions.”
-React docs

*We are going to create hooks by using the @apollo/react-hooks and extracting our Mutation and Query logic into reusable functions.*

### Examples
**Instead of:**
```javascript
import React from 'react'
import { useMutation, useQuery } from '@apollo/react-hooks'
import { gql } from 'apollo-boost'

const deleteTodoMutation = gql`
  mutation DeleteTodo($id: String!) {
    deleteTodo(id: $id)
  }
`;

const getTodosQuery = gql`
    query todos {
        todos {
            id
            title
            content
        }
    }
`

const Todos = () => {
    const [mutate] = useMutation(deleteTodoMutation)

    const { loading, error, data } = useQuery(getTodosQuery)
    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error :(</p>;

    return data.todos.map(todo => (
        <div key={todo.id}>
            <h3> {todo.title} </h3>
            <p> {todo.content} </p>
            <button onClick={() => mutate({
                variables: { id: todo.id },
                update: cache => {
                    const store = cache.readQuery({
                        query: getTodosQuery,
                        data: {
                            todos: store.todos.filter(t => t.id !== id)
                        }
                    })
                }
            })}>
                Delete
            </button>

        </div>
    ))

}

export default Todos
```



**We much rather:**
```javascript
import React from 'react'
import { useTodosQuery } from 'recipes/todos/queries'
import { useDeleteTodoMutation } from 'recipes/todos/mutations'

const Todos = () => {
    const deleteTodo = useDeleteTodoMutation()

    const { loading, error, data } = useTodosQuery()
    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error :(</p>;

    return data.todos.map(todo => (
        <div key={todo.id}>
            <h3> {todo.title} </h3>
            <p> {todo.content} </p>
            <button onClick={() => deleteTodo(todo.id)}>
                Delete
            </button>

        </div>
    ))
}

export default Todos
```


**Our `useDeleteTodoMutation` mutation hook would look like:**
```javascript
import { gql } from "apollo-boost";
import { useMutation } from "@apollo/react-hooks";
import { getTodosQuery } from "./queries";

const deleteTodoMutation = gql`
  mutation DeleteTodo($id: String!) {
    deleteTodo(id: $id)
  }
`;

export function useDeleteTodoMutation() {
  const [mutate] = useMutation(deleteTodoMutation);

  return (id) => {
    return mutate({
      variables: { id },
      update: proxy => {
        const store = proxy.readQuery({
          query: getTodosQuery
        });

        proxy.writeQuery({
          query: getTodosQuery,
          data: {
            todos: store.todos.filter(todo => todo.id !== id)
          }
        });
      }
    });
  };
}

```

**Our `useGetTodosQuery` would then look like:**
```typescript
import { gql } from 'apollo-boost'
import { useQuery } from '@apollo/react-hooks'

export const getTodosQuery = gql`
    query todos {
        todos {
            id
            title
            content
        }
    }
`

export const useTodosQuery = () => useQuery(getTodosQuery)
```


#### Recommended patterns 
**Structure**:

We will be using the “ducks” pattern popularized by Redux by keeping all our logic for a feature in the same file.
An example folder structure might look something like:
```
    /src
      index.tsx
      /components
      ....
      /recipes
          /todos
              mutations.ts
              queries.ts
```


  **Naming**:
   
  We try to mirror the query type definition for our queries.
  
  If we have a `query getTodos {...` we would call that: `useGetTodosQuery`
    
  If we have 2 queries using the same query typedef we can name it after fields / usage i.e.
  `useGetTodosTitlesQuery`
