---
layout: "../../layouts/BlogPostLayout.astro"
title: React Memo
date: 2023-03-17
author: Teki
image: {
  src: "/images/react/cover.png",
  alt: "cover image",
}
description: Some scattered knowledge points about react.
draft: false
category: Coding
---

- ### why react fast?

virtual DOM + Diffing algorithm

- ### Uncontrolled component data acquisition --- Refs

1. #### string ref (class component)

  ``` javascript

  showData = () => {
    const {input1} = this.refs
    alert(input1.value)
  }

  showData2 = () => {
    const {input2} = this.refs
    alert(input2.value)
  }
  ...
  render() {
    return (
    <div>
      <input ref="input1" type="text"/>&nbsp;
      <button onClick={this.showData}>show input1 data</button>
      &nbsp;
      <input ref="input2" onBlur={this.showData2} type="text"/>
    </div>
    )
  }
  ```

2. #### callback func ref (class component)

  ```javascript
  showData = () => {
    const {input1} = this
    alert(input1.value)
  }
  render(){
    return (
        <input ref={c => this.input1 = c} type="text"/>&nbsp;
        <button onClick={this.showData}>alert data</button>
    )
  }
  ```

3. #### createRef (class component)

  ```javascript
  myRef = React.createRef()
  showData = () => {
    alert(this.myRef.current.value);
  }
  ...
  <input ref={this.myRef} type="text"/>&nbsp;
  <button onClick={this.showData}>show data</button>
  ```

- ### lifecycle and related hooks

![lifecycle diagram](/public/images/react/lifecycle.jpeg)

- ### components

1. #### Fragment (<>)

  Fragment, often used via <>...</> syntax, lets you group elements without a wrapper node.

  ```html
    <>
      <OneChild />
      <AnotherChild />
    </>
  ```

2. #### Profiler

  Profiler lets you measure rendering performance of a React tree programmatically.

  ```javascript
  function onRender(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  // Aggregate or log render timings...
  } 
  <Profiler id="App" onRender={onRender}>
    <App />
  </Profiler>
  ```

3. #### StrictMode

  StrictMode lets you find common bugs in your components early during development.

  ```html
  <StrictMode>
    <App />
  </StrictMode>
  ```

4. #### Suspense

  Suspense lets you display a fallback until its children have finished loading.

  ```javascript
  <Suspense fallback={<Loading />}>
    <SomeComponent />
  </Suspense>
  ```

### hooks

1. #### useCallback

  useCallback is a React Hook that lets you cache a function definition between re-renders.

  ```javascript
  const cachedFn = useCallback(fn, dependencies)
  ```

  Parameters:
  fn: a callback function which is returned by useCallback
  dependencies: dependencies which fn is relying on. When dependencies changed, a new fn will be returned, otherwise, the same fn will be returned.
  Usage: Skipping re-rendering of certain component by useCallback and memo

  ```javascript
  function ProductPage({ productId, referrer, theme }) {
    // Tell React to cache your function between re-renders...
    const handleSubmit = useCallback((orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    }, [productId, referrer]); // ...so as long as these dependencies don't change...

    return (
      <div className={theme}>
        {/* ...ShippingForm will receive the same props and can skip re-rendering */}
        <ShippingForm onSubmit={handleSubmit} />
      </div>
    );
  }

  import { memo } from 'react';

  const ShippingForm = memo(function ShippingForm({ onSubmit }) {
    // ...
  });
  ```

2. #### useContext

  useContext is a React Hook that lets you read and subscribe to context from your component.

  ```javascript
  const value = useContext(SomeContext)
  ```

  useContext returns the context value for the calling component. It is determined as the value passed to the closest SomeContext.Provider above the calling component in the tree. If there is no such provider, then the returned value will be the defaultValue you have passed to createContext for that context. The returned value is always up-to-date. React automatically re-renders components that read some context if it changes.
  3. #### useDebugValue
  useDebugValue is a React Hook that lets you add a label to a custom Hook in React DevTools.

  ```javascript
  useDebugValue(value, format?)
  ```

