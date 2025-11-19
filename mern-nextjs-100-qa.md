# MERN Stack / Next.js Internship Interview: 100 Questions & Answers
## AariyaTech Corp Private Limited

---

## SECTION 1: JAVASCRIPT & TYPESCRIPT FUNDAMENTALS (15 Questions)

### Q1: Explain the difference between `var`, `let`, and `const` in JavaScript. Which should you prefer and why?

**Answer:** 
- **var**: Function-scoped, can be re-declared and updated, hoisted with `undefined`
- **let**: Block-scoped, cannot be re-declared in same scope, can be updated, hoisted but not initialized (Temporal Dead Zone)
- **const**: Block-scoped, cannot be re-declared or updated, hoisted but not initialized

**Preference**: Use `const` by default (immutability), `let` when reassignment is needed, avoid `var`. This follows modern best practices and prevents accidental modifications.

---

### Q2: What is hoisting in JavaScript and how does it work?

**Answer:** Hoisting is JavaScript's behavior of moving declarations to the top of their scope before code execution.

- **var declarations**: Hoisted and initialized with `undefined`
- **function declarations**: Fully hoisted (can be called before declaration)
- **let/const**: Hoisted but not initialized (Temporal Dead Zone from start until declaration)

**Example:**
```javascript
console.log(x); // undefined (not ReferenceError)
var x = 5;

foo(); // Works fine
function foo() { console.log('hoisted'); }

console.log(y); // ReferenceError: y is not defined
let y = 10;
```

---

### Q3: Explain closures and provide a practical use case relevant to React.

**Answer:** A closure is a function that has access to variables from another function's scope even after that function has returned.

**Practical React use case - Custom Hook:**
```javascript
function useCounter() {
  const [count, setCount] = useState(0);
  
  const increment = () => setCount(count + 1); // closure over count
  return { count, increment };
}
```

The `increment` function closes over the `count` state, creating a closure that remembers and updates that state.

---

### Q4: What are arrow functions and what are their key differences from regular functions?

**Answer:**
- **Syntax**: `() => {}` vs `function() {}`
- **No `this` binding**: Arrow functions inherit `this` from parent scope (useful in React class components)
- **No `arguments` object**: Use rest parameters `(...args)` instead
- **Cannot be used as constructors**: No `new` keyword with arrow functions
- **No `prototype` property**

**React example:**
```javascript
// Better for React event handlers
const handleClick = () => { // this refers to component
  this.setState({ clicked: true });
};
```

---

### Q5: Explain `this` keyword and its binding rules in JavaScript.

**Answer:** `this` refers to the object that is executing the current code.

**Binding rules (priority order):**
1. **new binding**: `new Constructor()` - `this` is the new object
2. **explicit binding**: `call()`, `apply()`, `bind()` - `this` is passed explicitly
3. **implicit binding**: `obj.method()` - `this` is the object before the dot
4. **default binding**: `function()` - `this` is global (undefined in strict mode)
5. **arrow functions**: `this` is lexically scoped (inherited from parent)

**React context**: Always bind methods or use arrow functions to preserve `this`.

---

### Q6: What is the difference between `==` and `===`? Provide examples.

**Answer:**
- **`==`**: Loose equality, performs type coercion before comparison
- **`===`**: Strict equality, no type coercion, must match type and value

**Examples:**
```javascript
5 == "5"    // true (type coercion)
5 === "5"   // false (different types)
null == undefined // true
null === undefined // false
0 == false  // true
0 === false // false
```

**Best practice**: Always use `===` to avoid unexpected type coercion bugs, especially in production code.

---

### Q7: Explain async/await and how it improves upon Promises.

**Answer:** Async/await is syntactic sugar over Promises, making asynchronous code look synchronous and easier to read.

**Key benefits:**
- Cleaner syntax without `.then()` chains
- Easier error handling with `try/catch`
- Better readability and debugging

**Comparison:**
```javascript
// Promises
fetch('/api/data')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Async/await
async function getData() {
  try {
    const res = await fetch('/api/data');
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

**In Node.js/Next.js**: Used extensively for API routes and server-side operations.

---

### Q8: What are destructuring and spread operators? Provide practical examples.

**Answer:**
- **Destructuring**: Extract values from arrays/objects into distinct variables
- **Spread operator**: Spread array elements or object properties

**Array destructuring:**
```javascript
const [a, b, ...rest] = [1, 2, 3, 4];
// a = 1, b = 2, rest = [3, 4]
```

**Object destructuring:**
```javascript
const { name, age } = { name: 'John', age: 30 };
const { name: personName } = user; // rename
```

**Spread operator:**
```javascript
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4]; // [1, 2, 3, 4]

const obj1 = { a: 1 };
const obj2 = { ...obj1, b: 2 }; // { a: 1, b: 2 }
```

**React usage**: Destructuring props, spreading state updates, passing props to child components.

---

### Q9: Explain higher-order functions and their use in functional programming.

**Answer:** A higher-order function is a function that takes another function as an argument or returns a function.

**Example:**
```javascript
// Function that takes a function as argument
function withLogger(fn) {
  return (...args) => {
    console.log('Function called with:', args);
    return fn(...args);
  };
}

const add = (a, b) => a + b;
const addWithLog = withLogger(add);
addWithLog(2, 3); // logs args, returns 5
```

**React examples:**
- Higher-Order Components (HOCs)
- Custom hooks (which are functions returning functions)
- Array methods: `map()`, `filter()`, `reduce()`

---

### Q10: What are TypeScript interfaces and types? How do they differ?

**Answer:**
- **Interface**: Defines contract for object shape, can be extended, merges with same name
- **Type**: Defines any type using type alias, doesn't merge, more flexible

**Comparison:**
```typescript
// Interface
interface User {
  name: string;
  age: number;
}

interface Admin extends User {
  permissions: string[];
}

// Type
type UserType = {
  name: string;
  age: number;
};

type AdminType = UserType & {
  permissions: string[];
};
```

**When to use:**
- **Interfaces**: For OOP, defining contracts (especially with classes)
- **Types**: For unions, tuples, function signatures, general type aliases

**Best practice in MERN**: Use interfaces for API responses and database schemas; use types for unions and complex type logic.

---

### Q11: Explain generics in TypeScript with an example relevant to API responses.

**Answer:** Generics allow writing reusable code that works with any type while maintaining type safety.

**API Response Example:**
```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

interface User {
  id: number;
  name: string;
}

const userResponse: ApiResponse<User> = {
  success: true,
  data: { id: 1, name: 'John' }
};

// Works with any type
const productsResponse: ApiResponse<Product[]> = {
  success: true,
  data: [...]
};

// Generic function
async function fetchData<T>(url: string): Promise<ApiResponse<T>> {
  const res = await fetch(url);
  return res.json();
}

const users = await fetchData<User[]>('/api/users');
```

**Benefits**: Type safety, code reusability, IDE autocomplete for API responses.

---

### Q12: What is the difference between `null` and `undefined` in JavaScript?

**Answer:**
- **`undefined`**: Default value for uninitialized variables, missing function parameters, functions with no return
- **`null`**: Represents intentional absence of value, must be explicitly assigned

**Examples:**
```javascript
let x; // undefined
const y = null; // null

function test(param) {
  console.log(param); // undefined if not passed
}

typeof undefined // "undefined"
typeof null // "object" (quirk in JS)

undefined == null // true
undefined === null // false
```

**In Node.js/Next.js**: Handle both when parsing API responses and environment variables.

---

### Q13: Explain the concept of immutability and why it matters in React.

**Answer:** Immutability means not modifying data directly but creating new copies with changes.

**Why it matters:**
- React uses reference equality to detect changes
- Enables time-travel debugging
- Prevents accidental mutations
- Easier to track state changes

**Bad (mutating):**
```javascript
const user = { name: 'John', age: 30 };
user.age = 31; // Direct mutation
setUser(user); // React may not detect change
```

**Good (immutable):**
```javascript
const user = { name: 'John', age: 30 };
setUser({ ...user, age: 31 }); // New object created
```

**With arrays:**
```javascript
// Bad
const arr = [1, 2, 3];
arr.push(4);
setItems(arr);

// Good
const arr = [1, 2, 3];
setItems([...arr, 4]);
```

---

### Q14: What are template literals and when would you use them in Node.js?

**Answer:** Template literals use backticks (`) and allow embedded expressions with `${variable}`.

**Benefits:**
- Multi-line strings without concatenation
- Embedded expressions
- Cleaner code

**Examples:**
```javascript
const name = 'John';
const age = 30;

// Template literal
console.log(`Name: ${name}, Age: ${age}`);

// Useful in Node.js/Express
app.get('/users/:id', (req, res) => {
  const message = `User ID: ${req.params.id} created at ${new Date()}`;
});

// Multi-line strings
const query = `
  SELECT * FROM users
  WHERE age > ${minAge}
  AND status = 'active'
`;

// Tagged templates (advanced)
function highlight(strings, ...values) {
  return strings
    .map((str, i) => str + (values[i] ? `[${values[i]}]` : ''))
    .join('');
}
```

**In production**: Used extensively in SQL queries, API endpoints, and error messages.

---

### Q15: Explain the concept of pure functions and their importance in functional programming.

**Answer:** A pure function always returns the same output for the same input and has no side effects (doesn't modify external state).

**Characteristics:**
- Deterministic: Same input → Same output
- No side effects: Doesn't modify global state or cause I/O
- Easier to test and debug

**Pure vs Impure:**
```javascript
// Impure
let globalCounter = 0;
function impureAdd(a, b) {
  globalCounter++; // modifies external state
  return a + b;
}

// Pure
function pureAdd(a, b) {
  return a + b;
}

// React hook - pure
function useSum(a, b) {
  return useMemo(() => a + b, [a, b]);
}
```

**In React:** Render functions should be pure to avoid unexpected behavior and enable proper reconciliation.

---

## SECTION 2: REACT FUNDAMENTALS (20 Questions)

### Q16: What is React and what problem does it solve?

**Answer:** React is a JavaScript library for building user interfaces with reusable components using a declarative approach.

**Problems it solves:**
- **State management complexity**: Manages UI state efficiently
- **Reusability**: Component-based architecture enables reuse
- **Performance**: Virtual DOM and diffing optimize rendering
- **Maintainability**: Declarative syntax easier to understand and maintain

**Key concepts:**
- Component-based architecture
- Unidirectional data flow
- Virtual DOM for performance
- Reactive updates when state changes

**In AariyaTech projects**: Used as the foundation for responsive, scalable web applications across EdTech, HRTech, FinTech platforms.

---

### Q17: Explain the difference between class components and functional components. When would you use each?

**Answer:**
- **Class components**: ES6 classes, have lifecycle methods, state via `this.state`
- **Functional components**: JavaScript functions, use hooks for state and effects, simpler syntax

**Modern approach**: Functional components with hooks are now preferred.

**Comparison:**
```javascript
// Class component
class UserProfile extends React.Component {
  state = { user: null };
  
  componentDidMount() {
    fetchUser();
  }
  
  render() {
    return <div>{this.state.user?.name}</div>;
  }
}

// Functional component (modern)
function UserProfile() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser();
  }, []);
  
  return <div>{user?.name}</div>;
}
```

**When to use each:**
- **Functional**: Default choice in modern React (with hooks)
- **Class**: Legacy code, rare edge cases requiring error boundaries

---

### Q18: What are React hooks? Explain useState and useEffect with examples.

**Answer:** Hooks allow functional components to use state and other React features.

**useState:**
```javascript
const [count, setCount] = useState(0);

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**useEffect:**
```javascript
useEffect(() => {
  // Effect runs after render
  
  return () => {
    // Cleanup function (optional)
  };
}, [dependencies]); // dependency array

// Practical example
useEffect(() => {
  const fetchData = async () => {
    const response = await fetch('/api/data');
    setData(await response.json());
  };
  
  fetchData();
  
  return () => {
    // Cleanup
  };
}, []); // runs once on mount
```

**Common patterns:**
- Empty dependency array: Run once on mount
- No dependency array: Run after every render
- Dependency array with values: Run when dependencies change

---

### Q19: What is the Virtual DOM and how does React use it for performance?

**Answer:** The Virtual DOM is React's in-memory representation of the real DOM. React uses it to optimize DOM updates.

**How it works:**
1. React renders components to Virtual DOM
2. React diffs (compares) new Virtual DOM with previous version
3. Only changed elements update in real DOM (reconciliation)
4. This is faster than direct DOM manipulation

**Benefits:**
- Batch updates reduce reflows
- Fewer expensive DOM operations
- Better performance with frequent updates

**Example:**
```javascript
// React efficiently updates only the changed text
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    return () => clearInterval(interval);
  }, []);
  
  return (
    <div>
      <h1>Time: {seconds}</h1> {/* Only this updates */}
      <p>Component</p> {/* Not re-rendered */}
    </div>
  );
}
```

---

### Q20: Explain prop drilling and how Context API solves it.

**Answer:** Prop drilling is passing data through multiple levels of components that don't need it, just to pass it deeper.

**Problem:**
```javascript
// Prop drilling - passing theme through every component
function App() {
  return <ComponentA theme="dark" />;
}

function ComponentA({ theme }) {
  return <ComponentB theme={theme} />;
}

function ComponentB({ theme }) {
  return <ComponentC theme={theme} />;
}

function ComponentC({ theme }) {
  return <div style={{ background: theme }}>Content</div>;
}
```

**Solution with Context API:**
```javascript
const ThemeContext = React.createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ComponentA />
    </ThemeContext.Provider>
  );
}

function ComponentC() {
  const theme = useContext(ThemeContext);
  return <div style={{ background: theme }}>Content</div>;
}
```

**For AariyaTech projects**: Use Context for theme, authentication, user preferences to avoid prop drilling in large applications.

---

### Q21: What is state management in React? Compare local state, Context API, and Redux.

**Answer:**

| Aspect | Local State | Context API | Redux |
|--------|------------|-------------|-------|
| Complexity | Simple | Medium | Complex |
| Use case | Component-specific | App-wide data | Complex state logic |
| Performance | Good | May cause re-renders | Optimized |
| Boilerplate | None | Minimal | Significant |
| DevTools | None | Limited | Excellent |
| Time-travel debugging | No | No | Yes |

**When to use:**
```javascript
// Local state - simple toggles
const [isOpen, setIsOpen] = useState(false);

