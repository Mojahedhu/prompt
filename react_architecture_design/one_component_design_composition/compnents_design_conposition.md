Component design is all about how you slice up your application into reusable, predictable, and testable pieces. When developers talk about "good React code," they are usually talking about mastering these specific concepts.

Here is a deep dive into how to design and compose React components effectively.

---

### 1. Separation of Concerns: Logic vs. Presentation

Early React patterns split components into "Smart" (Containers that fetch data) and "Dumb" (Presentational components that just render UI). While the concept remains, the modern approach uses **Custom Hooks** to achieve this separation.

A well-designed component should not know _how_ to fetch data, parse it, or handle complex state mutations. It should only care about rendering.

- **The Logic Layer:** You extract your `useEffects`, data fetching, and complex `useStates` into a custom hook (e.g., `useUserProfile`).
- **The UI Layer:** Your component simply calls that hook, receives the necessary data and functions, and renders the JSX.

This means if your UI changes (say, switching from a table to a grid view), your logic remains untouched. If your API endpoint changes, your UI component doesn't care.

### 2. Component Composition (The "Lego" Model)

Composition is React’s most powerful feature. It is the practice of building complex components from simpler ones, primarily using the `children` prop.

The biggest mistake junior developers make is **prop drilling**—passing a piece of data down through five layers of components just because the bottom component needs it.

Instead of passing _data_ down, you pass _components_ down.

**Instead of this (Prop Drilling):**

```jsx
<Dashboard user={user} theme={theme} showSidebar={true} />
```

**You do this (Composition):**

```jsx
<Dashboard>
  <Sidebar theme={theme} />
  <MainContent user={user} />
</Dashboard>
```

Now, `Dashboard` doesn't need to know anything about `user` or `theme`. It just renders `{children}`.

#### The "Slots" Pattern

When `children` isn't enough (e.g., you need to inject components into a header _and_ a footer), you can pass components as named props, often called "slots":

```jsx
<PageLayout topNav={<Navbar />} sidebar={<FilterMenu />}>
  <ProductGrid />
</PageLayout>
```

### 3. Avoiding the "God Component"

As applications grow, components often accumulate props until they become unmaintainable "God Components."

A button should not look like this:

```jsx
<Button
  isPrimary
  isSecondary
  isDanger
  isSubmit
  hasIcon
  isLoading
  size="large"
/>
```

Good component design enforces the **Single Responsibility Principle**. If a component has too many boolean flags changing its internal logic, it’s doing too much.

- **Use Variants:** Instead of 10 boolean flags, accept a single `variant="danger"` or `size="large"` prop.
- **Split them up:** If a component handles too many wildly different edge cases, break it into `<PrimaryButton>` and `<IconButton>`.

### 4. Advanced Composition: Compound Components

When multiple components must work together to share implicit state (like a `<select>` and `<option>` tag in HTML), modern React architecture uses the **Compound Component** pattern.

This is highly popular in UI libraries like Radix or Headless UI. It allows you to build highly flexible components that share state internally via Context, without passing props manually:

```jsx
<Menu>
  <Menu.Trigger>Open Options</Menu.Trigger>
  <Menu.Dropdown>
    <Menu.Item>Profile</Menu.Item>
    <Menu.Item>Settings</Menu.Item>
    <Menu.Item>Logout</Menu.Item>
  </Menu.Dropdown>
</Menu>
```

Here, the user of the `Menu` has complete control over the layout and ordering of the items, while the internal logic (like knowing when the dropdown is open) is handled secretly behind the scenes.

### 5. Keep boundaries & state clear (the missing piece):

Define clear prop contracts, isolate error/loading states so one bug does not crash the screen, and keep state close to where it is actually used.
