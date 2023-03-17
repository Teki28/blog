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