// Context - theme, auth, language
const AuthContext = createContext();

// Redux - complex app state with many interactions
// Consider for AariyaTech's AI-integrated applications
```

**For internship**: Start with local state and Context API; Redux for larger projects if needed.

---

### Q22: How do you handle forms in React? Compare controlled vs uncontrolled components.

**Answer:**

**Controlled component** - React state manages input value:
```javascript
function FormControlled() {
  const [email, setEmail] = useState('');
  
  return (
    <input 
      value={email} 
      onChange={(e) => setEmail(e.target.value)} 
    />
  );
}
```

**Uncontrolled component** - DOM manages input value:
```javascript
function FormUncontrolled() {
  const emailRef = useRef();
  
  const handleSubmit = () => {
    console.log(emailRef.current.value);
  };
  
  return (
    <>
      <input ref={emailRef} />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

**Best practice**: Use controlled components for most cases (easier validation, sync with state, better UX).

**Form library**: Use `react-hook-form` or `Formik` for complex forms in production.

---

### Q23: Explain React keys and why they are important in lists.

**Answer:** Keys help React identify which items have changed, been added, or removed.

**Why keys matter:**
- Maintains component state correctly in lists
- Improves performance by preventing re-rendering of unchanged items
- Without keys, reordering can cause state/input issues

**Bad (no key or index as key):**
```javascript
{todos.map((todo, index) => (
  <TodoItem key={index} todo={todo} /> // Bad if list can reorder
))}
```

**Good:**
```javascript
{todos.map((todo) => (
  <TodoItem key={todo.id} todo={todo} /> // Unique ID
))}
```

**Problem with index keys:**
```javascript
// If item at index 0 is deleted, index keys cause issues
const [items, setItems] = useState([
  { id: 1, text: 'A' },
  { id: 2, text: 'B' }
]);

// Use id, not index!
items.map(item => <div key={item.id}>{item.text}</div>)
```

---

### Q24: What is React.memo and when should you use it?

**Answer:** `React.memo` is a higher-order component that memoizes a component to prevent re-renders if props haven't changed.

**Usage:**
```javascript
const MyComponent = React.memo(function MyComponent(props) {
  return <div>{props.value}</div>;
});

// Component only re-renders if 'value' prop changes
```

**When to use:**
- Component has expensive render logic
- Receives same props frequently
- Parent re-renders often
- Props are stable (not created inline)

**Common mistake:**
```javascript
// Bad - function recreated every render
<MemoComponent onClick={() => handleClick()} />

// Good - pass stable function reference
const handleClick = useCallback(() => { ... }, []);
<MemoComponent onClick={handleClick} />
```

---

### Q25: Explain useCallback and useMemo hooks and their performance implications.

**Answer:** Both hooks optimize performance by memoizing values.

**useCallback** - Memoizes function:
```javascript
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// Same function reference unless a or b changes
```

**useMemo** - Memoizes computed value:
```javascript
const memoizedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);

// Value recalculated only when dependencies change
```

**When to use:**
- Expensive computations
- Passing callbacks to memoized components
- Optimization bottlenecks (use React DevTools Profiler first)

**Warning**: Over-memoization can hurt performance. Memoization itself has a cost.

**Best practice**: Don't optimize prematurely; measure first.

---

### Q26: What is the fragment (`<>`) syntax and when would you use it?

**Answer:** Fragments allow grouping multiple elements without adding extra DOM nodes.

**Usage:**
```javascript
// Without fragment - adds extra div to DOM
function Component() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

// With fragment - no extra div
function Component() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}

// Long form
function Component() {
  return (
    <React.Fragment>
      <h1>Title</h1>
      <p>Content</p>
    </React.Fragment>
  );
}

// With key (when mapping)
{items.map(item => (
  <React.Fragment key={item.id}>
    <h2>{item.title}</h2>
    <p>{item.content}</p>
  </React.Fragment>
))}
```

**Benefits:**
- Cleaner DOM structure
- No unnecessary wrapper elements
- Better CSS Grid/Flex layouts

---

### Q27: Explain lazy loading (code splitting) in React using React.lazy and Suspense.

**Answer:** Lazy loading delays loading components until they're needed, improving initial load time.

**Usage:**
```javascript
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**With React Router:**
```javascript
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/analytics" element={<Analytics />} />
      </Routes>
    </Suspense>
  );
}
```

**Benefits:**
- Smaller initial bundle size
- Faster initial page load
- Load components on demand

**For AariyaTech projects**: Essential for AI-integrated applications with multiple feature modules.

---

### Q28: What are error boundaries in React and how do you implement them?

**Answer:** Error boundaries catch JavaScript errors anywhere in the component tree and prevent white-screen crashes.

**Implementation:**
```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return <div>Something went wrong: {this.state.error.message}</div>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

**What error boundaries catch:**
- Render errors
- Lifecycle method errors
- Constructor errors

**What they don't catch:**
- Event handler errors (use try/catch)
- Async code (setTimeout, promises)
- Server-side rendering errors

---

### Q29: Explain the component lifecycle in React hooks vs class components.

**Answer:**

**Class component lifecycle:**
```javascript
componentDidMount() { } // After mount
componentDidUpdate() { } // After any update
componentWillUnmount() { } // Before unmount
```

**Functional component with hooks:**
```javascript
// Mount
useEffect(() => {
  return () => { }; // cleanup on unmount
}, []);

// Update
useEffect(() => {
  return () => { };
}, [dependencies]);

// Unmount - handled in cleanup function above
```

**Mapping:**
| Class | Hooks |
|-------|-------|
| componentDidMount | useEffect(() => {}, []) |
| componentDidUpdate | useEffect(() => {}, [deps]) |
| componentWillUnmount | useEffect return function |

**For AariyaTech projects**: Use hooks exclusively for new code due to simpler syntax and better code organization.

---

### Q30: What are custom hooks? How do you create and use them?

**Answer:** Custom hooks are JavaScript functions that use other hooks to extract component logic into reusable functions.

**Creating a custom hook:**
```javascript
function useFormInput(initialValue = '') {
  const [value, setValue] = useState(initialValue);

  return {
    value,
    setValue,
    bind: {
      value,
      onChange: e => setValue(e.target.value)
    }
  };
}

// Usage
function LoginForm() {
  const email = useFormInput('');
  const password = useFormInput('');

  return (
    <form>
      <input {...email.bind} placeholder="Email" />
      <input {...password.bind} placeholder="Password" type="password" />
    </form>
  );
}
```

**Another example - useFetch:**
```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data: users, loading, error } = useFetch('/api/users');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return users.map(user => <div key={user.id}>{user.name}</div>);
}
```

**Benefits:**
- Reusable logic across components
- Cleaner code
- Easier testing

---

### Q31: Explain conditional rendering in React with different approaches.

**Answer:**

**if/else:**
```javascript
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <div>Welcome back!</div>;
  }
  return <div>Please log in</div>;
}
```

**Ternary operator:**
```javascript
{isLoggedIn ? <Dashboard /> : <LoginForm />}
```

**Logical AND (&&):**
```javascript
{isLoggedIn && <Dashboard />}
```

**Switch statement:**
```javascript
function UserStatus({ status }) {
  switch(status) {
    case 'active':
      return <span>Active</span>;
    case 'inactive':
      return <span>Inactive</span>;
    default:
      return <span>Unknown</span>;
  }
}
```

**Ternary in JSX (best practice for complex conditions):**
```javascript
<div>
  {user?.isAdmin ? (
    <AdminPanel />
  ) : user?.isLoggedIn ? (
    <UserDashboard />
  ) : (
    <LoginForm />
  )}
</div>
```

**For AariyaTech projects**: Use simple ternary or && for basic conditions; consider separate components for complex conditional logic.

---

### Q32: How do you handle events in React? What are synthetic events?

**Answer:** React wraps native browser events in a cross-browser compatible `SyntheticEvent` object.

**Event handling:**
```javascript
function Button() {
  const handleClick = (e) => {
    e.preventDefault(); // prevent default behavior
    console.log(e.type); // "click"
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

**Synthetic events:**
- React pools synthetic events for performance
- Automatically stops propagation at the root
- Event object is nullified after callback (avoid storing references)

**Common events:**
```javascript
<input onChange={(e) => setValue(e.target.value)} />
<button onClick={handleClick} />
<form onSubmit={handleSubmit} />
<input onFocus={handleFocus} onBlur={handleBlur} />
<div onMouseEnter={handleEnter} onMouseLeave={handleLeave} />
```

**Common pitfall:**
```javascript
// Wrong - loses 'this' context
<button onClick={this.handleClick}>Click</button>

// Right
<button onClick={() => this.handleClick()}>Click</button>
// OR in constructor: this.handleClick = this.handleClick.bind(this);
```

---

### Q33: Explain the concept of lifting state up in React.

**Answer:** Lifting state up means moving state to the nearest common parent component to share it between sibling components.

**Problem - State in separate components:**
```javascript
function Celsius() {
  const [tempC, setTempC] = useState('');
  return <input value={tempC} onChange={(e) => setTempC(e.target.value)} />;
}

function Fahrenheit() {
  const [tempF, setTempF] = useState('');
  return <input value={tempF} onChange={(e) => setTempF(e.target.value)} />;
}

function App() {
  return (
    <div>
      <Celsius /> {/* Can't sync with Fahrenheit */}
      <Fahrenheit />
    </div>
  );
}
```

**Solution - Lift state to parent:**
```javascript
function App() {
  const [tempC, setTempC] = useState('');

  const tempF = (tempC * 9/5) + 32;

  return (
    <div>
      <Celsius temp={tempC} onChange={setTempC} />
      <Fahrenheit temp={tempF} />
    </div>
  );
}

function Celsius({ temp, onChange }) {
  return <input value={temp} onChange={(e) => onChange(e.target.value)} />;
}

function Fahrenheit({ temp }) {
  return <input value={temp} disabled />;
}
```

**When to do this:**
- Multiple components need same state
- Sibling communication needed
- State affects multiple components

---

### Q34: What is the difference between controlled and uncontrolled components in React forms?

**Answer:** (Already covered in Q22, but providing additional context for completeness)

**Key difference:**
- **Controlled**: React state is the single source of truth
- **Uncontrolled**: DOM is the source of truth (like traditional HTML)

**When to use each:**
```javascript
// Controlled - for complex forms, validation
function SearchForm() {
  const [query, setQuery] = useState('');
  
  return (
    <input 
      value={query}
      onChange={(e) => setQuery(e.target.value)}
    />
  );
}

// Uncontrolled - simple cases, integration with non-React code
function SearchForm() {
  const inputRef = useRef();
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>Search</button>
    </>
  );
}
```

---

### Q35: How do you optimize React performance? List key techniques.

**Answer:**

**1. Code splitting with React.lazy:**
```javascript
const Dashboard = lazy(() => import('./Dashboard'));
```

**2. Memoization:**
```javascript
const MemoComponent = memo(Component);
```

**3. useCallback for stable function references:**
```javascript
const handleClick = useCallback(() => { ... }, []);
```

**4. useMemo for expensive calculations:**
```javascript
const result = useMemo(() => expensiveCalc(a, b), [a, b]);
```

**5. Remove unused code:**
- Tree shaking in bundlers
- Dynamic imports for large libraries

**6. Image optimization:**
- Use modern formats (WebP)
- Lazy load images

**7. Virtual scrolling for large lists:**
```javascript
import { FixedSizeList } from 'react-window';
```

**8. Avoid inline functions:**
```javascript
// Bad
<Child onClick={() => handleClick()} />

// Good
const handleClick = useCallback(() => { ... }, []);
<Child onClick={handleClick} />
```

**9. Use Production build:**
```bash
npm run build  # Minification, optimizations
```

**10. React DevTools Profiler:**
- Identify slow components
- Check render times

**For AariyaTech AI applications**: Essential when handling large datasets and complex computations.

---

## SECTION 3: NEXT.JS FUNDAMENTALS (20 Questions)

### Q36: What is Next.js and what advantages does it have over Create React App?

**Answer:** Next.js is a React framework providing server-side rendering, static generation, API routes, and optimization out of the box.

**Advantages over Create React App (CRA):**

| Feature | CRA | Next.js |
|---------|-----|---------|
| SSR | No | Yes |
| Static Generation | No | Yes |
| API Routes | No | Yes |
| File-based routing | No | Yes |
| Built-in optimization | Basic | Advanced |
| Deployment | Complex | Simple |
| Performance | Slower initial load | Faster |

**Key benefits:**
- **SEO**: Server-side rendering improves search visibility
- **Performance**: Automatic code splitting, image optimization
- **Development**: Built-in routing, no configuration needed
- **Full-stack**: Add backend API routes without separate server

**For AariyaTech projects**: Essential for SEO-sensitive EdTech and HRTech platforms.

---

### Q37: Explain the difference between Static Site Generation (SSG), Server-Side Rendering (SSR), and Incremental Static Regeneration (ISR).

**Answer:**

**Static Site Generation (SSG):**
```javascript
// pages/blog/[slug].js
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  return {
    props: { post },
    revalidate: 3600 // ISR: revalidate after 1 hour
  };
}

export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: 'blocking'
  };
}

export default function PostPage({ post }) {
  return <article>{post.content}</article>;
}
```

**Benefits:** Pre-rendered at build time, super fast, great for blogs, static content.

**Server-Side Rendering (SSR):**
```javascript
export async function getServerSideProps({ params }) {
  const data = await fetchData(params.id);
  return {
    props: { data },
    revalidate: 10 // cache for 10 seconds
  };
}

export default function Page({ data }) {
  return <div>{data.content}</div>;
}
```

**Benefits:** Fresh data on every request, good for personalized content, user-specific data.

**Incremental Static Regeneration (ISR):**
```javascript
export async function getStaticProps() {
  return {
    props: { data },
    revalidate: 60 // regenerate every 60 seconds
  };
}
```

**Benefits:** Combines speed of SSG with freshness of SSR.

**When to use:**
- **SSG**: Blogs, documentation, product pages
- **SSR**: User dashboards, personalized content, real-time data
- **ISR**: High-traffic pages needing updates but not real-time

---

### Q38: Explain file-based routing in Next.js. How does it work?

**Answer:** Next.js automatically creates routes based on file structure in the `pages` directory.

**File to Route mapping:**
```
pages/
  index.js → /
  about.js → /about
  posts/
    index.js → /posts
    [id].js → /posts/:id (dynamic route)
  [[...slug]].js → catch-all route
  api/
    users.js → /api/users
