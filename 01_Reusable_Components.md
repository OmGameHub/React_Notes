# My Journey to Building Reusable Components in React

![My Journey to Building Reusable Components in React](/images/01_Reusable_Components/thumbnail.jpg)

I still remember my first React.js project, which I worked on with two classmates during a college hackathon five or six years ago. One of the most frustrating parts of that experience was the repetitive process of creating UI components. I had to copy and paste the same button, form, and table code over and over again, tweaking each instance individually.

When I wanted to change the styling of a button, I had to track down every occurrence, leading to redundant work and wasted time. The more the project grew, the harder it became to manage. It felt like I spent more time maintaining the codebase and fixing bugs than building new features.

![Repetitive Coding Frustration](/images/01_Reusable_Components/RepetitiveCodingFrustration.webp)

Over time, I discovered the value of **reusable components**, which not only simplified my workflow but also transformed how I approach code design and project management. In this blog, I want to share my insights into building reusable components in React and how they can save time and effort in your projects.

## What Are Reusable Components?

In React, a **component** is a self-contained, reusable piece of UI. A **reusable component** is a component designed to be generic and adaptable so that it can be used in multiple places across an application with little or no modification.

For example, a **Button** component can be designed to handle a variety of use cases, such as form submissions, navigation, or alerts, by simply passing different properties (props) to it.

## Steps to Build Reusable Components

### 1\. **Identify Common Patterns**

The first step is to identify the common UI patterns in your application. Look for repetitive elements like buttons, forms, tables, modals, or any other component that appears in multiple places. These are prime candidates for refactoring into reusable components.

### 2\. **Design Components with Props**

Props allow you to pass data and configurations to a component, making it flexible for various use cases. This is key to building reusable components.

Here’s an example of a reusable **Button** component:

```typescript
// Button.ts
import React from "react";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
    theme?: "primary" | "secondary"; // Define specific themes
    loading?: boolean; // Optional prop to indicate loading state
}

const Button: React.FC<ButtonProps> = ({
    className,
    children,
    theme = "primary",
    onClick,
    disabled = false,
    loading = false,
    ...rest
}) => (
    <button
        className={`${className} bg-${theme}`} // Apply dynamic styles
        onClick={onClick}
        disabled={disabled || loading}
        {...rest}
    >
        {loading ? "Loading..." : children} {/* Show loader if loading */}
    </button>
);

export default Button;
```

**Props Breakdown:**

-   `theme`: Specifies the button style (e.g., `"primary"` or `"secondary"`).

-   `loading`: A flag indicating whether the button is in a loading state.

-   `...rest`: Allows passing additional props like `id`, `type`, or `aria-label`.

**Usage:**

```typescript
<Button onClick={() => alert("Primary Button clicked")}>
    Primary Button
</Button>

<Button
    theme="secondary"
    onClick={() => alert("Secondary Button clicked")}
>
    Secondary Button
</Button>
```

### 3\. **Use Composition for Flexibility**

Composition is a powerful tool for building flexible and reusable components. Instead of relying solely on props, you can use `children` to pass nested content or even wrap other components.

Here’s an example of a **Card** component that uses composition:

```typescript
// Card.ts
import React from "react";

interface CardProps {
    title: string;
    children: React.ReactNode; // Allows nested content
}

const Card: React.FC<CardProps> = ({ title, children }) => (
    <div className="card">
        <div className="card-header">{title}</div>
        <div className="card-body">{children}</div>
    </div>
);

export default Card;
```

**Usage:**

```typescript
<Card title="My Card">
    <p>This is the content of my card.</p>
</Card>
```

### 4\. **Follow a Consistent Styling Approach**

A consistent styling approach ensures your components are easy to maintain and work seamlessly across the application. Popular options include:

-   **CSS Modules**: Scoped CSS to avoid conflicts.

-   **Styled Components**: CSS-in-JS for dynamic styling.

-   **Tailwind CSS**: Utility-first CSS framework.

Choose the method that aligns with your team’s workflow and project requirements.

### 5\. **Test Your Components**

Testing ensures your components work correctly and prevents regressions. Use tools like **Jest** and **React Testing Library** to write unit tests for your components. Test for various states, props, and edge cases.

## Advanced Concept for Building Reusable Components

