# React & Frontend – Interview Questions & Answers

**Question**: How does the component lifecycle differ between class components and the Hooks era?

**Answer**: Class components expose lifecycle methods — `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` — which split logic by timing, not concern. Hooks unify these into `useEffect`, letting you group related logic (e.g., subscription setup/teardown) in one place. The mental model shifts from "mount/update/unmount" to "synchronizing with state/props."

```javascript
// Class component
componentDidMount() { this.subscribe(this.props.id); }
componentDidUpdate(prevProps) {
  if (prevProps.id !== this.props.id) {
    this.unsubscribe();
    this.subscribe(this.props.id);
  }
}
componentWillUnmount() { this.unsubscribe(); }

// Hooks equivalent
useEffect(() => {
  subscribe(id);
  return () => unsubscribe();
}, [id]);
```

---

**Question**: How does JSX get transpiled, and what is the distinction between expressions and statements in JSX?

**Answer**: JSX is syntactic sugar for `React.createElement` calls — Babel compiles `<div className="x" />` into `React.createElement('div', { className: 'x' })`. Inside JSX `{}` you can embed any JavaScript expression (something that produces a value) but not statements (e.g., `if`, `for`, `switch`). Ternary operators and logical `&&` are common expression-based conditionals.

```javascript
// Transpiles to: React.createElement('h1', null, 'Hello')
const el = <h1>Hello</h1>;

// Expression OK; statement NOT OK
const greeting = user ? <h1>Hi {user.name}</h1> : <h1>Hi Guest</h1>;
// ❌ Cannot use if/else inside {}
```

---

**Question**: What is the difference between state and props in React, and why is immutability important?

**Answer**: Props are read-only data passed from a parent; state is owned and mutable within a component via `setState` or the `useState` setter. Immutability ensures React can detect changes efficiently with shallow reference checks — mutating state directly skips re-rendering because the object reference hasn't changed.

```javascript
// ❌ Wrong — mutating state directly
const [items, setItems] = useState([1, 2]);
items.push(3); // React won't re-render

// ✅ Correct — returning a new reference
setItems(prev => [...prev, 3]);
```

---

**Question**: Explain controlled vs uncontrolled components and when to use each.

**Answer**: Controlled components derive their value from state, with every input change dispatching a state update — giving React full control. Uncontrolled components store their value internally in the DOM, accessed via a ref. Use controlled for validation, instant feedback, or libraries like Formik; use uncontrolled for simple inputs, ref-based access, or libraries like React Hook Form.

```javascript
// Controlled — value driven by state
function ControlledInput() {
  const [val, setVal] = useState('');
  return <input value={val} onChange={e => setVal(e.target.value)} />;
}

// Uncontrolled — ref to read value on demand
function UncontrolledInput() {
  const ref = useRef();
  return <input ref={ref} defaultValue="initial" />;
}
```

---

**Question**: How does React use keys during reconciliation, and why is the array index an anti-pattern?

**Answer**: Keys help React identify which items changed, were added, or removed during the diffing phase — a stable key preserves component identity across re-renders. Index as a key causes issues when items are reordered, inserted, or deleted because React may reuse stale state or mismatched DOM nodes, leading to visual bugs or performance degradation.

```javascript
// ❌ Anti-pattern — index as key
{items.map((item, index) => <li key={index}>{item.text}</li>)}

// ✅ Stable, unique key
{items.map(item => <li key={item.id}>{item.text}</li>)}
```

---

**Question**: What are the different patterns for using refs in React — `useRef`, `forwardRef`, and callback refs?

**Answer**: `useRef` holds a mutable value that persists across renders without causing re-renders, commonly used for DOM access. `forwardRef` lets a parent pass a ref through to a child's DOM node. Callback refs give finer control — the callback fires both when the ref is attached and detached, useful for dynamic ref management or measuring nodes.

