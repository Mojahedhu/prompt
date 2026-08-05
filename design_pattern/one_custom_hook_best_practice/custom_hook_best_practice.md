Writing custom hooks is easy; writing _good_ custom hooks that don't cause infinite loops or memory leaks is difficult.

Because custom hooks are just JavaScript functions that can call other React hooks, they are bound by React's rendering lifecycle. If you get the API design or the dependencies wrong, you will infect every component that uses your hook.

Here are the best practices and the most dangerous pitfalls when authoring custom hooks.

---

### Best Practices for API Design

#### 1. Choose the Right Return Type (Array vs. Object)

How you return data from your hook determines how easy it is to use.

- **Return an Array (`[value, setter]`)** when your hook manages a single piece of state. This matches `useState` and allows the consumer to easily rename the variables.

```jsx
// Good for single-value states
const [isHovered, hoverRef] = useHover();
const [isDark, toggleTheme] = useDarkMode();
```

- **Return an Object (`{ data, error, isLoading }`)** when your hook returns multiple related values. If you return an array with more than two items, consumers have to memorize the exact index order, which leads to bugs.

```jsx
// Good for complex states
const { data, refetch, isFetching } = useUser(userId);
```

#### 2. Keep the Hook's Footprint Small (Single Responsibility)

A custom hook should do exactly one thing. Do not build "God Hooks" that manage the user's profile, their shopping cart, and the window size all at once. If your hook is longer than 50 lines, you should probably split it into two smaller hooks.

#### 3. Accept Configuration Objects, Not Endless Arguments

If your hook takes more than two arguments, pass a configuration object. This makes the hook self-documenting and future-proofs it if you need to add more options later.

```jsx
// Bad: What do these booleans mean?
useFetch("/users", true, false, 5000);

// Good: Self-documenting
useFetch("/users", {
  prefetch: true,
  suspense: false,
  timeout: 5000,
});
```

---

### Common Pitfalls (And How to Avoid Them)

#### 1. Sweeping Bad `useEffect` Logic Under the Rug

The most common mistake developers make is creating a custom hook simply to hide a massive, messy `useEffect`.

Moving a broken `useEffect` into a different file doesn't fix the race conditions, missing dependencies, or memory leaks; it just makes them harder to find. Before you extract an effect into a hook, ensure the logic itself is flawless. (Or better yet, use a library like React Query for data fetching instead of writing your own `useFetch` hook).

#### 2. Forgetting to Memoize Returned Functions (with presence of react compiler, this is not needed)

If your custom hook returns a function, and the consuming component passes that function into its own `useEffect` or to a memoized child component, you **must** wrap the function in `useCallback`.

If you don't, your hook will create a brand new function reference on every render, triggering infinite loops in the consuming component.

```jsx
// ❌ BAD: Returns a new function reference every render
function useCounter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount((c) => c + 1);
  return { count, increment };
}

// ✅ GOOD: Function reference remains stable
function useStableCounter() {
  const [count, setCount] = useState(0);
  const increment = useCallback(() => setCount((c) => c + 1), []);
  return { count, increment };
}
```

#### 3. State Desynchronization (The Derived State Trap)

Don't use local state inside a hook if the value can be calculated on the fly.

**The Trap:**

```jsx
function useFilteredUsers(users, query) {
  const [filtered, setFiltered] = useState([]);

  // ❌ BAD: Storing derived state in a useEffect
  useEffect(() => {
    setFiltered(users.filter((u) => u.name.includes(query)));
  }, [users, query]);

  return filtered;
}
```

This causes a "double render." React renders once when `query` changes, runs the effect, updates `filtered`, and renders _again_.

**The Fix:**
Just calculate it during render. If the calculation is expensive, wrap it in `useMemo`.

```jsx
function useFilteredUsers(users, query) {
  // ✅ GOOD: Derived during render, memoized for performance
  return useMemo(() => {
    return users.filter((u) => u.name.includes(query));
  }, [users, query]);
}
```

#### 4. Not Following the Rules of Hooks

Because custom hooks are abstracted, it's easy to accidentally violate React's golden rule: **Hooks must be called unconditionally.**

Never place a custom hook inside an `if` statement, a `for` loop, or a nested function. The React linter (`eslint-plugin-react-hooks`) will catch this, but only if you strictly follow the convention of naming your hook with the `use` prefix (e.g., `useAuth`, not `getAuth`).