Once you've mastered the basics of reusable components, it's time to explore advanced techniques that make your components even more flexible, maintainable, and scalable. Here are some advanced approaches to consider:

### 1\. **Controlled vs. Uncontrolled Components**

Reusable components often need to manage state, such as form inputs or toggles. Choosing between **controlled** (fully managed by parent components) and **uncontrolled** (internally managed) behaviors can make a significant difference in flexibility.

#### Controlled Components

With controlled components, all state is passed via props and managed by the parent. This allows fine-grained control and synchronization across components.

```typescript
interface InputProps {
    value: string;
    onChange: (value: string) => void;
}

const Input: React.FC<InputProps> = ({ value, onChange }) => {
    return <input value={value} onChange={(e) => onChange(e.target.value)} />;
};

// Usage
const [inputValue, setInputValue] = React.useState("");

<Input value={inputValue} onChange={setInputValue} />;
```

#### Uncontrolled Components

Uncontrolled components manage their own state internally. They are easier to use for simple scenarios but less flexible for complex workflows.

```typescript
interface InputProps {
    defaultValue?: string;
}

const Input: React.FC<InputProps> = ({ defaultValue }) => {
    const [value, setValue] = React.useState(defaultValue || "");

    return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// Usage
<Input defaultValue="Hello" />;
```

**Tip:** Provide the ability to switch between controlled and uncontrolled behavior for maximum reusability.

### 2\. **Higher-Order Components (HOCs)**

_HOCs are functions that take a component and return an enhanced version of it._

Use HOCs when you need to add common functionality to multiple components, such as logging, error handling, or data fetching, without repeating logic.

```typescript
function withLogging<T>(WrappedComponent: React.ComponentType<T>) {
    return (props: T) => {
        console.log("Component rendered with props:", props);
        return <WrappedComponent {...props} />;
    };
}

// Example component
const Button = ({ label }: { label: string }) => <button>{label}</button>;

// Wrap the component with logging
const LoggedButton = withLogging(Button);

// Usage
<LoggedButton label="Click Me" />;
```

**Use Cases:** Logging, Authorization and Error Boundaries

### 3\. **Render Props**

_Render props are functions passed as props that define how a component should render its children._

This pattern allows reusable components to provide flexibility for rendering logic.

```typescript
interface DataFetcherProps {
    url: string;
    children: (data: any, loading: boolean, error: any) => React.ReactNode;
}

const DataFetcher: React.FC<DataFetcherProps> = ({ url, children }) => {
    const [data, setData] = React.useState(null);
    const [loading, setLoading] = React.useState(true);
    const [error, setError] = React.useState(null);

    React.useEffect(() => {
        fetch(url)
            .then((res) => res.json())
            .then(setData)
            .catch(setError)
            .finally(() => setLoading(false));
    }, [url]);

    return <>{children(data, loading, error)}</>;
};

// Usage
<DataFetcher url="/api/data">
    {(data, loading, error) => {
        if (loading) return <p>Loading...</p>;
        if (error) return <p>Error: {error.message}</p>;
        return <p>Data: {JSON.stringify(data)}</p>;
    }}
</DataFetcher>;
```

**Use Cases:**

-   Customizing rendering logic for data fetching, forms, or dynamic UIs.

### 4\. **Custom Hooks for Logic Extraction**

To keep your reusable components simple, extract shared logic into **custom hooks**. This separates UI and logic, making both easier to test and reuse.

```typescript
function useCounter(initialValue: number = 0) {
    const [count, setCount] = React.useState(initialValue);
    const increment = () => setCount((prev) => prev + 1);
    const decrement = () => setCount((prev) => prev - 1);

    return { count, increment, decrement };
}

// Component using the custom hook
const Counter = () => {
    const { count, increment, decrement } = useCounter(5);

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>Increment</button>
            <button onClick={decrement}>Decrement</button>
        </div>
    );
};
```

**Benefits:**

-   Logic reuse across multiple components.

-   Clear separation of concerns.

### 5\. **Context for Global State Sharing**

Use React's **Context API** to create reusable components that share state or configuration globally.

```typescript
const ThemeContext = React.createContext("light");

const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({
    children,
}) => {
    const [theme, setTheme] = React.useState("light");
    return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
            {children}
        </ThemeContext.Provider>
    );
};

const Button = () => {
    const { theme } = React.useContext(ThemeContext);
    return <button className={`btn-${theme}`}>Themed Button</button>;
};

// Usage
<ThemeProvider>
    <Button />
</ThemeProvider>;
```

