💭
The results confirm that the standard approach is still using `renderHook` and `act` from `@testing-library/react` (the `react-hooks-testing-library` was merged into it).

Let's construct the response.

Plan:

1. Explain the "Invariant Violation" issue (Hooks can only be called inside the body of a function component).
2. Introduce `@testing-library/react`'s `renderHook`.
3. Provide a clear code example: Testing a basic `useCounter` hook. (Show `result.current`).
4. Explain testing state updates with `act()`.
5. Explain testing hooks with dependencies/props using `initialProps` and `rerender`.

Let's write it cleanly in Markdown.

```javascript
// Step 1: The Hook
import { useState } from "react";

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = () => setCount((c) => c + 1);
  return { count, increment };
}
```

```javascript
// Step 2: The Test
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

test("should increment the counter", () => {
  const { result } = renderHook(() => useCounter(0));

  expect(result.current.count).toBe(0);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

I will add a second example for asynchronous hooks (like fetching), showing `waitForNextUpdate` or `waitFor`. React Testing Library recommends `waitFor`.

Let's ensure we enforce SI Path C: No follow up chips or questions.

Let's write the response.To test a custom hook without the boilerplate of creating a dummy "wrapper" component, the industry standard is to use the **`renderHook`** utility from **React Testing Library**.

_(Note: Prior to React 18, you had to install a separate package called `@testing-library/react-hooks`, but this has since been merged directly into `@testing-library/react`.)_

Here is exactly how to test a custom hook in three common scenarios.

---

### 1. Testing Basic Returns and State Updates

Because React throws an error if you call a hook outside of a component, `renderHook` creates a hidden test component for you behind the scenes.

If your hook returns functions that update state, you **must** wrap the execution of those functions in `act()`. This ensures React finishes processing the state update before your test checks the result.

**The Hook:**

```javascript
import { useState } from "react";

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = () => setCount((c) => c + 1);
  return { count, increment };
}
```

**The Test:**

```javascript
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

test("should initialize and increment the counter", () => {
  // 1. Render the hook
  const { result } = renderHook(() => useCounter(10));

  // 2. Access the returned values via `result.current`
  expect(result.current.count).toBe(10);

  // 3. Wrap any state-changing functions in act()
  act(() => {
    result.current.increment();
  });

  // 4. Assert the new state
  expect(result.current.count).toBe(11);
});
```

### 2. Testing Hooks with Changing Props

Sometimes you need to test how your hook reacts when its input arguments change (e.g., simulating a prop change from a parent component). You can do this using `initialProps` and the `rerender` function provided by `renderHook`.

```javascript
test("should update when the initial value changes", () => {
  const { result, rerender } = renderHook(
    (initialValue) => useCounter(initialValue),
    { initialProps: 10 },
  );

  expect(result.current.count).toBe(10);

  // Re-run the hook with a new argument
  rerender(20);

  // Assuming your hook has a useEffect that syncs the state to the new prop
  expect(result.current.count).toBe(20);
});
```

### 3. Testing Hooks that use Context (Providers)

If your custom hook relies on a Context Provider (like a Theme, Redux store, or React Query client), it will crash if you test it in isolation because the context doesn't exist.

You can solve this by passing a `wrapper` option to `renderHook`.

```javascript
import { renderHook } from "@testing-library/react";
import { ThemeProvider } from "./ThemeProvider";
import { useTheme } from "./useTheme";

test("should return the current theme from context", () => {
  // Create a wrapper component that provides the necessary context
  const wrapper = ({ children }) => (
    <ThemeProvider initialTheme="dark">{children}</ThemeProvider>
  );

  // Pass the wrapper to renderHook
  const { result } = renderHook(() => useTheme(), { wrapper });

  expect(result.current.theme).toBe("dark");
});
```