```

**Dynamic routes:**
```javascript
// pages/posts/[id].js
export async function getStaticProps({ params }) {
  return {
    props: {
      postId: params.id
    }
  };
}

export default function Post({ postId }) {
  return <div>Post {postId}</div>;
}

// Access via: /posts/1, /posts/2, etc.
```

**Catch-all routes:**
```javascript
// pages/docs/[[...slug]].js
export default function Docs({ slug }) {
  return <div>Path: {slug?.join('/')}</div>;
}
// /docs → /docs/guide → /docs/guide/setup
```

**Accessing route parameters:**
```javascript
import { useRouter } from 'next/router';

export default function Post() {
  const router = useRouter();
  const { id } = router.query;
  
  return <div>Post ID: {id}</div>;
}
```

---

### Q39: What are API routes in Next.js? How do you create them?

**Answer:** API routes allow creating a full-stack application in Next.js by adding backend endpoints without a separate server.

**Creating API routes:**
```javascript
// pages/api/hello.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    res.status(200).json({ message: 'Hello World' });
  } else {
    res.status(405).end();
  }
}

// Access via: GET /api/hello
```

**POST endpoint with database:**
```javascript
// pages/api/users.js
import connectDB from '../../lib/mongodb';

export default async function handler(req, res) {
  if (req.method === 'POST') {
    try {
      const db = await connectDB();
      const user = await db.collection('users').insertOne(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}
```

**Using in frontend:**
```javascript
async function createUser(userData) {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData)
  });
  return response.json();
}
```

**Benefits:**
- No separate backend server needed
- Same deployment as frontend
- Environment variables work the same
- Faster development

**For AariyaTech projects**: Ideal for prototyping and medium-scale applications.

---

### Q40: How do you handle environment variables in Next.js?

**Answer:** Next.js supports environment variables with different visibility levels.

**`.env.local` (version controlled):**
```
NEXT_PUBLIC_API_URL=https://api.example.com
SECRET_KEY=secret_value
```

**Frontend access (NEXT_PUBLIC prefix):**
```javascript
// Accessible in browser
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Available in server-side code and API routes
const secretKey = process.env.SECRET_KEY;
```

**In components:**
```javascript
export default function MyComponent() {
  return <div>API: {process.env.NEXT_PUBLIC_API_URL}</div>;
}
```

**In API routes:**
```javascript
// pages/api/endpoint.js
export default async function handler(req, res) {
  const secret = process.env.SECRET_KEY; // Safe - not exposed to browser
  // Use secret for authentication, database connections, etc.
}
```

**Different environments:**
```
.env.local → Local development
.env.development → Development environment
.env.production → Production environment
.env.test → Testing
```

**Best practices:**
- Never commit `.env.local` with secrets
- Use `.env.local` for local development
- Set secrets in hosting platform (Vercel, AWS, etc.) for production

---

### Q41: Explain Image optimization in Next.js using the `<Image>` component.

**Answer:** Next.js `<Image>` component provides automatic optimization including lazy loading, responsive sizing, and modern formats.

**Basic usage:**
```javascript
import Image from 'next/image';

export default function ProfilePicture() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile"
      width={200}
      height={200}
    />
  );
}
```

**With external URLs:**
```javascript
<Image
  src="https://example.com/image.jpg"
  alt="External"
  width={800}
  height={600}
/>
```

**Responsive images:**
```javascript
<Image
  src="/image.jpg"
  alt="Responsive"
  width={800}
  height={600}
  responsive={true}
  layout="responsive"
/>
```

**Lazy loading (default):**
```javascript
<Image
  src="/image.jpg"
  alt="Lazy"
  width={800}
  height={600}
  loading="lazy"
/>
```

**Benefits:**
- Automatic format conversion (WebP)
- Responsive sizing
- Lazy loading by default
- Prevents Cumulative Layout Shift (CLS)

**Configuration in next.config.js:**
```javascript
module.exports = {
  images: {
    domains: ['cdn.example.com'],
    formats: ['image/avif', 'image/webp']
  }
};
```

---

### Q42: How do you implement authentication in Next.js?

**Answer:** Authentication can be implemented using various strategies.

**Simple JWT approach:**
```javascript
// pages/api/login.js
import jwt from 'jsonwebtoken';

export default async function login(req, res) {
  const { email, password } = req.body;
  
  // Validate credentials
  const user = await validateUser(email, password);
  
  const token = jwt.sign(
    { userId: user.id, email: user.email },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
  
  res.setHeader('Set-Cookie', `token=${token}; Path=/; HttpOnly`);
  res.json({ success: true });
}
```

**Middleware for protected routes:**
```javascript
// middleware/auth.js
import jwt from 'jsonwebtoken';

export function withAuth(handler) {
  return async (req, res) => {
    try {
      const token = req.cookies.token;
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      req.user = decoded;
      return handler(req, res);
    } catch (err) {
      res.status(401).json({ error: 'Unauthorized' });
    }
  };
}
```

**Protected API route:**
```javascript
// pages/api/profile.js
import { withAuth } from '../middleware/auth';

export default withAuth(async (req, res) => {
  res.json({ user: req.user });
});
```

**Using NextAuth.js (recommended):**
```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from "next-auth";
import GithubProvider from "next-auth/providers/github";

export const authOptions = {
  providers: [
    GithubProvider({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
  ],
};

export default NextAuth(authOptions);
```

**For AariyaTech projects**: Use NextAuth.js for robust authentication in EdTech and HRTech applications.

---

### Q43: Explain data fetching methods in Next.js. When would you use each?

**Answer:**

| Method | Timing | Use Case |
|--------|--------|----------|
| `getStaticProps` | Build time | Static content, blogs |
| `getServerSideProps` | Request time | Real-time data, personalized |
| `getStaticPaths` | Build time | Dynamic static pages |
| Client-side fetch | Browser | Interactive data, filtering |

**getStaticProps example:**
```javascript
export async function getStaticProps() {
  const posts = await fetch('https://api.example.com/posts');
  return {
    props: { posts },
    revalidate: 3600 // ISR - revalidate every hour
  };
}
```

**getServerSideProps example:**
```javascript
export async function getServerSideProps(context) {
  const { params, req } = context;
  const data = await fetch(`https://api.example.com/user/${params.id}`);
  
  return {
    props: { data },
    revalidate: 10 // cache for 10 seconds
  };
}
```

**Client-side fetch:**
```javascript
import { useEffect, useState } from 'react';

export default function Posts() {
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => setPosts(data));
  }, []);
  
  return posts.map(post => <div key={post.id}>{post.title}</div>);
}
```

**Decision tree:**
- Is data static? → `getStaticProps`
- Does data change per request? → `getServerSideProps`
- User-specific data? → Client-side fetch
- Need instant updates? → Real-time database + client-side

---

### Q44: How do you deploy a Next.js application? What are the options?

**Answer:**

**Vercel (easiest, made by Next.js creators):**
```bash
npm install -g vercel
vercel
```

**Benefits:**
- Zero-config deployment
- Automatic CI/CD
- Preview deployments
- Great for team collaboration

**Docker deployment:**
```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

**AWS/Self-hosted:**
```bash
npm run build
npm start
```

**Environment setup:**
```javascript
// next.config.js
module.exports = {
  swcMinify: true,
  compress: true,
  poweredByHeader: false
};
```

**For AariyaTech projects:** Vercel for rapid iteration, AWS/self-hosted for enterprise features.

---

### Q45: What is the Next.js middleware and how do you use it?

**Answer:** Middleware runs before a request is processed, useful for authentication, redirects, and logging.

**Creating middleware:**
```javascript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/admin')) {
    const token = request.cookies.get('token');
    
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/admin/:path*', '/api/:path*']
};
```

**Common use cases:**
```javascript
// Logging
export function middleware(request) {
  console.log('Request:', request.nextUrl.pathname);
  return NextResponse.next();
}

// Add headers
export function middleware(request) {
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');
  return response;
}

// Redirect based on region
export function middleware(request) {
  const country = request.geo?.country;
  if (country === 'US') {
    return NextResponse.redirect(new URL('/en-US', request.url));
  }
}
```

---

### Q46: How do you implement real-time features in Next.js (WebSockets, live updates)?

**Answer:**

**Using Socket.IO:**
```javascript
// pages/api/socket.js
import { Server } from 'socket.io';

export default function handler(req, res) {
  if (!res.socket.server.io) {
    const io = new Server(res.socket.server);
    
    io.on('connection', (socket) => {
      console.log('User connected:', socket.id);
      
      socket.on('message', (data) => {
        io.emit('message', data);
      });
      
      socket.on('disconnect', () => {
        console.log('User disconnected');
      });
    });
    
    res.socket.server.io = io;
  }
  
  res.end();
}
```

**Frontend with Socket.IO:**
```javascript
import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

export default function Chat() {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    const socket = io();
    
    socket.on('message', (data) => {
      setMessages(prev => [...prev, data]);
    });
    
    return () => socket.disconnect();
  }, []);
  
  const sendMessage = (msg) => {
    socket.emit('message', msg);
  };
  
  return (
    <div>
      {messages.map((msg, i) => <p key={i}>{msg}</p>)}
      <input onKeyPress={(e) => sendMessage(e.target.value)} />
    </div>
  );
}
```

**Using Server-Sent Events (SSE):**
```javascript
// pages/api/events.js
export default function handler(req, res) {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  
  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ time: new Date() })}\n\n`);
  }, 1000);
  
  req.on('close', () => clearInterval(interval));
}
```

**For AariyaTech AI applications**: Essential for real-time collaboration, live data updates, and AI processing status.

---

### Q47: Explain the `_app.js` and `_document.js` files in Next.js.

**Answer:**

**`_app.js` - Wraps every page:**
```javascript
// pages/_app.js
import type { AppProps } from 'next/app';
import { SessionProvider } from 'next-auth/react';

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <SessionProvider session={pageProps.session}>
      <Header />
      <Component {...pageProps} />
      <Footer />
    </SessionProvider>
  );
}

export default MyApp;
```

**Use cases:**
- Global styles
- Providers (Redux, Context)
- Layout persistence
- Global error handling

**`_document.js` - HTML structure:**
```javascript
// pages/_document.js
import { Html, Head, Main, NextScript } from 'next/document';

export default function Document() {
  return (
    <Html>
      <Head>
        <link rel="icon" href="/favicon.ico" />
        <meta property="og:title" content="My App" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

**Use cases:**
- SEO meta tags
- Custom fonts
- Third-party scripts
- Global styles
- HTML attributes

**Key difference:**
- `_app.js`: Runs on both server and client, wraps page content
- `_document.js`: Only on server, controls HTML structure

---

### Q48: How do you optimize Core Web Vitals in Next.js?

**Answer:** Core Web Vitals are metrics for user experience: LCP, FID, CLS.

**Largest Contentful Paint (LCP) - <2.5s:**
```javascript
// Lazy load images
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero"
  priority={true} // for above-the-fold
/>

// Or client-side lazy load
<img loading="lazy" src="/image.jpg" />
```

**First Input Delay (FID) - <100ms:**
```javascript
// Break long tasks
export default function HeavyComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // Process in chunks
    const process = async () => {
      const result = await processLargeData();
      setData(result);
    };
    
    process();
  }, []);
}

// Use web workers for heavy computation
const worker = new Worker('worker.js');
worker.postMessage(data);
```

**Cumulative Layout Shift (CLS) - <0.1:**
```javascript
// Set explicit dimensions
<Image
  src="/image.jpg"
  width={800}
  height={600}
  alt="Fixed size"
/>

// Or use aspect ratio
<div style={{ aspectRatio: '16/9' }}>
  <Image src="/image.jpg" alt="Responsive" fill />
</div>
```

**Configuration:**
```javascript
// next.config.js
module.exports = {
  swcMinify: true, // Smaller bundles
  compress: true,
  productionBrowserSourceMaps: false
};
```

**Monitoring:**
```javascript
// pages/_document.js
import Script from 'next/script';

<Script
  src="https://cdn.jsdelivr.net/npm/web-vitals@3/dist/web-vitals.iife.js"
  strategy="afterInteractive"
/>
```

---

### Q49: How do you integrate with Supabase in a Next.js application?

**Answer:**

**Installation:**
```bash
npm install @supabase/supabase-js
```

**Setup:**
```javascript
// lib/supabase.js
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

**Authentication:**
```javascript
// pages/login.js
import { supabase } from '../lib/supabase';

export default function Login() {
  const [email, setEmail] = useState('');

  const handleLogin = async () => {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password: 'password'
    });
    
    if (error) console.error(error);
  };

  return (
    <button onClick={handleLogin}>Login</button>
  );
}
```

**Database queries:**
```javascript
// Fetch data
const { data, error } = await supabase
  .from('users')
  .select('*')
  .eq('id', userId);

// Insert data
const { data, error } = await supabase
  .from('users')
  .insert([{ name: 'John', email: 'john@example.com' }]);

// Realtime subscription
const subscription = supabase
  .from('users')
  .on('*', (payload) => {
    console.log('Change:', payload);
  })
  .subscribe();
```

**In API routes:**
```javascript
// pages/api/users.js
import { supabase } from '../../lib/supabase';

export default async function handler(req, res) {
  const { data, error } = await supabase
    .from('users')
    .select('*');
  
  res.json(data);
}
```

**For AariyaTech projects**: Perfect for rapid prototyping with real-time database capabilities.

---

### Q50: Explain Error Handling and Logging in Next.js.

**Answer:**

**Error page:**
```javascript
// pages/404.js
export default function Custom404() {
  return <div>404 - Page not found</div>;
}

