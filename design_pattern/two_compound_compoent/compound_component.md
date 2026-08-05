Building a Compound Component is all about creating a hidden communication channel between a parent and its children using React Context.

Instead of building one massive `<Dropdown>` component that accepts a dozen props for configuration, we are going to build four small components: `<Dropdown>`, `<Trigger>`, `<Menu>`, and `<Item>`.

Here is how to build it from scratch in four steps.

---

### Step 1: The Context (The Messenger)

First, we need a way for the parent to secretly pass state down to the children without using props. We do this by creating a Context and a custom hook to consume it.

```jsx
import { createContext, useContext, useState } from "react";

const DropdownContext = createContext(null);

// Custom hook to consume the context securely
function useDropdownContext() {
  const context = useContext(DropdownContext);
  if (!context) {
    throw new Error(
      "Dropdown child components must be rendered within a <Dropdown> parent.",
    );
  }
  return context;
}
```

> **Key insight:** Throwing an error in the custom hook is a crucial best practice. It immediately warns developers if they accidentally render a `<Dropdown.Item>` outside of the main `<Dropdown>` wrapper.

### Step 2: The Parent (The State Manager)

The parent component doesn't care what the UI looks like. Its only job is to hold the `isOpen` state and provide it to any child components that ask for it.

```jsx
export function Dropdown({ children }) {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen((prev) => !prev);
  const close = () => setIsOpen(false);

  return (
    <DropdownContext.Provider value={{ isOpen, toggle, close }}>
      <div className="relative inline-block text-left">{children}</div>
    </DropdownContext.Provider>
  );
}
```

### Step 3: The Children (The UI Consumers)

Now we build the smaller pieces. Each piece reaches up into the Context to grab exactly what it needs to function.

- The **Trigger** needs the `toggle` function to open/close the menu.
- The **Menu** needs the `isOpen` boolean to know if it should render.
- The **Item** needs the `close` function so the menu hides automatically after a user makes a selection.

```jsx
function Trigger({ children }) {
  const { toggle } = useDropdownContext();
  return (
    <button
      onClick={toggle}
      className="px-4 py-2 bg-blue-600 text-white rounded"
    >
      {children}
    </button>
  );
}

function Menu({ children }) {
  const { isOpen } = useDropdownContext();

  // If the dropdown is closed, don't render the menu at all
  if (!isOpen) return null;

  return (
    <div className="absolute right-0 mt-2 w-48 bg-white border rounded shadow-lg">
      {children}
    </div>
  );
}

function Item({ children, onClick }) {
  const { close } = useDropdownContext();

  const handleClick = () => {
    if (onClick) onClick(); // Run whatever action the user passed in
    close(); // Auto-close the dropdown
  };

  return (
    <div
      onClick={handleClick}
      className="px-4 py-2 hover:bg-gray-100 cursor-pointer"
    >
      {children}
    </div>
  );
}
```

### Step 4: The Dot Notation (The Clean API)

To make the developer experience seamless, we attach the child components directly to the parent function. This isn't strictly required by React, but it's an industry standard that visually groups the components together.

```jsx
Dropdown.Trigger = Trigger;
Dropdown.Menu = Menu;
Dropdown.Item = Item;
```

---

### The Final Result

Now, look at how beautiful and readable the usage becomes in your actual application. You have complete control over the order of elements and the content inside them, and you didn't have to pass a single `isOpen` prop.

```jsx
export default function App() {
  return (
    <Dropdown>
      <Dropdown.Trigger>User Settings</Dropdown.Trigger>

      <Dropdown.Menu>
        <Dropdown.Item onClick={() => console.log("Routing to profile")}>
          My Profile
        </Dropdown.Item>
        <Dropdown.Item onClick={() => console.log("Opening billing")}>
          Billing
        </Dropdown.Item>
        {/* We can easily inject a divider here without altering the Dropdown logic! */}
        <hr />
        <Dropdown.Item onClick={() => console.log("Logging out")}>
          Logout
        </Dropdown.Item>
      </Dropdown.Menu>
    </Dropdown>
  );
}
```