```javascript
function AutoFocusInput() {
  const ref = useRef(null);
  useEffect(() => { ref.current?.focus(); }, []);
  return <input ref={ref} />;
}

const FancyButton = forwardRef((props, ref) => (
  <button ref={ref} className="fancy">{props.children}</button>
));

// Callback ref — fires on mount and unmount
<div ref={el => console.log('Attached:', el)} />
```

---

**Question**: How do `useState` lazy initializers and functional updates work?

**Answer**: The lazy initializer (`useState(() => expensiveComputation())`) runs once on mount, avoiding repeated work on every render. The functional updater (`setCount(prev => prev + 1)`) uses the previous state to compute the next value, which is essential when updates are batched or called in quick succession (e.g., inside a loop or `setInterval`).

```javascript
// Lazy initializer — runs once
const [items] = useState(() => {
  const saved = localStorage.getItem('items');
  return saved ? JSON.parse(saved) : [];
});

// Functional update — safe even with stale closures
const [count, setCount] = useState(0);
useEffect(() => {
  const id = setInterval(() => setCount(prev => prev + 1), 1000);
  return () => clearInterval(id);
}, []);
```

---

**Question**: Explain the `useEffect` dependency array, cleanup function, and common mount/unmount patterns.

**Answer**: The dependency array controls when the effect re-runs — empty `[]` runs once on mount (and cleanup on unmount), omitted runs on every render. The cleanup function (returned from the effect) handles subscription teardown, timer clearing, or aborting fetch requests, preventing memory leaks and stale side effects.

```javascript
// Mount / unmount pattern
useEffect(() => {
  const ws = new WebSocket(url);
  return () => ws.close(); // cleanup on unmount or deps change
}, [url]);

// Skipping effect on initial render
const didMount = useRef(false);
useEffect(() => {
  if (!didMount.current) { didMount.current = true; return; }
  // run on updates only
}, [dep]);
```

---

**Question**: How does `useContext` work with the Provider pattern, and what are the performance implications?

**Answer**: `useContext` consumes a context created by `React.createContext` and returns its current value from the nearest matching `Provider` above in the tree. Every time the context value changes, **all** consumers re-render — even if they only read a subset of the value. Mitigate by splitting contexts, using `useMemo` on the provider value, or selecting slices with custom hooks.

```javascript
const ThemeContext = React.createContext('light');

function App() {
  const [theme, setTheme] = useState('dark');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return (
    <ThemeContext.Provider value={value}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const { theme } = useContext(ThemeContext);
  return <div className={theme}>Toolbar</div>;
}
```

---

**Question**: When should you use `useMemo` and `useCallback`, and what are the tradeoffs?

**Answer**: `useMemo` caches a computed value until its dependencies change; `useCallback` caches a function reference. Use them to avoid expensive recalculations or to preserve referential stability for child components wrapped in `React.memo`. Overusing them adds memory overhead and complexity — profile first and only memoize when you measure a bottleneck.

```javascript
// useMemo — avoids recalculating sorted list on every render
const sorted = useMemo(() => 
  items.toSorted((a, b) => a.name.localeCompare(b.name)), 
[items]);

// useCallback — stable reference for React.memo children
const handleClick = useCallback(() => {
  setCount(prev => prev + 1);
}, []);
```

---

**Question**: How does `useReducer` help manage complex state, and when would you model state as a state machine?

**Answer**: `useReducer` centralizes state transitions in a reducer function, making complex state logic (multiple sub-values, interdependent updates) predictable and testable. Modeling state as a finite state machine (loading → success → error) with `useReducer` prevents impossible states and reduces bugs by explicitly defining all allowed transitions.

```javascript
const initialState = { status: 'idle', data: null, error: null };

function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START': return { ...state, status: 'loading' };
    case 'FETCH_SUCCESS': return { status: 'success', data: action.payload, error: null };
    case 'FETCH_ERROR': return { status: 'error', data: null, error: action.error };
    default: return state;
  }
}

function DataFetcher({ url }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  // dispatch({ type: 'FETCH_START' }) etc.
}
```

---