// pages/500.js
export default function Custom500() {
  return <div>500 - Server error</div>;
}
```

**Try-catch in API routes:**
```javascript
// pages/api/users.js
export default async function handler(req, res) {
  try {
    const users = await fetchUsers();
    res.json(users);
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

**Global error handler:**
```javascript
// pages/_app.js
function MyApp({ Component, pageProps }) {
  useEffect(() => {
    const handleError = (event) => {
      console.error('Error:', event.error);
      // Send to error tracking service
    };

    window.addEventListener('error', handleError);
    return () => window.removeEventListener('error', handleError);
  }, []);

  return <Component {...pageProps} />;
}
```

**Logging service integration:**
```javascript
// lib/logger.js
export const logger = {
  error: (msg, error) => {
    console.error(msg, error);
    // Send to Sentry, LogRocket, etc.
    sendToErrorTracking(msg, error);
  },
  info: (msg) => console.log(msg),
};
```

**Using in components:**
```javascript
try {
  const response = await fetch('/api/data');
  const data = await response.json();
} catch (error) {
  logger.error('Failed to fetch data', error);
  setError('Unable to load data');
}
```

---

## SECTION 4: NODE.JS & EXPRESS FUNDAMENTALS (15 Questions)

### Q51: What is Node.js and why use it for backend development?

**Answer:** Node.js is a JavaScript runtime for server-side development, allowing full-stack development in JavaScript.

**Benefits:**
- **Single language**: Use JavaScript everywhere (frontend and backend)
- **Non-blocking I/O**: Asynchronous operations for high concurrency
- **NPM ecosystem**: Largest package repository
- **Event-driven**: Perfect for real-time applications
- **Scalability**: Handle many concurrent connections

**Event-driven architecture:**
```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();

emitter.on('data', (data) => {
  console.log('Data received:', data);
});

emitter.emit('data', 'Hello');
```

**For AariyaTech projects**: Essential for building AI-integrated, scalable backend services.

---

### Q52: Explain the Event Loop in Node.js.

**Answer:** The Event Loop continuously checks for tasks and executes them in phases.

**Phases (simplified):**
1. **Timers**: Execute `setTimeout`, `setInterval` callbacks
2. **Pending callbacks**: Execute deferred I/O operations
3. **Idle/Prepare**: Internal preparation
4. **Poll**: Wait for I/O events
5. **Check**: Execute `setImmediate` callbacks
6. **Close**: Clean up connections

**Example:**
```javascript
console.log('1. Start');

setTimeout(() => console.log('4. setTimeout'), 0);

Promise.resolve()
  .then(() => console.log('3. Promise (Microtask)'));

setImmediate(() => console.log('5. setImmediate'));

console.log('2. End');

// Output:
// 1. Start
// 2. End
// 3. Promise
// 4. setTimeout
// 5. setImmediate
```

**Key insight:** Microtasks (Promises, `process.nextTick`) execute before macrotasks (setTimeout, setImmediate).

---

### Q53: What are the differences between `require()` and `import`?

**Answer:**

| Feature | require() | import |
|---------|-----------|--------|
| Type | CommonJS | ES6 modules |
| Syntax | Dynamic | Static |
| Performance | Slower | Faster (static analysis) |
| Circular deps | Handled | Strict |
| Browser support | No | Yes (with bundlers) |

**require() example:**
```javascript
const express = require('express');
const { PORT } = require('./config');

module.exports = app;
```

**import example:**
```javascript
import express from 'express';
import { PORT } from './config.js';

export default app;
```

**Mixed usage (CommonJS default in Node.js):**
```javascript
// .js files use CommonJS
const app = require('express')();

// .mjs files use ES modules
import express from 'express';
```

**In Next.js:** Use ES6 `import` for cleaner syntax.

---

### Q54: What is Express.js and what are its core features?

**Answer:** Express is a minimal web framework for Node.js providing routing, middleware, and utilities.

**Core features:**
- **Routing**: Define URL patterns and handlers
- **Middleware**: Processing pipeline for requests
- **Templating**: Render HTML (though Next.js handles this)
- **Error handling**: Global error middleware

**Basic setup:**
```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json());

// Routes
app.get('/', (req, res) => {
  res.json({ message: 'Hello World' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

**Benefits:**
- Lightweight and flexible
- Large community and ecosystem
- Easy to learn and extend

**For AariyaTech projects**: Great for microservices and API backends alongside Next.js.

---

### Q55: Explain middleware in Express and provide examples.

**Answer:** Middleware functions have access to `req`, `res`, and `next` and can modify them or end the request cycle.

**Basic middleware:**
```javascript
const logMiddleware = (req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // Continue to next middleware
};

app.use(logMiddleware);
```

**Authentication middleware:**
```javascript
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

app.get('/protected', authMiddleware, (req, res) => {
  res.json({ user: req.user });
});
```

**Error handling middleware (must be last):**
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Server error' });
});
```

**CORS middleware:**
```javascript
const cors = require('cors');
app.use(cors());
```

**Order matters:**
```javascript
app.use(logMiddleware); // Logs all requests
app.use(authMiddleware); // Checks auth on protected routes
app.get('/data', (req, res) => { ... }); // Handler
```

---

### Q56: How do you handle asynchronous errors in Express?

**Answer:**

**Without try/catch - Error propagates unhandled:**
```javascript
app.get('/users/:id', async (req, res) => {
  const user = await getUser(req.params.id); // May throw
  res.json(user);
});
```

**With try/catch:**
```javascript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await getUser(req.params.id);
    res.json(user);
  } catch (error) {
    next(error); // Pass to error handler
  }
});
```

**Wrapper function (cleaner):**
```javascript
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await getUser(req.params.id);
  res.json(user);
}));
```

**Global error handler:**
```javascript
app.use((err, req, res, next) => {
  console.error(err);
  
  if (err.statusCode) {
    res.status(err.statusCode).json({ error: err.message });
  } else {
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

---

### Q57: Explain REST API principles and status codes.

**Answer:** REST (Representational State Transfer) uses HTTP methods and status codes for CRUD operations.

**HTTP Methods:**
```javascript
// GET - Retrieve data
GET /api/users
GET /api/users/1

// POST - Create data
POST /api/users
body: { name: 'John', email: 'john@example.com' }

// PUT - Replace entire resource
PUT /api/users/1
body: { name: 'Jane', email: 'jane@example.com' }

// PATCH - Partial update
PATCH /api/users/1
body: { name: 'Jane' }

// DELETE - Remove resource
DELETE /api/users/1
```

**Status codes:**
```javascript
200 OK - Successful GET, PUT, PATCH
201 Created - Successful POST
204 No Content - Successful DELETE
400 Bad Request - Invalid input
401 Unauthorized - Authentication required
403 Forbidden - No permission
404 Not Found - Resource doesn't exist
500 Internal Server Error - Server error
```

**Express example:**
```javascript
// GET
app.get('/api/users/:id', (req, res) => {
  const user = getUserById(req.params.id);
  res.json(user);
});

// POST
app.post('/api/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user);
});

// DELETE
app.delete('/api/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.status(204).send();
});
```

---

### Q58: How do you validate request data in Express?

**Answer:**

**Using express-validator:**
```bash
npm install express-validator
```

**Basic validation:**
```javascript
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
  body('email').isEmail(),
  body('name').notEmpty().isLength({ min: 3 }),
  body('age').isInt({ min: 18 })
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  
  // Process valid data
  res.json({ success: true });
});
```

**Custom validation:**
```javascript
body('email').custom((value) => {
  return User.findByEmail(value).then(user => {
    if (user) {
      throw new Error('Email already registered');
    }
  });
})
```

**Sanitization:**
```javascript
body('email').trim().toLowerCase(),
body('name').trim().escape()
```

**Quick validation with Joi:**
```javascript
const schema = Joi.object({
  email: Joi.string().email().required(),
  name: Joi.string().min(3).required(),
  age: Joi.number().min(18)
});

const { error, value } = schema.validate(req.body);
```

---

### Q59: How do you handle file uploads in Express?

**Answer:**

**Using multer:**
```bash
npm install multer
```

**Basic file upload:**
```javascript
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
  res.json({
    filename: req.file.filename,
    size: req.file.size
  });
});
```

**Multiple files:**
```javascript
app.post('/upload-multiple', upload.array('files', 5), (req, res) => {
  const files = req.files.map(f => ({
    name: f.filename,
    size: f.size
  }));
  res.json(files);
});
```

**Custom storage:**
```javascript
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    cb(null, `${Date.now()}-${file.originalname}`);
  }
});

const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype === 'image/jpeg') {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});

app.post('/upload', upload.single('image'), (req, res) => {
  res.json({ filename: req.file.filename });
});
```

**For AariyaTech projects**: Essential for AI-powered file processing applications.

---

### Q60: How do you connect to MongoDB from Node.js?

**Answer:**

**Using MongoDB driver:**
```bash
npm install mongodb
```

**Connection:**
```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient(process.env.MONGODB_URI);

async function connectDB() {
  try {
    await client.connect();
    console.log('Connected to MongoDB');
  } catch (err) {
    console.error('Connection failed:', err);
  }
}

connectDB();
```

**Using Mongoose (recommended for structure):**
```bash
npm install mongoose
```

**Setup with Express:**
```javascript
const mongoose = require('mongoose');
const express = require('express');

const app = express();

mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('DB error:', err));

// Define schema
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  createdAt: { type: Date, default: Date.now }
});

// Model
const User = mongoose.model('User', userSchema);

// Routes
app.post('/api/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.get('/api/users', async (req, res) => {
  const users = await User.find();
  res.json(users);
});
```

**Queries:**
```javascript
// Find
const user = await User.findById(id);
const users = await User.find({ status: 'active' });

// Update
await User.findByIdAndUpdate(id, { name: 'New Name' });

// Delete
await User.findByIdAndDelete(id);
```

---

## SECTION 5: TYPESCRIPT & PRACTICAL CONCEPTS (15 Questions)

### Q61: Why use TypeScript in a Node.js/Next.js project?

**Answer:** TypeScript adds static typing to JavaScript, catching errors at compile-time and improving development experience.

**Benefits:**
- **Type safety**: Catch errors before runtime
- **Better IDE support**: Autocomplete, go-to-definition
- **Self-documenting code**: Types serve as documentation
- **Refactoring safety**: Rename safely across codebase
- **Better for teams**: Clear contracts between functions

**Example:**
```typescript
// Without TypeScript - unclear what fetchUser returns
function getUserData(id) {
  return fetch(`/api/users/${id}`).then(r => r.json());
}

// With TypeScript - types are clear
interface User {
  id: number;
  name: string;
  email: string;
}

async function getUserData(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

**Setup in Next.js:**
```bash
# Create tsconfig.json
touch tsconfig.json

# Start dev server - Next.js auto-configures TypeScript
npm run dev
```

---

### Q62: Explain common TypeScript types and when to use them.

**Answer:**

**Primitive types:**
```typescript
const name: string = 'John';
const age: number = 30;
const isActive: boolean = true;
const nothing: null = null;
const undefined_value: undefined = undefined;
```

**Arrays:**
```typescript
const numbers: number[] = [1, 2, 3];
const strings: Array<string> = ['a', 'b'];
const mixed: (string | number)[] = [1, 'two', 3];
```

**Tuples:**
```typescript
const tuple: [string, number] = ['hello', 42];
```

**Any (avoid):**
```typescript
const value: any = 'anything'; // Defeats purpose of TypeScript
```

**Union types:**
```typescript
type Status = 'active' | 'inactive' | 'pending';
const status: Status = 'active'; // Type-safe
```

**Enum (for fixed values):**
```typescript
enum Role {
  Admin = 'ADMIN',
  User = 'USER',
  Guest = 'GUEST'
}

const userRole: Role = Role.Admin;
```

**Object type:**
```typescript
type User = {
  id: number;
  name: string;
  email?: string; // optional
};
```

---

### Q63: How do you define and use interfaces for API responses?

**Answer:**

**Define interfaces for consistency:**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}
```

**Use in API routes:**
```typescript
// pages/api/users.ts
import type { NextApiRequest, NextApiResponse } from 'next';

interface User {
  id: number;
  name: string;
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<User[]>
) {
  const users: User[] = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' }
  ];
  res.json(users);
}
```

**Use in components:**
```typescript
'use client';

interface User {
  id: number;
  name: string;
  email: string;
}

export default function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);

  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

**Generic responses:**
```typescript
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json();
}

// Usage with type
const user = await fetchData<User>('/api/users/1');
```

---

### Q64: Explain what are path aliases and how to set them up in TypeScript.

**Answer:** Path aliases allow cleaner imports by mapping paths to avoid relative paths (`../../../`).