4. #### useDeferredValue

  useDeferredValue is a React Hook that lets you defer updating a part of the UI.

  ```javascript
  const deferredValue = useDeferredValue(value)
  ```

  ```javascript
  import { useState, useDeferredValue } from 'react';

  function SearchPage() {
    const [query, setQuery] = useState('');
    const deferredQuery = useDeferredValue(query);
    // ...
  }
  ```

  when query changed, deferredQuery will be stale value at first, then it will change to new value. So there is a lag existing.
  Usage: For searching, old results could be kept until new result is ready.

  ```javascript
  import { Suspense, useState, useDeferredValue } from 'react';
  import SearchResults from './SearchResults.js';

  export default function App() {
    const [query, setQuery] = useState('');
    const deferredQuery = useDeferredValue(query);
    return (
      <>
        <label>
          Search albums:
          <input value={query} onChange={e => setQuery(e.target.value)} />
        </label>
        <Suspense fallback={<h2>Loading...</h2>}>
          <SearchResults query={deferredQuery} />
        </Suspense>
      </>
    );
  }
  ```

  ```
5. #### useEffect
useEffect is a React Hook that lets you synchronize a component with an external system.
```javascript
useEffect(setup, dependencies?)
```

Parameters:

- setup: The function with your Effect’s logic. Your setup function may also optionally return a cleanup function. When your component is first added to the DOM, React will run your setup function. After every re-render with changed dependencies, React will first run the cleanup function (if you provided it) with the old values, and then run your setup function with the new values. After your component is removed from the DOM, React will run your cleanup function one last time.

- optional dependencies: The list of all reactive values referenced inside of the setup code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is configured for React, it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like [dep1, dep2, dep3]. React will compare each dependency with its previous value using the Object.is comparison. If you omit this argument, your Effect will re-run after every re-render of the component. See the difference between passing an array of dependencies, an empty array, and no dependencies at all.

example:Connecting to an external system

```javascript
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
   const connection = createConnection(serverUrl, roomId);
    connection.connect();
   return () => {
      connection.disconnect();
   };
  }, [serverUrl, roomId]);
  // ...
}
```

6. #### useId

useId is a React Hook for generating unique IDs that can be passed to accessibility attributes.

```javascript
const id = useId()
```

example:

```javascript
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...

<>
  <input type="password" aria-describedby={passwordHintId} />
  <p id={passwordHintId}>
</>

```

7. #### useImperativeHandle

useImperativeHandle is a React Hook that lets you customize the handle exposed as a ref.

```javascript
useImperativeHandle(ref, createHandle, dependencies?)
```

example:Exposing a custom ref handle to the parent component

```javascript
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

8. #### useInsertionEffect

useInsertionEffect is a version of useEffect that fires before any DOM mutations.

```javascript
useInsertionEffect(setup, dependencies?)
```

example: Only use for CSS-in-JS lib

```javascript

// Inside your CSS-in-JS library
let isInserted = new Set();
function useCSS(rule) {
  useInsertionEffect(() => {
    // As explained earlier, we don't recommend runtime injection of <style> tags.
    // But if you have to do it, then it's important to do in useInsertionEffect.
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Button() {
  const className = useCSS('...');
  return <div className={className} />;
}
```

9. #### useLayoutEffect

useLayoutEffect is a version of useEffect that fires before the browser repaints the screen.

```javascript
useLayoutEffect(setup, dependencies?)
```

example:Measuring layout before the browser repaints the screen

```javascript
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // You don't know real height yet

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); // Re-render now that you know the real height
  }, []);

  // ...use tooltipHeight in the rendering logic below...
}
```

10. #### useMemo

useMemo is a React Hook that lets you cache the result of a calculation between re-renders.

```javascript
const cachedValue = useMemo(calculateValue, dependencies)
```

example:Skipping expensive recalculations

```javascript
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

11. #### useReducer

useReducer is a React Hook that lets you add a reducer to your component.

```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

example:

```javascript
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

12. #### useRef

useRef is a React Hook that lets you reference a value that’s not needed for rendering.

```javascript
const ref = useRef(initialValue)
```

example: manipulating DOM by ref

```javascript
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

13. #### useState

useState is a React Hook that lets you add a state variable to your component.

```javascript
const [state, setState] = useState(initialState)
```

example: Adding state to a component

```javascript
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
```

14. #### useSyncExternalStore

useSyncExternalStore is a React Hook that lets you subscribe to an external store.

```javascript
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

example:

```javascript
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

// This is an example of a third-party store
// that you might need to integrate with React.

// If your app is fully built with React,
// we recommend using React state instead.

let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
    emitChange();
  },
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() {
    return todos;
  }
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}
```

example:Subscribing to a browser API

```javascript
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

15. #### useTransition

useTransition is a React Hook that lets you update the state without blocking the UI.

```javascript
const [isPending, startTransition] = useTransition()
```

example:Marking a state update as a non-blocking transition

```javascript
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

- ### APIs

1. #### createContext

createContext lets you create a context that components can provide or read.

```javascript
const SomeContext = createContext(defaultValue)
```

example:

```javascript
function App() {
  const [theme, setTheme] = useState('light');
  // ...
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}
```

```javascript
function App() {
  const [theme, setTheme] = useState('dark');
  const [currentUser, setCurrentUser] = useState({ name: 'Taylor' });

  // ...

  return (
    <ThemeContext.Provider value={theme}>
      <AuthContext.Provider value={currentUser}>
        <Page />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```

2. #### forwardRef

forwardRef lets your component expose a DOM node to parent component with a ref.

```javascript
const SomeComponent = forwardRef(render)
```

example: Exposing a DOM node to the parent component

```javascript
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```

This lets the parent Form component access the Input DOM node exposed by MyInput:

```javascript
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

3. #### lazy

lazy lets you defer loading component's code until it is rendered for the first time.

```javascript
const SomeComponent = lazy(load)
```

```javascript
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

4. #### memo

memo lets you skip re-rendering a component when its props are unchanged.

```javascript
const MemoizedComponent = memo(SomeComponent,arePropsEqual?)
```

example:

```javascript
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

5. #### startTransition

startTransition lets you update the state without blocking the UI.

```javascript
import { startTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```