**Question**: What makes a good custom hook, and how do you compose hooks for reuse?

**Answer**: A custom hook extracts reusable stateful logic — it can call other hooks, manage local state, and return values or functions. Compose smaller hooks (e.g., `useFetch`, `useDebounce`) inside larger ones to build abstractions without repeating side-effect patterns. Custom hooks should follow the naming convention `use*` and have a single responsibility.

```javascript
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}

function useSearch(query) {
  const debounced = useDebounce(query, 300);
  return useFetch(`/api/search?q=${debounced}`);
}
```

---

**Question**: What are the Rules of Hooks, and what problems do they prevent?

**Answer**: Hooks must be called at the top level of a component (not inside loops, conditions, or nested functions) and in the same order on every render. These rules ensure that React's internal linked-list of hook states remains consistent across renders — violating them leads to stale state, wrong hook associations, or infinite loops.

```javascript
// ❌ Violation — hook inside a condition
function Bad({ flag }) {
  if (flag) {
    const [x, setX] = useState(0); // Reorder on next render!
  }
}

// ✅ Always call hooks unconditionally at top level
function Good({ flag }) {
  const [x, setX] = useState(0);
  useEffect(() => { if (flag) setX(1); }, [flag]);
}
```

---

**Question**: How does lifting state up work, and when is it preferable to Context or a store?

**Answer**: Lifting state up means moving shared state to the lowest common ancestor and passing it down via props. It's the simplest solution for sharing state between siblings or a parent and a distant child when the component tree is shallow. Prefer it over Context or a store when only 2–3 levels of nesting are involved, as it keeps data flow explicit and avoids unnecessary re-renders.

```javascript
function Parent() {
  const [value, setValue] = useState('');
  return (
    <>
      <Input value={value} onChange={setValue} />
      <Display value={value} />
    </>
  );
}
```

---

**Question**: How do you combine Context with `useReducer` for state management, and what are the drawbacks?

**Answer**: Wrap a `useReducer` in a Context provider to expose a global-like state and dispatch to any descendant. This pattern works well for medium-complexity apps (auth, theme, shopping cart) without external libraries. The drawback is that any context consumer re-renders when the state changes, regardless of which slice it reads — splitting contexts by domain mitigates this.

```javascript
const AuthContext = React.createContext();

function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, initialState);
  const value = useMemo(() => ({ state, dispatch }), [state]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be inside AuthProvider');
  return ctx;
}
```

---

**Question**: Compare Redux and Zustand for global state management.

**Answer**: Both provide a global store with actions and reducers, but Redux enforces immutability via reducers and a single store with middleware (thunks, sagas), while Zustand offers a simpler API with mutable updates via `set` and no boilerplate. Redux scales better for large teams with strict patterns; Zustand is preferred for smaller teams or projects where minimal setup is desired.

```javascript
// Redux slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: { increment: state => { state.value += 1; } }
});

// Zustand store
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));
```

---

**Question**: How do React Query (TanStack Query) and SWR handle server state — caching, revalidation, and optimistic updates?

**Answer**: Both libraries cache server responses in a key-value store, automatically revalidate on focus, interval, or mutation. They provide stale-while-revalidate semantics: show cached data immediately, then refetch in the background. Optimistic updates apply mutations to the cache instantly and roll back on error, giving the user immediate feedback.

```javascript
// React Query — optimistic update
const mutation = useMutation({
  mutationFn: (newTodo) => api.post('/todos', newTodo),
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    const previous = queryClient.getQueryData(['todos']);
    queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
    return { previous };
  },
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(['todos'], context.previous);
  },
});
```

---

**Question**: Explain React's reconciliation algorithm and how the Fiber architecture changed it.

**Answer**: Reconciliation is React's process of diffing the virtual DOM against the previous tree to determine DOM mutations. Fiber (React 16+) broke this work into incremental units that can be paused, prioritized, or aborted — enabling concurrent features like Suspense and transitions. The diff is O(n) by assuming elements of the same type produce similar trees and keys identify stable siblings.