**Setup in tsconfig.json:**
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@hooks/*": ["src/hooks/*"],
      "@types/*": ["src/types/*"]
    }
  }
}
```

**Without aliases (messy):**
```javascript
import Button from '../../../components/Button';
import { useAuth } from '../../hooks/useAuth';
import { formatDate } from '../../../utils/date';
```

**With aliases (clean):**
```javascript
import Button from '@components/Button';
import { useAuth } from '@hooks/useAuth';
import { formatDate } from '@utils/date';
```

**Common pattern:**
```json
{
  "paths": {
    "@/*": ["./*"],
    "@app/*": ["./app/*"],
    "@components/*": ["./components/*"],
    "@api/*": ["./pages/api/*"],
    "@lib/*": ["./lib/*"],
    "@public/*": ["./public/*"]
  }
}
```

**In Next.js:** Already configured by default with `@/*` pointing to the project root.

---

### Q65: How do you use TypeScript with React hooks?

**Answer:**

**useState with types:**
```typescript
// Explicit type
const [count, setCount] = useState<number>(0);

// Inferred type
const [count, setCount] = useState(0); // TypeScript knows it's number

// Object state
interface User {
  id: number;
  name: string;
}

const [user, setUser] = useState<User | null>(null);

// Union type
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
```

**useEffect with types:**
```typescript
useEffect(() => {
  // ...
}, []); // TypeScript checks dependencies

useEffect(() => {
  const timer: NodeJS.Timeout = setTimeout(() => {
    // ...
  }, 1000);

  return () => clearTimeout(timer);
}, []);
```

**useCallback with types:**
```typescript
const handleClick: React.MouseEventHandler<HTMLButtonElement> = (event) => {
  console.log(event.currentTarget.value);
};

const memoCallback = useCallback((id: number): Promise<void> => {
  return fetch(`/api/users/${id}`).then(r => r.json());
}, []);
```

**useRef with types:**
```typescript
const inputRef = useRef<HTMLInputElement>(null);

const handleClick = () => {
  if (inputRef.current) {
    inputRef.current.focus();
  }
};

return <input ref={inputRef} />;
```

**Custom hook with types:**
```typescript
function useCounter(initial: number = 0): { count: number; increment: () => void } {
  const [count, setCount] = useState(initial);
  
  const increment = useCallback(() => setCount(c => c + 1), []);
  
  return { count, increment };
}
```

---

### Q66: How do you handle forms with TypeScript in React?

**Answer:**

**Controlled form:**
```typescript
interface FormData {
  email: string;
  password: string;
}

export default function LoginForm() {
  const [form, setForm] = useState<FormData>({
    email: '',
    password: ''
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.currentTarget;
    setForm(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log(form);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
      />
      <input
        name="password"
        type="password"
        value={form.password}
        onChange={handleChange}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

**With react-hook-form (production):**
```typescript
import { useForm } from 'react-hook-form';

interface LoginInput {
  email: string;
  password: string;
}

export default function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<LoginInput>();

  const onSubmit = (data: LoginInput) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: true })} />
      {errors.email && <span>Email is required</span>}
      
      <input {...register('password', { required: true })} type="password" />
      {errors.password && <span>Password is required</span>}
      
      <button type="submit">Login</button>
    </form>
  );
}
```

---

### Q67: Explain async/await error handling in TypeScript.

**Answer:**

**Basic try/catch:**
```typescript
async function fetchUser(id: number): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const user: User = await response.json();
    return user;
  } catch (error) {
    if (error instanceof Error) {
      console.error('Error:', error.message);
    }
    throw error;
  }
}
```

**Custom error types:**
```typescript
class ApiError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string
  ) {
    super(message);
  }
}

async function fetchData<T>(url: string): Promise<T> {
  try {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new ApiError(
        response.status,
        'FETCH_ERROR',
        `Failed to fetch: ${response.statusText}`
      );
    }
    
    return response.json();
  } catch (error) {
    if (error instanceof ApiError) {
      console.error(`${error.code}: ${error.message}`);
    }
    throw error;
  }
}
```

**In React components:**
```typescript
const [error, setError] = useState<string | null>(null);

useEffect(() => {
  const loadUser = async () => {
    try {
      const data = await fetchUser(userId);
      setUser(data);
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Unknown error';
      setError(message);
    }
  };

  loadUser();
}, [userId]);
```

---

### Q68: What is the difference between `type` and `interface` in TypeScript? When to use each?

**Answer:** (Similar to Q10, but with more depth for production use)

**Differences:**
```typescript
// Interface - can be extended/implemented
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Type - uses intersection
type AnimalType = {
  name: string;
};

type DogType = AnimalType & {
  breed: string;
};
```

**Merging:**
```typescript
// Interfaces merge automatically
interface Window {
  customProperty: string;
}

interface Window {
  anotherProperty: number; // merged
}

// Types don't merge (error if you try)
type Config = { name: string };
type Config = { age: number }; // Error
```

**Use cases:**
```typescript
// Use interface for:
// - Object shapes (most common)
// - Class implementation
interface UserService {
  getUser(id: number): Promise<User>;
}

class UserServiceImpl implements UserService {
  async getUser(id: number): Promise<User> {
    // ...
  }
}

// Use type for:
// - Union types
type Status = 'active' | 'inactive';

// - Tuples
type Coordinates = [number, number];

// - Function signatures
type Formatter = (value: string) => string;

// - Generics
type Response<T> = { data: T; error?: string };
```

**Best practice for AariyaTech projects:**
- Use `interface` for API contracts and database schemas
- Use `type` for unions, function signatures, and utilities

---

### Q69: How do you debug TypeScript code in VS Code?

**Answer:**

**.vscode/launch.json for debugging:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/next",
      "args": ["dev"],
      "preLaunchTask": "npm",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

**Breakpoints:**
- Click left of line number to set breakpoint
- Run with `F5` to debug
- Step through code with `F10` (step over), `F11` (step into)

**Console debugging:**
```typescript
console.log('Value:', myVar);
console.table(arrayOfObjects); // Pretty print array
console.error('Error:', error);
console.trace(); // Stack trace
```

**VS Code Extensions:**
- Thunder Client or REST Client for API testing
- Debugger for Chrome for frontend debugging

---

### Q70: Explain TypeScript decorators and their use cases in Node.js/NestJS.

**Answer:** Decorators are functions that modify classes, methods, properties, or parameters. (Advanced topic, mainly used in NestJS.)

**Enable in tsconfig.json:**
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

**Class decorator:**
```typescript
function Entity(tableName: string) {
  return function <T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      tableName = tableName;
    };
  };
}

@Entity('users')
class User {
  name: string;
}
```

**Method decorator (NestJS example):**
```typescript
import { Controller, Get, Post } from '@nestjs/common';

@Controller('users')
export class UserController {
  @Get()
  getAllUsers() {
    return [];
  }

  @Post()
  createUser() {
    return {};
  }
}
```

**Property decorator:**
```typescript
function Validate(pattern: RegExp) {
  return function (target: Object, propertyKey: string) {
    let value: string;

    const getter = () => value;
    const setter = (newVal: string) => {
      if (!pattern.test(newVal)) {
        throw new Error('Invalid');
      }
      value = newVal;
    };

    Object.defineProperty(target, propertyKey, { get: getter, set: setter });
  };
}

class User {
  @Validate(/^\d+$/)
  age: string;
}
```

**For AariyaTech projects:** NestJS (uses decorators heavily) is great for enterprise-scale Node.js applications.

---

## SECTION 6: DATABASES & AWS (15 Questions)

### Q71: Explain MongoDB data model and when to use it vs SQL databases.

**Answer:** MongoDB is a NoSQL document database storing data as JSON-like documents.

**MongoDB data model:**
```javascript
// Collection: users
[
  {
    _id: ObjectId("..."),
    name: "John",
    email: "john@example.com",
    posts: [
      { title: "Post 1", content: "..." },
      { title: "Post 2", content: "..." }
    ],
    profile: {
      bio: "Developer",
      avatar: "url"
    }
  }
]
```

**MongoDB vs SQL:**

| Feature | MongoDB | SQL |
|---------|---------|-----|
| Structure | Flexible | Strict schema |
| Relationships | Embedded/References | Foreign keys |
| Transactions | ACID (4.0+) | Full ACID |
| Scale | Horizontal | Vertical |
| Query | JavaScript | SQL |
| Joins | Complex | Native |

**When to use MongoDB:**
- Rapidly changing schema
- Document-oriented data
- High write throughput
- Horizontal scaling needed
- Flexibility over consistency

**When to use SQL:**
- Structured data
- Complex relationships
- ACID transactions
- Data integrity critical
- Reporting/analytics

**For AariyaTech projects:**
- MongoDB: AI output storage, user-generated content
- SQL/PostgreSQL: Financial records, structured data

---

### Q72: How do you design a MongoDB schema? Best practices?

**Answer:**

**Schema design considerations:**
```javascript
// Option 1: Embed data (denormalization)
db.users.insertOne({
  _id: 1,
  name: 'John',
  address: {
    street: '123 Main',
    city: 'NYC'
  },
  posts: [
    { title: 'Post 1', content: '...' }
  ]
});

// Option 2: Reference data (normalization)
db.users.insertOne({
  _id: 1,
  name: 'John',
  addressId: ObjectId('...')
});

db.addresses.insertOne({
  _id: ObjectId('...'),
  street: '123 Main',
  city: 'NYC'
});
```

**Best practices:**
```javascript
// 1. Use proper types
const schema = new Schema({
  name: { type: String, required: true },
  email: { type: String, unique: true, lowercase: true },
  age: { type: Number, min: 0, max: 150 },
  createdAt: { type: Date, default: Date.now },
  isActive: { type: Boolean, default: true }
});

// 2. Index frequently queried fields
schema.index({ email: 1 });
schema.index({ createdAt: -1 });

// 3. Embed when relationship is one-to-few
const postSchema = new Schema({
  title: String,
  comments: [{
    author: String,
    text: String
  }]
});

// 4. Reference when relationship is many-to-many
const userSchema = new Schema({
  name: String,
  friends: [{ type: ObjectId, ref: 'User' }]
});

// 5. Use validation
schema.pre('save', function(next) {
  if (this.age < 18) {
    throw new Error('Must be 18+');
  }
  next();
});
```

---

### Q73: What are indexes in MongoDB and when should you create them?

**Answer:** Indexes improve query performance by allowing fast data retrieval without scanning all documents.

**Creating indexes:**
```javascript
// Single field index
db.users.createIndex({ email: 1 });

// Ascending (1) or descending (-1)
db.posts.createIndex({ createdAt: -1 });

// Compound index
db.users.createIndex({ status: 1, createdAt: -1 });

// Unique index
db.users.createIndex({ email: 1 }, { unique: true });

// Text index for full-text search
db.posts.createIndex({ title: 'text', content: 'text' });
```

**In Mongoose:**
```javascript
const schema = new Schema({
  email: { type: String, index: true, unique: true },
  username: { type: String, index: true },
  createdAt: { type: Date, index: true }
});
```

**When to index:**
- Frequently queried fields
- Sort operations
- Fields in WHERE clauses
- Unique constraints

**Performance implications:**
- Faster reads
- Slower writes (index maintenance)
- Extra disk space

**Analyze queries:**
```javascript
// Check if index is used
db.users.find({ email: 'test@example.com' }).explain('executionStats');

// Look for 'COLLSCAN' (bad - full scan) vs 'IXSCAN' (good - index scan)
```

---

### Q74: What is AWS and which services are relevant for MERN stack applications?

**Answer:** AWS (Amazon Web Services) provides cloud infrastructure and services. Key services for MERN:

**Compute:**
- **EC2**: Virtual machines for running Node.js/Next.js servers
- **Lambda**: Serverless functions (perfect for API routes)
- **App Runner**: Container deployment (simpler than EC2)

**Storage:**
- **S3**: File storage (images, documents, AI outputs)
- **RDS**: Managed databases (PostgreSQL, MySQL)
- **DynamoDB**: NoSQL database
- **ElastiCache**: Redis caching

**Networking:**
- **CloudFront**: CDN for fast content delivery
- **Route 53**: DNS management
- **ALB**: Load balancer

**Useful services:**
- **SQS**: Message queuing for async tasks
- **SNS**: Notifications
- **CloudWatch**: Logging and monitoring

**For AariyaTech projects:**
```
Frontend (Next.js) → CloudFront (CDN)
         ↓
API (Node.js) → EC2 or App Runner
         ↓
Database → RDS (PostgreSQL) or DynamoDB
         ↓
File storage → S3
```

---

### Q75: How do you deploy a MERN application on AWS?

**Answer:**

**Architecture:**
```
GitHub → CodePipeline → CodeBuild → CodeDeploy → EC2/App Runner
              ↓
         Store artifacts in S3
              ↓
         Update CloudFront cache
```

**Step 1: Prepare application**
```bash
# Build frontend
npm run build

# Create Dockerfile
docker build -t mern-app .

# Push to ECR (Elastic Container Registry)
docker tag mern-app:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/mern-app
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/mern-app
```

**Step 2: Create EC2 instance**
```bash
# SSH into EC2
ssh -i key.pem ec2-user@instance-ip

# Install Docker
sudo yum install docker -y
sudo service docker start

# Pull and run container
docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/mern-app
docker run -d -p 80:3000 mern-app
```

**Step 3: Setup RDS database**
```bash
# Create RDS instance (PostgreSQL)
# Connect from application
DATABASE_URL=postgresql://user:pass@rds-endpoint:5432/dbname
```

**Step 4: Setup S3 for file uploads**
```javascript
// Node.js code
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

app.post('/upload', async (req, res) => {
  const params = {
    Bucket: 'my-bucket',
    Key: `uploads/${Date.now()}-${req.file.name}`,
    Body: req.file.data
  };
  
  const result = await s3.upload(params).promise();
  res.json({ url: result.Location });
});
```

**Step 5: Configure CloudFront**
- Origin: Your EC2/App Runner instance
- Cache settings for static assets
- HTTPS certificate

---

### Q76: How do you handle environment-specific configurations in Node.js?

**Answer:**

**Using .env files:**
```bash
# .env.development
NODE_ENV=development
DATABASE_URL=mongodb://localhost/dev_db
API_URL=http://localhost:3000

# .env.production
NODE_ENV=production
DATABASE_URL=mongodb://prod-server/db
API_URL=https://api.example.com
```

**Load environment variables:**
```bash
npm install dotenv
```

**In Node.js:**
```javascript
require('dotenv').config();

const db = process.env.DATABASE_URL;
const env = process.env.NODE_ENV;
```

**Configuration module (best practice):**
```javascript
// config/config.js
module.exports = {
  development: {
    db: 'mongodb://localhost/dev',
    apiUrl: 'http://localhost:3000',
    debug: true
  },
  production: {
    db: process.env.DATABASE_URL,
    apiUrl: process.env.API_URL,
    debug: false
  },
  test: {
    db: 'mongodb://localhost/test',
    apiUrl: 'http://localhost:3000',
    debug: true
  }
};

const config = require('./config')[process.env.NODE_ENV || 'development'];
```

**In Express app:**
```javascript
const config = require('./config/config');

app.use((req, res, next) => {
  req.config = config;
  next();
});

app.get('/api/config', (req, res) => {
  res.json({ debug: req.config.debug });
});
```

**AWS Secrets Manager (production):**
```javascript
const AWS = require('aws-sdk');
const client = new AWS.SecretsManager();

async function getSecrets() {
  const data = await client.getSecretValue({ SecretId: 'prod/db' }).promise();
  return JSON.parse(data.SecretString);
}
```

---

### Q77: Explain database transactions and ACID properties.

**Answer:** Transactions ensure database operations are reliable and data-consistent.

**ACID properties:**
- **Atomicity**: All-or-nothing (commit or rollback)
- **Consistency**: Data moves from valid to valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data survives failures

**MongoDB transactions:**
```javascript
const session = db.getMongo().startSession();

try {
  session.startTransaction();
  
  // Operations
  db.users.updateOne(
    { _id: userId },
    { $inc: { balance: -100 } },
    { session }
  );
  
  db.accounts.updateOne(
    { _id: accountId },
    { $inc: { balance: 100 } },
    { session }
  );
  
  session.commitTransaction();
} catch (err) {
  session.abortTransaction();
  throw err;
} finally {
  session.endSession();
}
```

**SQL transactions:**
```sql
BEGIN TRANSACTION;

UPDATE users SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Rollback if error
-- ROLLBACK;

-- Commit if success
COMMIT;
```

**With Mongoose:**
```javascript
const session = await mongoose.startSession();

try {
  session.startTransaction();
  
  await User.findByIdAndUpdate(userId, { balance: -100 }, { session });
  await Account.findByIdAndUpdate(accountId, { balance: 100 }, { session });
  
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
  throw err;
} finally {
  session.endSession();
}
```

---

### Q78: How do you optimize database queries for performance?

**Answer:**

**1. Use indexes:**
```javascript
// Create index on frequently queried fields
db.users.createIndex({ email: 1, status: 1 });

// Check if query uses index
db.users.find({ email: 'test@example.com' }).explain('executionStats');
```

**2. Select only needed fields:**
```javascript
// Bad - retrieves all fields
const users = await User.find();

// Good - select specific fields
const users = await User.find().select('name email');
```

**3. Use aggregation pipeline:**
```javascript
const stats = await User.aggregate([
  { $match: { status: 'active' } },
  { $group: { _id: '$category', count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);
```

**4. Pagination for large datasets:**
```javascript
app.get('/api/users', async (req, res) => {
  const page = req.query.page || 1;
  const limit = 20;
  const skip = (page - 1) * limit;
  
  const users = await User.find()
    .skip(skip)
    .limit(limit)
    .sort({ createdAt: -1 });
  
  res.json(users);
});
```

**5. Lazy load relationships:**
```javascript
// Bad - eager load all posts for each user
const users = await User.find().populate('posts');

// Good - populate only when needed
const user = await User.findById(userId).populate('posts');
```

**6. Cache frequently accessed data:**
```javascript
const redis = require('redis');
const client = redis.createClient();

app.get('/api/posts/:id', async (req, res) => {
  const cacheKey = `post:${req.params.id}`;
  
  // Check cache
  const cached = await client.get(cacheKey);
  if (cached) return res.json(JSON.parse(cached));
  
  // Query database
  const post = await Post.findById(req.params.id);
  
  // Store in cache for 1 hour
  await client.setex(cacheKey, 3600, JSON.stringify(post));
  
  res.json(post);
});
```

---

### Q79: What is Supabase and how does it compare to MongoDB?

**Answer:** Supabase is an open-source Firebase alternative providing PostgreSQL database, authentication, and real-time updates.

**Supabase vs MongoDB:**

| Feature | Supabase | MongoDB |
|---------|----------|---------|
| Database | PostgreSQL | Document |
| Real-time | Built-in | No |
| Auth | Included | Need to implement |
| Functions | SQL/Plpgsql | JavaScript |
| Relationships | Foreign keys | Denormalization |
| Scale | Vertical | Horizontal |
| Consistency | Strong | Eventual |

**Supabase setup:**
```bash
npm install @supabase/supabase-js
```

**Usage:**
```javascript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(url, key);

// Query
const { data, error } = await supabase
  .from('users')
  .select('*')
  .eq('id', 1);

// Insert
const { data, error } = await supabase
  .from('users')
  .insert([{ name: 'John', email: 'john@example.com' }]);

// Real-time subscription
const subscription = supabase
  .from('users')
  .on('*', (payload) => {
    console.log('Change received!', payload);
  })
  .subscribe();
```

**For AariyaTech projects:**
- **MongoDB**: Flexible schemas, high write throughput
- **Supabase**: Strong consistency, real-time features, authentication out-of-the-box

---

### Q80: Explain connection pooling and why it's important for database performance.

**Answer:** Connection pooling reuses database connections instead of creating new ones for each request, improving performance.

**Without pooling (inefficient):**
```javascript
// Each request creates new connection
app.get('/users', async (req, res) => {
  const client = new MongoClient(url);
  await client.connect();
  const users = await client.db('mydb').collection('users').find().toArray();
  await client.close();
  res.json(users);
});
// Very slow!
```

**With connection pooling:**
```javascript
// Create one pool, reuse connections
const pool = new Pool({
  max: 20, // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

app.get('/users', async (req, res) => {
  const client = await pool.connect();
  try {
    const users = await client.query('SELECT * FROM users');
    res.json(users.rows);
  } finally {
    client.release(); // Return to pool
  }
});
```

**Mongoose (automatic pooling):**
```javascript
mongoose.connect(url, {
  maxPoolSize: 10,
  minPoolSize: 5
});
```

**Benefits:**
- Faster response times
- Lower resource usage
- Better concurrency handling
- Improved scalability

---

## SECTION 7: PRACTICAL SKILLS & SOFT SKILLS (10 Questions)

### Q81: How would you approach debugging a performance issue in a React application?

**Answer:**

**Step 1: Identify the issue**
```javascript
// Use browser DevTools Performance tab
// Or React DevTools Profiler
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration) {
  console.log(`Component ${id} (${phase}) took ${actualDuration}ms`);
}

<Profiler id="App" onRender={onRenderCallback}>
  <App />
</Profiler>
```

**Step 2: Measure before/after**
```javascript
const startTime = performance.now();
// code
const endTime = performance.now();
console.log(`Time: ${endTime - startTime}ms`);
```

**Step 3: Common performance issues**
```javascript
// 1. Unnecessary re-renders
const MemoComponent = memo(Component);

// 2. Missing keys in lists
{items.map(item => <Item key={item.id} />)} // Good

// 3. Inline functions
const handleClick = useCallback(() => {}, []); // Good

// 4. Large bundles
const HeavyComponent = lazy(() => import('./Heavy'));

// 5. Inefficient selectors
// Bad: computes on every render
const total = users.filter(u => u.active).reduce((sum, u) => sum + u.value, 0);

// Good: memoized
const total = useMemo(() => 
  users.filter(u => u.active).reduce((sum, u) => sum + u.value, 0),
  [users]
);
```

**Step 4: Use profiler tools**
- Chrome DevTools Performance
- React DevTools Profiler
- Lighthouse
- Web Vitals

---

### Q82: How do you handle API rate limiting and what are best practices?

**Answer:**

**Implementing rate limiting on backend:**
```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit per IP
  message: 'Too many requests',
  standardHeaders: true,
  legacyHeaders: false
});

app.use(limiter);

// Specific route limits
const strictLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5
});

app.post('/api/login', strictLimiter, loginHandler);
```

**Client-side handling:**
```javascript
async function fetchWithRetry(url, options, retries = 3) {
  try {
    const response = await fetch(url, options);
    
    if (response.status === 429) { // Too Many Requests
      const retryAfter = response.headers.get('Retry-After');
      const delay = (retryAfter || 60) * 1000;
      
      await new Promise(resolve => setTimeout(resolve, delay));
      
      if (retries > 0) {
        return fetchWithRetry(url, options, retries - 1);
      }
    }
    
    return response;
  } catch (error) {
    throw error;
  }
}
```

**Using Redis for distributed rate limiting:**
```javascript
const RedisStore = require('rate-limit-redis');
const redis = require('redis');
const redisClient = redis.createClient();

const limiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate-limit:'
  }),
  windowMs: 15 * 60 * 1000,
  max: 100
});

app.use('/api/', limiter);
```

**Best practices:**
- Implement per-user limits
- Use different limits for different endpoints
- Provide clear error messages
- Implement exponential backoff on client
- Monitor rate limit hits

---

### Q83: How do you approach code review and provide constructive feedback?

**Answer:**

**As a reviewer:**
1. **Check functionality**: Does it solve the problem?
2. **Review code quality**: Is it clean, readable, maintainable?
3. **Security**: Any vulnerabilities?
4. **Performance**: Any bottlenecks?
5. **Tests**: Are there adequate tests?

**Constructive feedback example:**
```
# Bad feedback
❌ "This is bad code"

# Good feedback
✅ "This query could be optimized by adding an index on the email field. 
   It currently scans all documents (COLLSCAN). Here's how to fix it: ..."
```

**Code review checklist:**
```javascript
// Security
- No hardcoded secrets
- Input validation
- SQL injection prevention
- XSS protection

// Performance
- Unnecessary API calls
- Large bundle sizes
- N+1 queries
- Memory leaks

// Maintainability
- Clear variable names
- Comments for complex logic
- DRY principle (Don't Repeat Yourself)
- Consistent coding style

// Testing
- Unit tests for logic
- Integration tests for flows
- Edge case coverage
```

**Tools:**
- GitHub pull requests
- Prettier (code formatting)
- ESLint (code quality)
- Pre-commit hooks

---

### Q84: How do you stay updated with latest technologies and best practices?

**Answer:**

**For the MERN stack and AariyaTech projects:**

1. **Follow core maintainers:**
   - React: Dan Abramov, Kent C. Dodds blogs
   - Next.js: Vercel blog
   - Node.js: Official release notes

2. **Subscribe to newsletters:**
   - JavaScript Weekly
   - React Newsletter
   - Node Weekly
   - CSS-Tricks

3. **Read documentation:**
   - React docs
   - Next.js docs
   - Node.js docs
   - MDN Web Docs

4. **Contribute to open source:**
   - Build portfolio
   - Learn from experienced developers
   - Gain real-world experience

5. **Follow tech communities:**
   - Dev.to
   - Hacker News
   - Twitter tech accounts
   - Stack Overflow

6. **Take courses and certifications:**
   - Udemy
   - Coursera
   - Pluralsight
   - Egghead

7. **Build side projects:**
   - Best way to learn
   - Apply new technologies
   - Build portfolio

**For AariyaTech internship:**
- Follow AI/ML updates (given company focus)
- Learn about spatial computing
- Understand EdTech/HRTech trends
- Study market gaps in green tech

---

### Q85: How do you approach learning a new technology or framework?

**Answer:**

**Step 1: Understand the fundamentals**
```
- What problem does it solve?
- How does it compare to alternatives?
- When should you use it?
```

**Step 2: Set up and experiment**
```bash
# Create hello world
# Follow official tutorial
# Build small project
```

**Step 3: Build a real project**
```javascript
// Don't just follow tutorials
// Build something you'd actually use
// Encounter real problems
// Learn how to solve them
```

**Step 4: Read source code and best practices**
```
- Study popular open-source projects
- Understand design patterns
- Learn from experienced developers
```

**Example: Learning Next.js**
```
1. Read: "What is Next.js and what problems does it solve?"
2. Setup: Create new Next.js app
3. Build: Create multi-page application
4. Debug: Encounter issues, solve them
5. Optimize: Learn SSR, SSG, ISR
6. Deploy: Put on production (Vercel)
7. Study: Read Next.js source code, popular projects
```

**For AariyaTech projects:**
- Learn the specific tech stack they use
- Study their existing codebase
- Understand their architecture
- Ask senior devs for guidance

---

### Q86: How would you handle disagreement with a senior developer about architecture?

**Answer:**

**Respectful approach:**
1. **Listen first**: Understand their perspective
2. **Ask questions**: "Can you explain why this approach?"
3. **Present your idea**: "I was thinking about..."
4. **Focus on tradeoffs**: "This approach has benefits X, but risks Y"
5. **Defer to experience**: Senior developers usually have good reasons
6. **Offer to prototype**: "Can I build a proof-of-concept?"

**Example:**
```
You: "I think we should use MongoDB for this project"
Senior: "I prefer PostgreSQL because..."

Good response:
"That makes sense. I understand the need for strong consistency.
However, I was thinking about the flexibility we'd gain. 
Can we discuss the tradeoffs? What if we prototype with PostgreSQL first
and reassess after we understand the data model better?"
```

**Key principles:**
- Be humble and open to learning
- Focus on project goals, not ego
- Provide data/evidence for your position
- Trust experienced developers
- Document decisions for future reference

---

### Q87: How do you estimate time for a development task?

**Answer:**

**Method 1: Break down into sub-tasks**
```
Feature: Build user authentication
├── Design database schema (1h)
├── Create Supabase project (30m)
├── Build login API (2h)
├── Build register API (2h)
├── Build forgot password (2h)
├── Add email verification (1.5h)
├── Write tests (2h)
├── Security review (1h)
└── Buffer (1h)
Total: ~13.5 hours → 2 days
```

**Method 2: Use past experience**
```
Feature size: Medium
Similar feature took: 5 days
Buffer: +20%
Estimate: 6 days
```

**Best practices:**
- Always add buffer (20-30%)
- Break into smaller tasks
- Estimate pessimistically
- Account for code review, testing
- Don't include meetings/interruptions
- Re-estimate as you learn

**For AariyaTech projects:**
```
Sprint estimation:
- Complexity assessment (simple/medium/complex)
- Story points: 1-2 (simple), 3-5 (medium), 8-13 (complex)
- Velocity: track completed points per sprint
- Use velocity to forecast next sprint
```

**Common underestimation sources:**
- Meetings and interruptions
- Testing and debugging
- Code review iterations
- Unexpected blockers
- Learning new technologies

---

### Q88: How would you approach testing a complex feature?

**Answer:**

**Unit tests** - Test individual functions:
```javascript
// validateEmail.test.js
import { validateEmail } from './validateEmail';

describe('validateEmail', () => {
  it('should return true for valid email', () => {
    expect(validateEmail('test@example.com')).toBe(true);
  });

  it('should return false for invalid email', () => {
    expect(validateEmail('invalid')).toBe(false);
  });

  it('should handle edge cases', () => {
    expect(validateEmail('')).toBe(false);
    expect(validateEmail('test@')).toBe(false);
  });
});
```

**Integration tests** - Test component interactions:
```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import LoginForm from './LoginForm';

describe('LoginForm integration', () => {
  it('should submit form with email and password', async () => {
    render(<LoginForm />);
    
    const emailInput = screen.getByPlaceholderText('Email');
    const passwordInput = screen.getByPlaceholderText('Password');
    const submitButton = screen.getByText('Login');
    
    fireEvent.change(emailInput, { target: { value: 'test@example.com' } });
    fireEvent.change(passwordInput, { target: { value: 'password123' } });
    fireEvent.click(submitButton);
    
    expect(screen.getByText('Login successful')).toBeInTheDocument();
  });
});
```

**End-to-end tests** - Test complete user flow:
```javascript
// cypress/e2e/auth.cy.js
describe('Authentication flow', () => {
  it('should register and login user', () => {
    cy.visit('http://localhost:3000');
    
    cy.get('[data-testid="register-link"]').click();
    cy.get('input[name="email"]').type('newuser@example.com');
    cy.get('input[name="password"]').type('Password123');
    cy.get('button[type="submit"]').click();
    
    cy.url().should('include', '/login');
    
    cy.get('input[name="email"]').type('newuser@example.com');
    cy.get('input[name="password"]').type('Password123');
    cy.get('button[type="submit"]').click();
    
    cy.url().should('include', '/dashboard');
  });
});
```

**Test pyramid:**
```
         E2E (5%)
       Integration (15%)
    Unit (80%)
```

---

### Q89: How do you write clean, maintainable code?

**Answer:**

**1. Clear naming:**
```javascript
// Bad
const d = 5;
const proc = (x) => x * 2;

// Good
const maxRetries = 5;
const calculateTotalPrice = (price) => price * 2;
```

**2. Functions should do one thing:**
```javascript
// Bad
function processUserData(user) {
  // validate user
  // save to database
  // send email
  // log activity
}

// Good
function validateUser(user) { ... }
function saveUser(user) { ... }
function sendWelcomeEmail(user) { ... }
function logActivity(action) { ... }
```

**3. DRY principle (Don't Repeat Yourself):**
```javascript
// Bad
function getAdminUsers() {
  return db.users.find({ role: 'admin' });
}

function getActiveUsers() {
  return db.users.find({ status: 'active' });
}

// Good
function getUsers(query) {
  return db.users.find(query);
}
```

**4. SOLID principles:**
```javascript
// S - Single Responsibility
class UserRepository { /* only handles user data */ }
class EmailService { /* only handles emails */ }

