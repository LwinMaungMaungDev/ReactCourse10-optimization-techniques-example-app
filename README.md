# React Native course by Maximilian Schwarzmüller

## Profiler

- Install React Dev Tools for Chrome.
- Open the profiler.
- Click the start profiling circle button.
- Interact with the page (click a button).
- Stop the profiling.
- You can switch between Flamegraph chart and Ranked chart representation as you prefer.

### Flamegraph chart

With the Flamegraph chart, you get a representation that also shows you the order in which component functions were executed. You also get a relation between component functions. For example, we can see the App component at the top because it's the root component. You can see its children under it.
If you hover over the App or Header, you can see "Did not client render". This means your interaction (clicking a button) didn't make these components to get executed again.
All components that are in some colors are executed again. You can see the component names if you hover over them. The component that causes the re-render is colored orange and its children that are also get re-rendered are colored green.

### Ranked chart

Here, we only see the components that were re-rendered.

### Setting

You can see the setting button. Under profiler tab, we can check "Record why each component rendered while profiling.". Then start the profiling again, make interactions and stop profiling. Then if you hover over the components, you can see the reason why each component gets rendered.

## memo

Here, whenever we type something in input, the _handleChange_ will get triggered and call the _setEnteredNumber_. This will cause the component re-render. Notice this is the **App** component (the root component). This will cause all children to execute again, including the **Counter** component. To fix that we can use **_memo_**.

```
function App() {
  log("<App /> rendered");
  const [enteredNumber, setEnteredNumber] = useState(0);
  const [chosenCount, setChosenCount] = useState(0);
  function handleChange(event) {
    setEnteredNumber(+event.target.value);
  }
  function handleSetClick() {
    setChosenCount(enteredNumber);
    setEnteredNumber(0);
  }
  return (
    <>
      <Header />
      <main>
        <section id="configure-counter">
          <h2>Set Counter</h2>
          <input type="number" onChange={handleChange} value={enteredNumber} />
          <button onClick={handleSetClick}>Set</button>
        </section>
        <Counter initialCount={chosenCount} />
      </main>
    </>
  );
}
export default App;

```

**_memo_** will take a look at the props of the component function. Whenever the component function would normally execute again, **_memo_** will take a look at the **old prop value** and at the **new prop value**. If those values are exactly the same (with arrays and objects, they have to be exactly the same array or object in memory), this component function execution will be prevented by memo. This component function will only get executed again when its props change or its internal states change. **_memo_** only prevents function executions that are triggered by the parent component. Get the **_memo_** function from **_react_**.

```
const Counter = memo(function Counter({ initialCount }) {
  log("<Counter /> rendered", 1);
  const initialCountIsPrime = isPrime(initialCount);
  const [counter, setCounter] = useState(initialCount);
  function handleDecrement() {
    setCounter((prevCounter) => prevCounter - 1);
  }
  function handleIncrement() {
    setCounter((prevCounter) => prevCounter + 1);
  }
  return (
    <section className="counter">
      …
      <p>
        <IconButton icon={MinusIcon} onClick={handleDecrement}>
          Decrement
        </IconButton>
        <CounterOutput value={counter} />
        <IconButton icon={PlusIcon} onClick={handleIncrement}>
          Increment
        </IconButton>
      </p>
    </section>
  );
});
export default Counter;

```

### Don't overuse memo()! (Component Composition Is Better)

Wrap it around a component that's as high up in the component tree as possible. If that component is then prevented from executing again, all the nested components also won't be executed again.

If you would wrap memo around all your components, that would simply mean that React always has to check the props before it executes the component function. Checking the prop values for equality also costs some performance.
"memo" is not only way to prevent unnecessary renders. Another technique that is often even more powerful than memo is a clever component composition.

Nothing fancy. We just create the Counter component and separate the user input state in that component. So, the App component will not get executed again every time the user types something.

```
function App() {
  log("<App /> rendered");
  const [chosenCount, setChosenCount] = useState(0);
  function handleSetCount(newCount) {
    setChosenCount(newCount);
  }
  return (
    <>
      <Header />
      <main>
        <ConfigureCounter onSetCount={handleSetCount} />
        <Counter initialCount={chosenCount} />
      </main>
    </>
  );
}

```