```javascript
// Fiber enables interruption:
// render phase (can be paused) → commit phase (DOM mutations, synchronous)
// Prev tree: <div><A/><B/></div>
// Next tree: <div><A/><C/></div>
// React reuses the <div> and <A/> nodes, unmounts <B/>, mounts <C/>
```

---

**Question**: How does the Virtual DOM work with batching and keyed updates?

**Answer**: The Virtual DOM is a lightweight JavaScript representation of the real DOM. React batches state updates within event handlers and lifecycle methods, coalescing multiple `setState` calls into a single re-render. During reconciliation, keys tell React whether a node moved, was added, or removed, avoiding costly re-creation of components that merely changed position.

```javascript
// Batching — both updates happen together (React 18+ automatic batching)
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // Single re-render, not two
}

// Keyed updates — moving an item keeps its state
{items.map(item => <ListItem key={item.id} />)}
```

---

**Question**: What does `React.memo` do, and how does shallow comparison work?

**Answer**: `React.memo` is a higher-order component that memoizes the render output — it re-renders only if props change according to a shallow comparison (=== for each prop). It's useful for pure components that re-render often with the same props. Custom comparison functions can be passed as the second argument to override shallow equality.

```javascript
const ExpensiveList = React.memo(({ items }) => {
  return items.map(item => <li key={item.id}>{item.name}</li>);
});

// Custom comparison
const List = React.memo(
  ({ items }) => <ul>{items.map(i => <li key={i.id}>{i.name}</li>)}</ul>,
  (prev, next) => prev.items.length === next.items.length
    && prev.items.every((item, i) => item.id === next.items[i].id && item.name === next.items[i].name)
);
```

---

**Question**: How do you profile `useMemo` and `useCallback` usage with React DevTools Profiler?

**Answer**: Record a flamegraph in the React DevTools Profiler to see component render durations and commit information. Components that render often with the same props are candidates for `React.memo`. The Profiler also shows why a component re-rendered (props changed, state changed, context changed, parent re-rendered), helping you decide whether `useMemo`/`useCallback` actually prevent wasted work.

```
React DevTools Profiler workflow:
1. Start profiling
2. Interact with the UI
3. Inspect commits — look for components that rendered
   but whose props/state didn't meaningfully change
4. Apply useMemo/useCallback only where rendering cost > memoization cost
```

---

**Question**: How does code splitting work with `React.lazy`, `Suspense`, and dynamic imports?

**Answer**: `React.lazy` wraps a dynamic `import()` to lazily load a component only when it's rendered. `Suspense` provides a fallback UI (e.g., a spinner) while the chunk loads. This reduces the initial bundle size by splitting code at route boundaries or heavy component boundaries. Named exports require a re-export module because `React.lazy` only supports default exports.

```javascript
// Dynamic import — Webpack/Rollup creates a separate chunk
const HeavyChart = React.lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  );
}
```

---

**Question**: Compare SSR, SSG, ISR, and CSR — what are the tradeoffs for SEO, bundle size, and first paint?

**Answer**: CSR sends an empty HTML shell; everything renders client-side — fast subsequent navigation but slow first paint and poor SEO. SSR (Next.js `getServerSideProps`) renders HTML per-request — good SEO, slower TTFB. SSG (`getStaticProps`) pre-builds HTML at build time — fastest TTFB but stale data. ISR combines SSG with periodic revalidation, serving stale pages then updating. Choose based on content freshness needs and traffic patterns.

```javascript
// SSG — pre-built at build time
export async function getStaticProps() {
  const data = await fetch('https://api.example.com/posts').then(r => r.json());
  return { props: { data } };
}

// SSR — per-request
export async function getServerSideProps(context) {
  const data = await fetch(`https://api.example.com/posts/${context.params.id}`).then(r => r.json());
  return { props: { data } };
}