// O - Open/Closed
class PaymentProcessor {
  process(payment) {
    const processor = PaymentFactory.create(payment.type);
    return processor.process(payment);
  }
}

// L - Liskov Substitution
interface PaymentMethod {
  process(amount: number): Promise<void>;
}

// I - Interface Segregation
interface UserRepository {
  getUser(id): User;
  saveUser(user): void;
}

// D - Dependency Injection
class UserService {
  constructor(private repository: UserRepository) {}
}
```

**5. Error handling:**
```javascript
// Good error handling
try {
  const user = await getUser(id);
  if (!user) throw new NotFoundError('User not found');
  return user;
} catch (error) {
  logger.error('Failed to get user', { id, error });
  throw error;
}
```

---

### Q90: How would you propose a significant refactoring of legacy code?

**Answer:**

**Step 1: Document current state**
```
- Map all modules and dependencies
- Identify pain points
- Measure performance metrics
- Document business logic
```

**Step 2: Build business case**
```
Current problems:
- New feature development takes 3x longer
- Bug fix complexity increasing
- Technical debt growing

Benefits of refactoring:
- 30% faster feature development
- Fewer bugs
- Easier onboarding
- Lower maintenance cost

Estimated effort: 4 weeks
Expected ROI: 6 months
```

**Step 3: Propose incremental approach**
```javascript
// Instead of rewriting everything at once
// Refactor piece by piece while maintaining functionality