### Use useCallback with memo

Here, you can see handleIncrement and handleDecrement functions will cause the state update and make this Counter component gets re-rendered. When it re-renders, the IconButton components will also get re-rendered. But this IconButton doesn't care about the counter state, so re-rendering is unnecessary.

So, we did wrap with "memo" inside the IconButton component just like we did above. But that won't be enough. When this Counter component is executed again, the handleDecrement and handleIncrement functions will also be recreated. Since we pass them to the IconButton component, that makes the props in IconButton change and which will still cause the re-renders of IconButton. So, we use "useCallback" here.

```
const Counter = memo(function Counter({ initialCount }) {
  const initialCountIsPrime = isPrime(initialCount);
  const [counter, setCounter] = useState(initialCount);
  const handleDecrement = useCallback(() => {
    setCounter((prevCounter) => prevCounter - 1);
  }, []);
  const handleIncrement = useCallback(function handleIncrement() {
    setCounter((prevCounter) => prevCounter + 1);
  }, []);
  return (
    <section className="counter">
      …
      <p>
        <IconButton icon={MinusIcon} onClick={handleDecrement}>
          Decrement
        </IconButton>
        <CounterOutput value={counter} />
        <IconButton icon={PlusIcon} onClick={handleIncrement}>
          Increment
        </IconButton>
      </p>
    </section>
  );
});

```

## useMemo

Notice the "isPrime" function here. It uses the "initialCount" value which is the prop in this Counter component. Unless this initialCount changes, "isPrime" will always yield the same result.

So, when the internal state (counter here) of this Counter component changes, this "isPrime" function will be executed although we know this will yield the same result because the initialCount hasn't changed.

```
const Counter = memo(function Counter({ initialCount }) {
  const initialCountIsPrime = isPrime(initialCount);
  const [counter, setCounter] = useState(initialCount);
  const handleDecrement = useCallback(() => {
    setCounter((prevCounter) => prevCounter - 1);
  }, []);
  const handleIncrement = useCallback(function handleIncrement() {
    setCounter((prevCounter) => prevCounter + 1);
  }, []);
  return ( JSX );
});

```

So, we wrap this "isPrime" function execution with "useMemo".

```
  const initialCountIsPrime = useMemo(
    () => isPrime(initialCount),
    [initialCount]
  );

```

## Virtual DOM

When the app first loads, React creates a Virtual DOM and from which HTML elements. When a component gets executed again because of state updates, React creates a new Virtual DOM starting from the component that triggers the state changes (Not the whole Virtual DOM). Then React compares the new Virtual DOM with the old Virtual DOM and decides which HTML elements should be updated. Finally, only parts of the real DOM (HTML) are updated.

## Keys Matter When Managing State

**React tracks state by component type and position (of that component) in the tree.** If we don't use keys in the components list, that would be okay as long as the positions of the components don't change.

```
<ol>
  {history.map((count, index) => (
    <HistoryItem key={index} count={count} />
  ))}
</ol>

```

With this key assignment above, it also affects the performance. Since the key is not unique and changing when the items are added or removed from the list, all the list items inside the list will be destroyed and re-created.
If we use the unique value as the key, React knows these items exist in the DOM before and only adds the new items to the list affecting only this one item.

We can also use keys to reset a component when some state changes. Use this approach instead of using the "useEffect" (inside the Counter component) because "useEffect" enforces the additional re-renders.

```
<Counter key={chosenCount} initialCount={chosenCount} />
```

## State Scheduling & Batching

When you call a state updating function, the state update will be scheduled by React. It will not be executed instantly.

If we call two state updating functions, the component function won't run twice. It only runs once. That is the case because React performs state batching. This means multiple state updates that are triggered from the same function are batched together and will only lead to one component function execution.

## MillionJS

Search for "millionjs". You can go with Automatic or Manual modes.
Automatic => npx million@latest

This will also help update the "vite.config.js" file like below.

```
import MillionLint from '@million/lint';
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [MillionLint.vite({
    enabled: true
  }), react()],
})

```

Then you can just run => npm run dev

To solve the conflict with some components, you can ignore the component file with this line just above the component function.
=>

```
// million-ignore
```
