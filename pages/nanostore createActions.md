## TL:DR
I made createActions utility like redux-toolkit's createSlice. TypeScript ready. Good Type Inference.
## Problem
It takes a long time to create multiple [actions](https://github.com/nanostores/nanostores#actions). 

There are many boilerplates now. Like next TodoList example.

```ts
export const todoListStore = atom<TodoT[]>([])

export const actions = {
addTodo: action(todoListStore, 'addTodo', (store, content: string) => {
  const newTodo = {
    id: generateId(),
    content,
    isCompleted: false,
  }
  store.set([...store.get(), newTodo])
}),
completeTodo: action(todoListStore, 'completeTodo', (store, id: number, isCompleted: boolean) => {
  const todoList = store.get();
  const todo = todoList.find((todo) => todo.id === id)
  if(todo){
    todo.isCompleted = isCompleted
  }
  store.set([...todoList]);
}),
changeTodo: action(todoListStore, 'changeTodo', (store,id: number, newContent: string) => {
  const todoList = store.get();
  const todo = todoList.find((todo) => todo.id === id)
  if(todo){
    todo.content = newContent
  }
  store.set([...todoList]);
}),
deleteTodo: action(todoListStore, 'deleteTodo', (store, id: number) => {
  store.set(store.get().filter((todo) => todo.id !== id)) 
})
}
```

Did you find any duplicated patterns here?
```ts
{
[name]: action(store, name, (store, args) => {
  const oldState = store.get();
  ...
  store.set(newState)
})
}
```
## Solution
So I made createActions utility like redux-toolkit's createSlice. Here it is...

```ts
export const todoListStore = atom<TodoT[]>([]);

// use createActions
export const actions = createActions(todoListStore,  {
// just pure old javascript function
addTodo(old, content: string) {
  if (content.length === 0) return old;
  
  const newTodo = {
    id: generateId(),
    content,
    isCompleted: false,
  };
  return [...old, newTodo]; // return next state
},
completeTodo(old, id: TodoT["id"], isCompleted: boolean) {
  return old.map((todo) =>
    todo.id === id ? { ...todo, isCompleted } : todo
  );
},
changeTodo(old, id: TodoT["id"], content: string) {
  return old.map((todo) => (todo.id === id ? { ...todo, content } : todo));
},
deleteTodo(old, id: TodoT["id"]) {
  return old.filter((todo) => todo.id !== id);
},
});
```

It is Typescript ready, and can inference types well. [Thanks to](https://twitter.com/xiniha_1e88df/status/1562330128684032000) [@XiNiha](https://github.com/XiNiHa) 


![image](https://user-images.githubusercontent.com/13915810/186354821-29860b38-3896-4317-842a-23a644829f83.png)
## Implementation

It is simple, but just a proof of concept.

```ts
// XiNiHa's work
type AsAction<T, I extends Record<string, (old: T, ...args: any[]) => T>> = {
[K in keyof I]: I[K] extends (old: T, ...args: infer A) => T ? (...args: A) => void : never
}

function createActions<
T,
I extends Record<string, (old: T, ...args: any) => T>
>(store: WritableAtom<T>, rawActions: I): AsAction<T, I> {
return Object.fromEntries(
  Object.entries(rawActions).map(([name, rawAction]) => {
    return [
      name,
      action(store, name, (store, ...args) => {
        const old = store.get();
        store.set(rawAction(old, ...args));
      }),
    ];
  })
) as AsAction<T, I>;
}
```
## Limitation