// ISR — revalidate every 60 seconds
export async function getStaticProps() {
  const data = await fetch('https://api.example.com/posts').then(r => r.json());
  return { props: { data }, revalidate: 60 };
}
```

*Note: In Next.js 13+ App Router, SSR is the default for Server Components; SSG is achieved with `dynamic = 'force-static'`; ISR uses `revalidate` in `fetch()` calls.*

---

**Question**: What causes hydration mismatch errors, and how does progressive hydration work?

**Answer**: Hydration mismatches occur when the server-rendered HTML differs from the client's first render — caused by browser-only APIs (`window`), non-deterministic data, or timestamp-dependent content. Selective hydration (React 18) uses Suspense boundaries to prioritize which components hydrate first — interactive components (e.g., click handlers) hydrate before static content, enabling interactivity without waiting for the full page to hydrate.

```javascript
// ❌ Mismatch — server renders one value, client another
function Time() {
  return <div>{new Date().toISOString()}</div>;
}

// ✅ Fix — wait for client to match
function Time() {
  const [time, setTime] = useState('');
  useEffect(() => setTime(new Date().toISOString()), []);
  if (!time) return <div suppressHydrationWarning />;
  return <div>{time}</div>;
}
```

---

**Question**: How does Streaming SSR work in React 18 with Suspense boundaries?

**Answer**: Streaming SSR lets the server send HTML in chunks as components become ready, rather than waiting for the full tree. Each `<Suspense>` boundary creates a shell that can be streamed early, with fallback content replaced later by the actual component HTML and its hydration scripts. This improves Time to First Byte (TTFB) and First Contentful Paint (FCP).

```javascript
// Server sends the shell immediately, streams <Comments> later
<Layout>
  <Nav />
  <Suspense fallback={<Spinner />}>
    <Comments />  {/* streamed after data resolves */}
  </Suspense>