// Phase 1: Extract domain logic (week 1)
// Phase 2: Modernize API layer (week 2)
// Phase 3: Update database layer (week 3)
// Phase 4: Testing and documentation (week 4)

// Each phase independently deployable
// Minimize risk of breaking functionality
```

**Step 4: Add tests during refactoring**
```javascript
// Ensure current behavior is tested
// Refactor with confidence
// Tests prevent regressions

const testsBeforeRefactoring = 40;
const testCoverageGoal = 80;
```

**Step 5: Get buy-in from team**
```
- Share ROI analysis
- Demonstrate risk mitigation
- Include team in planning
- Get agreement on timeline
- Assign clear ownership
```

**Example proposal for AariyaTech:**
```
Proposal: Migrate auth system to NextAuth.js

Current state:
- Manual JWT implementation
- Repetitive authentication logic
- Limited provider integrations

Benefits:
- Eliminate 500+ lines of code
- Support OAuth providers (Google, GitHub)
- Better security practices
- 40% faster implementation of new features

Timeline: 2 weeks
Risk: Low (clear migration path)
```

---

## SECTION 8: AI/ML & ADVANCED CONCEPTS (10 Questions)

### Q91: How would you integrate AI/ML features into a MERN application?

**Answer:** Given AariyaTech's AI focus, this is important.

**Using Google Generative AI (Gemini) API:**
```bash
npm install @google/generative-ai
```

**Backend integration:**
```javascript
// pages/api/ai-generate.js
import { GoogleGenerativeAI } from '@google/generative-ai';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { prompt } = req.body;
    
    const model = genAI.getGenerativeModel({ model: 'gemini-pro' });
    const result = await model.generateContent(prompt);
    const text = result.response.text();
    
    res.json({ generated: text });
  }
}
```

**Frontend integration:**
```javascript
function AIGenerator() {
  const [prompt, setPrompt] = useState('');
  const [result, setResult] = useState('');
  const [loading, setLoading] = useState(false);

  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/ai-generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ prompt })
    });
    
    const data = await response.json();
    setResult(data.generated);
    setLoading(false);
  };

  return (
    <div>
      <textarea value={prompt} onChange={(e) => setPrompt(e.target.value)} />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate'}
      </button>
      {result && <div>{result}</div>}
    </div>
  );
}
```

**Using TensorFlow.js (browser-based ML):**
```bash
npm install @tensorflow/tfjs @tensorflow/tfjs-automl
```

**Image classification:**
```javascript
import * as tf from '@tensorflow/tfjs';

async function classifyImage(imageElement) {
  const model = await tf.loadLayersModel('model.json');
  
  const predictions = tf.tidy(() => {
    let img = tf.browser.fromPixels(imageElement);
    img = img.asType('float32').div(255.0);
    
    return model.predict(img.expandDims(0));
  });
  
  return predictions.data();
}
```

**For AariyaTech projects:**
- Content generation (AI design tools)
- Data analysis (HRTech insights)
- Personalization (EdTech recommendations)
- Chatbots (customer support)

---

### Q92: Explain what prompt engineering is and why it matters for AI applications.

**Answer:** Prompt engineering is crafting inputs to AI models to get desired outputs.

**Basic prompts:**
```javascript
// Vague prompt - poor results
"Write about technology"

// Well-engineered prompt - better results
`Write a brief overview of machine learning for non-technical audience.
Include 2 examples relevant to e-commerce.
Keep it to 150 words.
Format: Introduction, Examples, Conclusion`
```

**Prompt engineering techniques:**

1. **Specificity:**
```
❌ "Summarize this"
✅ "Summarize this technical article in 3 bullet points for a non-technical audience"
```

2. **Context and role:**
```
"You are an experienced software architect. 
Design a microservices architecture for an e-commerce platform."
```

3. **Temperature and creativity:**
```javascript
const response = await model.generateContent({
  prompt: "Write creative product descriptions",
  temperature: 0.8 // Higher = more creative
});

const response2 = await model.generateContent({
  prompt: "Extract key entities from text",
  temperature: 0.1 // Lower = more factual
});
```

4. **Few-shot learning:**
```
Example 1:
Input: "I love this product!"
Output: Sentiment: Positive

Example 2:
Input: "This is terrible"
Output: Sentiment: Negative

Now analyze: "It's okay, nothing special"
```

**For AariyaTech UI design plugin (from user bio):**
```javascript
const designPrompt = `
You are a UI/UX expert. Convert this description into a React component structure.

Description: ${userDescription}

Requirements:
- Component should be reusable
- Include Tailwind CSS classes
- Make it responsive
- Follow modern design patterns

Output format:
\`\`\`jsx
// Your code here
\`\`\`
`;
```

**Best practices:**
- Be specific and clear
- Provide context
- Give examples
- Specify output format
- Test and iterate
- Monitor costs (API calls)

---

### Q93: How do you optimize AI model inference for production?

**Answer:**

**1. Model compression:**
```bash
# Convert large models to optimized format
npm install onnx onnx-optimizer

# Quantization reduces model size
# int8 quantization: 4x smaller model
# Minimal accuracy loss
```

**2. Batch processing:**
```javascript
// Instead of single predictions
for (let i = 0; i < data.length; i++) {
  const prediction = await model.predict(data[i]);
}

// Process as batch (much faster)
const predictions = await model.predict(tf.stack(data));
```

**3. Caching predictions:**
```javascript
// Redis caching for expensive predictions
const cache = new Map();

async function predictWithCache(input) {
  const key = JSON.stringify(input);
  
  if (cache.has(key)) {
    return cache.get(key);
  }
  
  const result = await model.predict(input);
  cache.set(key, result);
  
  return result;
}
```

**4. Use edge computing for simple models:**
```javascript
// Run small models on client (TensorFlow.js)
// Send complex models to server only
// Reduce latency and server load

const model = await tf.loadLayersModel('model.json');
const clientPrediction = model.predict(input);
```

**5. Use GPU acceleration:**
```javascript
// For Node.js/Python backends
// TensorFlow with CUDA for GPU
// Significantly faster inference

// MongoDB Atlas vectorized search
// For semantic search without ML overhead
```

**Performance monitoring:**
```javascript
async function predictWithMetrics(input) {
  const start = Date.now();
  const result = await model.predict(input);
  const duration = Date.now() - start;
  
  metrics.recordInferenceTime(duration);
  
  return result;
}
```

---

### Q94: Explain vector databases and embeddings. Why are they important for AI applications?

**Answer:** Vector databases store high-dimensional vectors (embeddings), enabling semantic search and similarity matching.

**What are embeddings:**
```javascript
// Text converted to numerical vectors
"The cat sat on the mat" 
→ [0.1, 0.2, 0.3, ..., 0.9] (768 dimensions)

"A feline rested on a mat"
→ [0.11, 0.19, 0.31, ..., 0.88] (similar vectors)
```

**Using Pinecone (vector database):**
```bash
npm install pinecone-client
```

```javascript
import { Pinecone } from '@pinecone-database/pinecone';

const pc = new Pinecone({
  apiKey: process.env.PINECONE_API_KEY
});

const index = pc.Index('products');

// Store embeddings
await index.upsert([
  {
    id: '1',
    values: [0.1, 0.2, 0.3, ...], // embedding vector
    metadata: {
      title: 'Product 1',
      description: 'Best product'
    }
  }
]);

// Semantic search
const results = await index.query({
  vector: queryEmbedding,
  topK: 5,
  includeMetadata: true
});
```

**Creating embeddings with OpenAI:**
```javascript
import { OpenAIApi, Configuration } from 'openai';

const openai = new OpenAIApi(
  new Configuration({ apiKey: process.env.OPENAI_API_KEY })
);

async function getEmbedding(text) {
  const response = await openai.createEmbedding({
    model: 'text-embedding-3-small',
    input: text
  });
  
  return response.data.data[0].embedding;
}
```

**Use cases for AariyaTech:**
- **Semantic search**: Find similar documents
- **Recommendation**: Suggest related content
- **Duplicate detection**: Find similar user queries
- **Clustering**: Group similar AI outputs
- **RAG (Retrieval-Augmented Generation)**: Fetch relevant context for AI

**RAG Example (critical for AI applications):**
```javascript
// Retrieve relevant documents before generating
async function generateWithContext(query) {
  // 1. Create embedding of query
  const queryEmbedding = await getEmbedding(query);
  
  // 2. Search vector database for similar documents
  const relevantDocs = await vectorDb.search(queryEmbedding, topK: 5);
  
  // 3. Pass context to AI model
  const context = relevantDocs
    .map(doc => doc.metadata.content)
    .join('\n\n');
  
  const prompt = `
    Context: ${context}
    
    Question: ${query}
    
    Answer based on context:
  `;
  
  // 4. Generate response
  return await genAI.generateContent(prompt);
}
```

---

### Q95: How would you implement a chatbot with context memory?

**Answer:**

**Architecture:**
```
User Message
    ↓
Vector Embedding
    ↓
Retrieve Similar Messages (Vector DB)
    ↓
Build Context Window
    ↓
Send to AI Model
    ↓
Generate Response
    ↓
Store in Chat History
```

**Implementation:**
```javascript
// pages/api/chat.js
import { GoogleGenerativeAI } from '@google/generative-ai';
import { Pinecone } from '@pinecone-database/pinecone';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const pc = new Pinecone();
const index = pc.Index('chatbot-memories');

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end();
  
  const { userMessage, conversationId } = req.body;
  
  // Get chat history
  const chatHistory = await getChatHistory(conversationId);
  
  // Create embedding of user message
  const userEmbedding = await getEmbedding(userMessage);
  
  // Retrieve relevant past conversations
  const relevantContext = await index.query({
    vector: userEmbedding,
    topK: 3,
    filter: { conversationId }
  });
  
  // Build context
  const context = relevantContext.matches
    .map(m => m.metadata.content)
    .join('\n');
  
  // Build full conversation for model
  const messages = [
    ...chatHistory.map(msg => ({
      role: msg.role,
      content: msg.content
    })),
    { role: 'user', content: userMessage }
  ];
  
  // Generate response
  const model = genAI.getGenerativeModel({ model: 'gemini-pro' });
  const response = await model.generateContent({
    contents: messages
  });
  
  const assistantMessage = response.response.text();
  
  // Store messages in vector database
  await index.upsert([
    {
      id: `${conversationId}-${Date.now()}`,
      values: userEmbedding,
      metadata: {
        conversationId,
        role: 'user',
        content: userMessage,
        timestamp: Date.now()
      }
    }
  ]);
  
  // Store in regular database
  await saveChatMessage(conversationId, 'user', userMessage);
  await saveChatMessage(conversationId, 'assistant', assistantMessage);
  
  res.json({ message: assistantMessage });
}

async function getChatHistory(conversationId, limit = 10) {
  return await ChatMessage.find({ conversationId })
    .sort({ createdAt: -1 })
    .limit(limit)
    .sort({ createdAt: 1 });
}

async function getEmbedding(text) {
  // Use OpenAI or Supabase embeddings
  const response = await fetch('https://api.openai.com/v1/embeddings', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'text-embedding-3-small',
      input: text
    })
  });
  
  const data = await response.json();
  return data.data[0].embedding;
}
```

**Frontend:**
```javascript
'use client';

import { useState, useRef, useEffect } from 'react';

