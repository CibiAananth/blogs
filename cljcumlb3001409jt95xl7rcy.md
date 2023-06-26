---
title: "React.useState with "as const""
seoTitle: "React.useState with "as const""
seoDescription: "Optimize React.useState with "as const" for type safety, preventing bugs of the states in your components"
datePublished: Mon Jun 26 2023 12:42:27 GMT+0000 (Coordinated Universal Time)
cuid: cljcumlb3001409jt95xl7rcy
slug: as-const-in-react
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687774918683/4c9ce807-8ce6-4494-8dbc-64c76e745e42.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1687782911389/93b4dd8d-6092-482c-a4a1-ef1d2f29d10a.png
tags: javascript, web-development, react-native, reactjs, typescript

---

Let's take a typical React Component that fetches data from an external source and updates the UI based on the network state

```typescript
// Users.tsx
import { useState, useEffect } from "react";

export const REQUEST_STATUS = {
  idle: "IDLE",
  pending: "PENDING",
  success: "SUCCESS",
  error: "ERROR",
};

async function fetchUsers(): Promise<{ name: string; id: number }[]> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([{ name: "User 1", id: 1 }]);
    }, 2000);
  });
}

export function App() {
  const [networkState, setNetworkState] = useState(REQUEST_STATUS.idle);
  const [users, setUsers] = useState<{ name: string; id: number }[]>([]);

  useEffect(() => {
    setNetworkState(REQUEST_STATUS.pending);
    // or setNetworkState("IDLE"); // valid
    const getUsers = async () => {
      const users = await fetchUsers();
      setUsers(users);
      setNetworkState(REQUEST_STATUS.success);
    };
    getUsers();
  }, []);

  return (
    <div>
      {networkState === "PENDING" ? (
        <p>Fetching data...</p>
      ) : (
        networkState === "SUCCESS" && (
          <li>
            {users?.map((user) => (
              <ul key={user.id}> {user.name} </ul>
            ))}
          </li>
        )
      )}
      {networkState === "ERROR" && (
        <div>An error occured while fetching users</div>
      )}
    </div>
  );
}
```

At first glance, this code may seem perfectly fine and in fact, this will render the component without any error.

**BUT...**

Let's assume that the `REQUEST_STATUS` is imported from some global file and used in our components like,

```typescript
import { REQUEST_STATUS } from '../some/other/file'
```

and someone changed the statuses in the global file to something that breaks our `Users.tsx` component. Boom!

```typescript
export const REQUEST_STATUS = {
  idle: "idle",
  pending: "pending",
  success: "success",
  error: "error",
};
```

## Typechecking in `useState`

While adding a type to the `setNetworkState` is the ultimate fix to this problem, how do we do it without much change in the code?

First, let's infer the type of `REQUEST_STATUS` in our User component,

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687778030310/b8e20e23-a0c7-4c64-85d3-aa212635cd80.png align="center")

Here the type of the keys `idle`, `pending`, `success`, `error` is inferred as a `string` therefore using `keyof` and `typeof` would still result in `string` as the overall type.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687778162427/fdf7cf0c-7498-4bfd-bda5-0a232e3ddf83.png align="center")

## Detour - `keyof` `typeof`

If the above syntax is confusing, let me break it down for you ;) Otherwise, skip to the next section.

### `typeof`

Here TypeScript is going to be able to infer the type of your object for you. It's going to lift this object that you created from the value level to the type level.

so essentially `typeof REQUEST_STATUS` will be inferred as `{ idle: string, pending: string, success: string, error: string }`

So if we want to get a type of particular key in the object, we can do something like

```typescript
type IDLE_TYPE = typeof REQUEST_STATUS["idle"]; // string
type PENDING_TYPE = typeof REQUEST_STATUS["pending"]; // string
type SUCCESS_TYPE = typeof REQUEST_STATUS["success"]; // string
type ERROR_TYPE = typeof REQUEST_STATUS["error"]; // string
```

### `keyof`

The `keyof` operator can be used to extract the keys of an object type.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687778900732/087740e5-bfe1-4a3f-9e49-0057c721dd85.png align="center")

`keyof` can be used in conjunction with `typeof` to create a union type of all keys in the object.

By extracting all the keys of the object type, we can combine these keywords to create union types

```typescript
type REQUEST_TYPE = tyepof REQUEST_STATUS[keyof typeof REQUEST_STATUS] // string

// the above expression is equivalent to
type REQUEST_TYPE =
| { idle: string, pending: string, success: string, error: string }["idle"] 
| { idle: string, pending: string, success: string, error: string }["pending"]
| { idle: string, pending: string, success: string, error: string }["success"]
| { idle: string, pending: string, success: string, error: string }["error"]
```

## Back to rescue with "as const"

Now that we know how `typeof` and `keyof` can be used together to create union type from objects, let's see how "as const" fits in here.

The above type `export type REQUEST_TYPE = tyepof REQUEST_STATUS[keyof typeof REQUEST_STATUS]` returns `string` as the type, so if typing our `setState` wouldn't solve our problem. This is because as long as we set a "string", TypeScript accepts it as a valid value.

```typescript
import { REQUEST_TYPE, REQUEST_STATUS } from '../some/other/file'

const [networkState, setNetworkState] = useState<REQUEST_TYPE>(REQUEST_STATES.idle);

// ...somewhere in the code, we can still do
setNetworkState("idle"); // no error
setNetworkState("invalid_state"); // no error
setNetworkState(12) // Error as type "string" is inferred from initial state
```

Without further explanation, this is where "as const" assertion comes into play and makes it strongly typed. By simply adding `as const` to the object we can make it as readonly and ensure that all properties are assigned the literal type instead of a more general version like `string` or `number`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687780699673/c708e0c7-d266-45c7-ae69-33118c26978f.png align="center")

Combining with our previously learned trick we can create a union type of object values

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687780909419/e80f251b-4af0-40ae-8d80-5bacab52d4e4.png align="center")

## Putting it together

```typescript
// types
export const REQUEST_STATUS = {
  idle: "IDLE",
  pending: "PENDING",
  success: "SUCCESS",
  error: "ERROR",
} as const;

export type REQUEST_TYPES = typeof REQUEST_STATUS[keyof typeof REQUEST_STATUS];

// User.tsx
const [networkState, setNetworkState] = useState<REQUEST_TYPES>(REQUEST_STATUS.idle);
```

With this updated `useState`, TypeScript compiler will throw an error for invalid statuses.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687782249115/cb80a824-ac2c-4fce-9d42-b59bcf974d6c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687782253799/6d539ab7-0626-47b9-a968-896ae125e750.png align="center")

## Conclusion

In conclusion, using "as const" in conjunction with `typeof` and `keyof` can help you create strongly typed union types for your React `useState` hook, preventing potential bugs caused by unexpected value changes in your application. By implementing this approach, you'll ensure better type safety and maintainability in your codebase.

Happy coding!