</Layout>
```

---

**Question**: What are the core APIs of React Testing Library — `render`, `screen`, `fireEvent`, `userEvent` — and how do you prioritize queries?

**Answer**: `render` mounts a component into a test DOM environment. `screen` provides global query access without destructuring. `userEvent` simulates realistic interactions (clicks, typing) and should be preferred over `fireEvent` because it dispatches higher-level events closer to user behavior. Query priority follows the Testing Library guiding principle: getByRole > getByLabelText > getByText > getByDisplayValue > getByAltText > getByTestId.

```javascript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('submits form with username', async () => {
  const user = userEvent.setup();
  render(<LoginForm />);
  await user.type(screen.getByLabelText(/username/i), 'john');
  await user.click(screen.getByRole('button', { name: /submit/i }));
  expect(await screen.findByText(/welcome, john/i)).toBeInTheDocument();
});
```

---

**Question**: How do you approach component vs integration tests, and what is the role of mocking API calls?

**Answer**: Component tests focus on behavior (not implementation) — test what the user sees and does, not internal state or method calls. Integration tests exercise a full user flow across multiple components, mocking API calls at the network layer (MSW) rather than mocking individual modules. This ensures the test breaks only when user-facing behavior changes, not when implementation details are refactored.

```javascript
// MSW — mock at the network level, not module level
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.get('/api/user', () => {
    return HttpResponse.json({ name: 'Alice' });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('displays user name', async () => {
  render(<UserProfile />);
  expect(await screen.findByText(/alice/i)).toBeInTheDocument();
});
```

---

**Question**: What are the key differences between Next.js Pages Router and App Router, and what are Server Components vs Client Components?

**Answer**: The Pages Router uses file-system routing with `getServerSideProps`/`getStaticProps` for data fetching. App Router (Next.js 13+) uses a nested layout system with `page.js`, `layout.js`, and `loading.js` conventions. Server Components run and render on the server, reducing client JS — they cannot use hooks or browser APIs. Client Components (marked with `'use client'`) hydrate on the browser and support interactivity.

```javascript
// App Router — Server Component (default)
// app/page.tsx
async function Page() {
  const data = await fetch('https://api.example.com/data');
  const json = await data.json();
  return <div>{json.title}</div>;
}

// Client Component — opt in with 'use client'
// app/Counter.tsx
'use client';
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

**Question**: How do Remix loaders/actions and React Router data APIs differ from Next.js data fetching?

**Answer**: Remix uses `loader` (GET data) and `action` (POST/PUT/DELETE mutations) exported from a route module — data is fetched on the server and the page re-renders after actions. React Router 6.4+ introduced route `loader` and `action` with `useLoaderData`, enabling data fetching co-located with routes. Both Remix and React Router embrace web fundamentals (Form, fetch), while Next.js uses `getServerSideProps` or Server Components with `async` functions.

```javascript
// Remix route
export async function loader() {
  return json(await db.getPosts());
}

export async function action({ request }) {
  const formData = await request.formData();
  await db.createPost({ title: formData.get('title') });
  return redirect('/posts');
}

export default function Posts() {
  const posts = useLoaderData<typeof loader>();
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

---

**Question**: How do you type React components, hooks, and generic components in TypeScript?

**Answer**: Use function declarations with explicit prop types (preferred) or `React.FC` for components, generic type parameters for polymorphic components, and utility types like `React.PropsWithChildren` or `ComponentPropsWithoutRef` for extending native elements. Hooks benefit from typed state and custom return types. Generic components infer types from usage, improving type safety without explicit annotations.

```typescript
// Typed props
interface ButtonProps {
  label: string;
  variant: 'primary' | 'secondary';
  onClick: () => void;
}

function Button({ label, variant, onClick }: ButtonProps) {
  return <button className={variant} onClick={onClick}>{label}</button>;
}

// Generic component
import { ReactNode } from 'react';

function List<T>({ items, render }: { items: T[]; render: (item: T) => ReactNode }) {
  return <ul>{items.map(render)}</ul>;
}

// Usage — T is inferred
<List items={[1, 2, 3]} render={item => <li>{item.toFixed(2)}</li>} />

// Typed hook
function useLocalStorage<T>(key: string, initial: T): [T, (v: T) => void] {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initial;
  });
  const set = useCallback((v: T) => {
    setValue(v);
    localStorage.setItem(key, JSON.stringify(v));
  }, [key]);
  return [value, set];
}
```

---

**Question**: How does Vite's HMR and Fast Refresh differ from webpack's live reload?

**Answer**: Vite serves native ES modules in development — no bundling required — so HMR updates are instant regardless of project size. It leverages esbuild for pre-bundling dependencies and native ESM for source files. Fast Refresh preserves component state across edits by replacing modules without a full page reload, using React's runtime to re-render only affected components. webpack's HMR re-compiles the changed module and its affected dependents, which degrades as the project grows.

```
Vite Dev Flow:
  Browser imports ESM directly → Server sends patched modules on change
  → React Fast Refresh replaces component
  → State preserved, no full reload

webpack Dev Flow:
  Webpack re-compiles affected modules → Sends updated bundle
  → HMR runtime evaluates new modules → Full re-render often required
```

---

**Question**: How do React Router nested routes and layouts work?

**Answer**: Nested routes in React Router 6+ use `<Outlet />` as a placeholder where child routes render inside a parent layout. The parent route's element wraps all children, enabling shared UI (headers, sidebars, data providers). Each route can have its own `loader` for data fetching, and `useLoaderData` accesses it in the component — layout data and child data are independently fetched and scoped.

```javascript
// Route config
const router = createBrowserRouter([
  {
    path: '/dashboard',
    element: <DashboardLayout />,  // shared layout
    loader: () => fetch('/api/dashboard'),
    children: [
      { index: true, element: <Overview /> },
      {
        path: 'settings',
        element: <Settings />,
        loader: () => fetch('/api/settings'),
      },
    ],
  },
]);

function DashboardLayout() {
  const data = useLoaderData();
  return (
    <div>
      <Sidebar data={data} />
      <Outlet />  {/* child route renders here */}
    </div>
  );
}
```