export default function ChatBot() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  const [conversationId] = useState(() => crypto.randomUUID());

  const handleSend = async () => {
    setLoading(true);
    setMessages(prev => [...prev, { role: 'user', content: input }]);
    setInput('');

    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ userMessage: input, conversationId })
    });

    const data = await response.json();
    setMessages(prev => [...prev, { role: 'assistant', content: data.message }]);
    setLoading(false);
  };

  return (
    <div className="chatbot">
      <div className="messages">
        {messages.map((msg, i) => (
          <div key={i} className={msg.role}>
            {msg.content}
          </div>
        ))}
      </div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && handleSend()}
        disabled={loading}
      />
      <button onClick={handleSend} disabled={loading}>
        {loading ? 'Sending...' : 'Send'}
      </button>
    </div>
  );
}
```

---

### Q96: How would you handle long-running background jobs in a Node.js application?

**Answer:** For tasks like AI processing, email sending, report generation.

**Using Celery (Python) or Bull (Node.js):**
```bash
npm install bull redis
```

**Queue setup:**
```javascript
// lib/queue.js
const Queue = require('bull');
const redis = require('redis');

export const aiProcessingQueue = new Queue('ai-processing', {
  redis: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT
  }
});

export const emailQueue = new Queue('emails', {
  redis: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT
  }
});
```

**Add job to queue:**
```javascript
// pages/api/generate-ai-output.js
import { aiProcessingQueue } from '../../lib/queue';

export default async function handler(req, res) {
  const { prompt, userId } = req.body;

  // Add job to queue
  const job = await aiProcessingQueue.add(
    {
      prompt,
      userId
    },
    {
      attempts: 3, // retry 3 times
      backoff: {
        type: 'exponential',
        delay: 2000
      },
      removeOnComplete: true
    }
  );

  res.json({ jobId: job.id, status: 'queued' });
}
```

**Process jobs:**
```javascript
// workers/aiProcessor.js
import { aiProcessingQueue } from '../lib/queue';

aiProcessingQueue.process(10, async (job) => {
  console.log(`Processing job ${job.id}`);
  
  const { prompt, userId } = job.data;
  
  try {
    // Long-running AI processing
    const result = await generateAIContent(prompt);
    
    // Update job progress
    job.progress(50);
    
    // Save result to database
    await saveResult({
      userId,
      prompt,
      result,
      status: 'completed'
    });
    
    job.progress(100);
    return result;
  } catch (error) {
    console.error('Job failed:', error);
    throw error; // Retry
  }
});

// Handle job events
aiProcessingQueue.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
  // Notify user (WebSocket, email, etc.)
});

aiProcessingQueue.on('failed', (job, err) => {
  console.log(`Job ${job.id} failed: ${err.message}`);
});
```

**Check job status:**
```javascript
// pages/api/job-status/[jobId].js
import { aiProcessingQueue } from '../../../lib/queue';

export default async function handler(req, res) {
  const { jobId } = req.query;
  const job = await aiProcessingQueue.getJob(jobId);
  
  if (!job) return res.status(404).json({ error: 'Job not found' });
  
  const progress = job.progress();
  const state = await job.getState();
  
  res.json({
    id: job.id,
    state,
    progress,
    data: job.data,
    result: job.returnvalue
  });
}
```

---

### Q97: How do you implement real-time notifications for AI processing completion?

**Answer:**

**WebSocket approach:**
```bash
npm install socket.io
```

**Server setup:**
```javascript
// pages/api/socket.js
import { Server } from 'socket.io';
import { aiProcessingQueue } from '../../lib/queue';

export default function handler(req, res) {
  if (!res.socket.server.io) {
    const io = new Server(res.socket.server);

    // Track active connections
    const userConnections = new Map();

    io.on('connection', (socket) => {
      socket.on('join', (userId) => {
        userConnections.set(userId, socket.id);
        console.log(`User ${userId} connected: ${socket.id}`);
      });

      socket.on('disconnect', () => {
        userConnections.forEach((socketId, userId) => {
          if (socketId === socket.id) {
            userConnections.delete(userId);
          }
        });
      });
    });

    // Listen to queue events
    aiProcessingQueue.on('progress', (job, progress) => {
      const socketId = userConnections.get(job.data.userId);
      if (socketId) {
        io.to(socketId).emit('progress', {
          jobId: job.id,
          progress
        });
      }
    });

    aiProcessingQueue.on('completed', (job, result) => {
      const socketId = userConnections.get(job.data.userId);
      if (socketId) {
        io.to(socketId).emit('completed', {
          jobId: job.id,
          result
        });
      }
    });

    res.socket.server.io = io;
  }

  res.end();
}
```

**Frontend:**
```javascript
'use client';

import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

export default function AIProcessingStatus({ userId, jobId }) {
  const [progress, setProgress] = useState(0);
  const [result, setResult] = useState(null);
  const [socket, setSocket] = useState(null);

  useEffect(() => {
    const newSocket = io();
    
    newSocket.on('connect', () => {
      newSocket.emit('join', userId);
    });

    newSocket.on('progress', (data) => {
      if (data.jobId === jobId) {
        setProgress(data.progress);
      }
    });

    newSocket.on('completed', (data) => {
      if (data.jobId === jobId) {
        setResult(data.result);
        newSocket.close();
      }
    });

    setSocket(newSocket);

    return () => newSocket.close();
  }, [userId, jobId]);

  return (
    <div>
      <div className="progress-bar">
        <div style={{ width: `${progress}%` }}>{progress}%</div>
      </div>
      
      {result && (
        <div className="result">
          <h2>Processing Complete!</h2>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

---

### Q98: How would you implement feature flags in a production application?

**Answer:** Feature flags allow enabling/disabling features without deploying code.

**Using a library like Unleash or LaunchDarkly:**
```bash
npm install flag-sdk
```

**Simple in-memory implementation:**
```javascript
// lib/features.js
const features = {
  'ai-generation': { enabled: true, rollout: 100 },
  'beta-dashboard': { enabled: true, rollout: 50 },
  'new-payment': { enabled: false, rollout: 0 }
};

export function isFeatureEnabled(featureName, userId) {
  const feature = features[featureName];
  
  if (!feature || !feature.enabled) return false;
  
  // Rollout percentage (e.g., 50% of users)
  const hash = hashUserId(userId);
  return (hash % 100) < feature.rollout;
}

function hashUserId(userId) {
  return userId.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0);
}
```

**Backend usage:**
```javascript
app.get('/api/features', (req, res) => {
  if (isFeatureEnabled('ai-generation', req.user.id)) {
    return res.json({ features: ['ai-generation', 'beta-dashboard'] });
  }
  res.json({ features: [] });
});
```

**Frontend usage:**
```javascript
'use client';

export default function Dashboard() {
  const [features, setFeatures] = useState([]);

  useEffect(() => {
    fetch('/api/features')
      .then(res => res.json())
      .then(data => setFeatures(data.features));
  }, []);

  return (
    <div>
      {features.includes('ai-generation') && <AIGenerator />}
      {features.includes('beta-dashboard') && <BetaDashboard />}
      <StandardDashboard />
    </div>
  );
}
```

**With database:**
```javascript
// Store in MongoDB for easy updates
const Feature = mongoose.model('Feature', {
  name: String,
  enabled: Boolean,
  rollout: Number, // 0-100
  targetUsers: [String],
  createdAt: Date,
  updatedAt: Date
});

app.get('/api/features', async (req, res) => {
  const enabledFeatures = await Feature.find({
    enabled: true,
    $or: [
      { rollout: 100 },
      { targetUsers: req.user.id }
    ]
  });
  
  res.json({ features: enabledFeatures.map(f => f.name) });
});

// Admin panel to toggle features
app.post('/api/admin/features/:name', async (req, res) => {
  const { enabled, rollout } = req.body;
  
  await Feature.findOneAndUpdate(
    { name: req.params.name },
    { enabled, rollout, updatedAt: Date.now() },
    { upsert: true }
  );
  
  res.json({ success: true });
});
```

**For AariyaTech projects:**
- Gradual rollout of AI features
- A/B testing new UI/UX
- Canary deployments
- Quick feature disabling if issues

---

### Q99: How do you implement error tracking and monitoring in production?

**Answer:** Essential for catching issues before users do.

**Using Sentry:**
```bash
npm install @sentry/nextjs
```

**Setup in Next.js:**
```javascript
// sentry.server.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0
});

// sentry.edge.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0
});

// sentry.client.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0
});
```

**Capturing errors:**
```javascript
// API route error handling
export default async function handler(req, res) {
  try {
    // code
  } catch (error) {
    Sentry.captureException(error, {
      tags: {
        section: 'auth',
        endpoint: '/api/login'
      },
      extra: {
        userId: req.user?.id,
        timestamp: new Date()
      }
    });
    
    res.status(500).json({ error: 'Server error' });
  }
}
```

**Custom logging:**
```javascript
// logger.js
import * as Sentry from '@sentry/nextjs';

export const logger = {
  error: (message, error, context = {}) => {
    Sentry.captureException(error, { extra: context });
    console.error(message, error, context);
  },
  
  info: (message, context = {}) => {
    Sentry.captureMessage(message, 'info');
    console.log(message, context);
  },
  
  warning: (message, context = {}) => {
    Sentry.captureMessage(message, 'warning');
    console.warn(message, context);
  }
};
```

**Performance monitoring:**
```javascript
// pages/api/expensive-operation.js
import * as Sentry from '@sentry/nextjs';

export default async function handler(req, res) {
  const transaction = Sentry.startTransaction({
    op: 'expensive-operation',
    name: '/api/expensive-operation'
  });

  try {
    const result = await heavyComputation();
    res.json(result);
  } finally {
    transaction.finish();
  }
}
```

**Frontend monitoring:**
```javascript
// pages/_app.js
import * as Sentry from '@sentry/nextjs';

function MyApp({ Component, pageProps }) {
  // Automatically captures uncaught errors
  return <Component {...pageProps} />;
}

export default Sentry.withProfiler(MyApp);
```

**Alternative: LogRocket**
```bash
npm install logrocket
```

```javascript
import LogRocket from 'logrocket';

LogRocket.init('app-id/key');

// Identifies users
LogRocket.identify('user-id', {
  name: 'User Name',
  email: 'user@example.com'
});

// Custom logging
LogRocket.log('User action', { action: 'purchase' });
```

---

### Q100: How do you approach security in a MERN stack application?

**Answer:** Security is critical, especially for AariyaTech (EdTech, FinTech).

**Authentication & Authorization:**
```javascript
// Use strong JWT secrets
const token = jwt.sign(
  { userId, role },
  process.env.JWT_SECRET, // 32+ character random string
  { expiresIn: '24h' }
);

// Verify on every protected route
export default withAuth((req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  // Handle request
});
```

**Input validation & sanitization:**
```javascript
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).trim().escape(),
  body('name').trim().escape()
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Process validated data
});
```

**HTTPS & CSP (Content Security Policy):**
```javascript
// next.config.js
module.exports = {
  headers: async () => [
    {
      source: '/(.*)',
      headers: [
        {
          key: 'Strict-Transport-Security',
          value: 'max-age=63072000; includeSubDomains'
        },
        {
          key: 'Content-Security-Policy',
          value: "default-src 'self'; script-src 'self' 'unsafe-inline'"
        }
      ]
    }
  ]
};
```

**Database security:**
```javascript
// Use parameterized queries
const user = await User.findOne({ email: email }); // Safe

// Never concatenate SQL strings
// BAD: `SELECT * FROM users WHERE email = '${email}'`
```

**API rate limiting & DDoS protection:**
```javascript
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  standardHeaders: true
});

app.use(limiter);
```

**Environment variables (no secrets in code):**
```javascript
// ✅ Good
const dbPassword = process.env.DB_PASSWORD;

// ❌ Bad
const dbPassword = 'myPassword123'; // Exposed!
```

**CORS configuration:**
```javascript
const cors = require('cors');
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE']
}));
```

**File upload security:**
```javascript
const storage = multer.diskStorage({
  destination: 'uploads/',
  filename: (req, file, cb) => {
    // Sanitize filename
    const sanitized = file.originalname
      .replace(/[^a-zA-Z0-9.-]/g, '')
      .substring(0, 50);
    cb(null, `${Date.now()}-${sanitized}`);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB max
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'application/pdf'];
    if (allowed.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});
```

**Logging sensitive operations:**
```javascript
logger.info('User login', {
  userId: user.id,
  email: user.email,
  timestamp: new Date(),
  ip: req.ip
});

// Store in secure audit log
await AuditLog.create({
  action: 'LOGIN',
  userId: user.id,
  timestamp: new Date(),
  ipAddress: req.ip,
  userAgent: req.headers['user-agent']
});
```

**For AariyaTech projects (especially FinTech/EdTech):**
- Regular security audits
- Penetration testing
- GDPR/Privacy compliance
- Data encryption (at rest and in transit)
- Regular security updates
- Two-factor authentication for critical operations

---

## Summary

This comprehensive set of 100 questions and answers covers:

1. **JavaScript/TypeScript Fundamentals** (15 Qs): Core language concepts essential for MERN
2. **React Fundamentals** (20 Qs): Component lifecycle, hooks, state management, optimization
3. **Next.js Fundamentals** (20 Qs): SSR/SSG, API routes, optimization, deployment
4. **Node.js & Express** (15 Qs): Backend development, routing, middleware, databases
5. **TypeScript & Practical Concepts** (15 Qs): Type system, forms, async/await
6. **Databases & AWS** (15 Qs): MongoDB, PostgreSQL, AWS services, optimization
7. **Practical & Soft Skills** (10 Qs): Debugging, code review, estimation, testing
8. **AI/ML & Advanced** (10 Qs): AI integration, embeddings, chatbots, monitoring

---

### Preparation Tips for AariyaTech Interview:

1. **Practice coding**: Use LeetCode, HackerRank for algorithm practice
2. **Build projects**: Create portfolio projects using MERN + AI/ML
3. **Understand architecture**: Study how to design scalable systems
4. **Learn their stack**: Their tech focus on AI, spatial computing, green impact
5. **Soft skills**: Be enthusiastic about learning, contribute to team, ask good questions
6. **Ask questions**: Show interest in their projects and tech decisions

**Good luck with your interview at AariyaTech! 🚀**