**Use Cases:** Theming, Authentication and Localization

### 6\. **Dynamic Component Rendering**

For highly reusable and flexible UIs, create components that dynamically render based on configuration or data.

```typescript
interface Field {
    type: string;
    name: string;
}

interface DynamicFormProps {
    fields: Field[];
}

const DynamicForm: React.FC<DynamicFormProps> = ({ fields }) => {
    return (
        <form>
            {fields.map((field) => {
                switch (field.type) {
                    case "text":
                        return (
                            <input
                                key={field.name}
                                name={field.name}
                                type="text"
                            />
                        );
                    case "checkbox":
                        return (
                            <input
                                key={field.name}
                                name={field.name}
                                type="checkbox"
                            />
                        );
                    default:
                        return null;
                }
            })}
        </form>
    );
};

// Usage
<DynamicForm
    fields={[
        { type: "text", name: "username" },
        { type: "checkbox", name: "subscribe" },
    ]}
/>;
```

**Benefits:**

-   Minimizes hardcoding of repetitive structures like forms and tables.

-   Facilitates dynamic UIs that can be driven by configurations or APIs.

### 7\. Custom Hook for Managing Asynchronous Transitions

For components that require handling asynchronous transitions (e.g., loading states during network requests), you can create a custom hook to manage the transition state efficiently.

```typescript
import { useState } from "react";

type TransitionCallback = () => Promise<void>;

const useTransition = () => {
    const [isPending, setIsPending] = useState(false);

    const startTransition = async (callback: TransitionCallback) => {
        setIsPending(true);
        try {
            await callback();
        } catch (error) {
            console.error("Error during transition:", error);
        } finally {
            setIsPending(false);
        }
    };

    return [isPending, startTransition] as const;
};

export default useTransition;
```

**Usage:**

```typescript
const MyComponent = () => {
    const [isPending, startTransition] = useTransition();

    const handleClick = () => {
        startTransition(async () => {
            // Simulate a network request or some async operation
            await new Promise((resolve) => setTimeout(resolve, 2000));
            console.log("Transition complete");
        });
    };

    return (
        <div>
            <button onClick={handleClick} disabled={isPending}>
                {isPending ? "Loading..." : "Click Me"}
            </button>
        </div>
    );
};
```

## Best Practices for Reusable Components

1. **Keep Components Small and Focused (Single Responsibility Principle)** Each component should handle a single responsibility. For example, a **Button** should render a button and handle click events, but it shouldn’t manage application state or business logic.

    ```typescript
    // Bad
    const Button = () => {
        const [count, setCount] = React.useState(0);
        return (
            <button onClick={() => setCount((prev) => prev + 1)}>
                Clicked {count} times
            </button>
        );
    };

    // Good
    const Button = ({ onClick, children }) => (
        <button onClick={onClick}>{children}</button>
    );
    ```

2. **Use Clear and Descriptive Prop Names** Choose prop names that clearly convey their purpose. For instance, use `theme` instead of `color` if the prop controls the button’s style.

    ```typescript
    // Bad
    <Button color="primary">Primary Button</Button>;

    // Good
    <Button theme="primary">Primary Button</Button>;
    ```

3. **Provide Default Props (Open/Closed Principle)** Set default values for props to ensure the component works even when certain props are omitted.

    ```typescript
    const Button = ({ theme = "primary", ...rest }) => (
        <button className={`btn-${theme}`} {...rest} />
    );
    ```

4. **Avoid Over-Optimization (Keep It Simple, Stupid)** Don’t try to make your components handle every possible use case. Focus on common scenarios, and allow flexibility for edge cases through props or composition.

5. **Write Documentation** Clear documentation helps other developers understand how to use your components. Include usage examples and explain the available props.

6. **Test Thoroughly** Reusable components are used throughout your application, so ensure they are robust and well-tested.

## Conclusion

Building reusable components in React requires striking a balance between simplicity and flexibility. By focusing on component isolation, prop-driven customization, and efficient composition, you can create a library of components that speeds up development, ensures consistency, and simplifies project maintenance.

What reusable components have you found most valuable in your projects? Let me know in the comments below—I’d love to hear your experiences!

Thanks for reading! 🚀
