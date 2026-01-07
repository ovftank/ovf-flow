# Framework Debug Guide

Hướng dẫn debug chi tiết cho các JavaScript frameworks phổ biến.

## React

### 1. useEffect Issues

#### Problem: Effect chạy quá nhiều lần hoặc không chạy

**Symptoms:**

- Infinite loop
- Effect không trigger khi dependency thay đổi
- Stale data trong effect

**Root Causes:**

1. **Wrong dependency array**

```javascript
// WRONG - Missing dependency
useEffect(() => {
  fetchData(userId);
}, []); // userId không trong dependencies

// CORRECT - Include all dependencies
useEffect(() => {
  fetchData(userId);
}, [userId]);

// OR disable ESLint rule with comment if intentional
useEffect(() => {
  fetchData(userId);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []); // Only run on mount
```

1. **Object/Array dependency causing infinite loop**

```javascript
// WRONG - Object dependency tạo reference mới mỗi render
useEffect(() => {
  processObject(config);
}, [config]);

// SOLUTION 1: useMemo cho object
const memoizedConfig = useMemo(() => config, [config.id, config.value]);
useEffect(() => {
  processObject(memoizedConfig);
}, [memoizedConfig]);

// SOLUTION 2: Chỉ lấy properties cần thiết
useEffect(() => {
  processObject(config);
}, [config.id, config.value]);

// SOLUTION 3: Dùng JSON.stringify cho simple objects
useEffect(() => {
  processObject(config);
}, [JSON.stringify(config)]);
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Trace effect runs
useEffect(() => {
  console.log('[DEBUG] Effect triggered', {
    timestamp: new Date().toISOString(),
    dependencies: { userId, config }
  });
}, [userId, config]);

// TODO:DEBUG - Check if effect runs at all
useEffect(() => {
  console.log('[DEBUG] Effect mounted');
  return () => {
    console.log('[DEBUG] Effect unmounted');
  };
}, []);
```

#### Problem: Stale Closure trong useEffect

**Symptoms:**

- State/prop values cũ được capture
- Click handlers dùng outdated state
- Timer/interval bị stuck với old values

**Solution:**

```javascript
// WRONG - Stale closure
useEffect(() => {
  const interval = setInterval(() => {
    console.log(count); // Luôn là giá trị ban đầu
  }, 1000);
  return () => clearInterval(interval);
}, []);

// CORRECT 1: Functional update
useEffect(() => {
  const interval = setInterval(() => {
    setCount(c => c + 1); // Dùng callback form
  }, 1000);
  return () => clearInterval(interval);
}, []);

// CORRECT 2: Add dependencies
useEffect(() => {
  const interval = setInterval(() => {
    console.log(count); // Sẽ update khi count thay đổi
  }, 1000);
  return () => clearInterval(interval);
}, [count]);

// CORRECT 3: useRef cho mutable values
const countRef = useRef(count);
useEffect(() => {
  countRef.current = count;
});
useEffect(() => {
  const interval = setInterval(() => {
    console.log(countRef.current); // Luôn latest value
  }, 1000);
  return () => clearInterval(interval);
}, []);
```

### 2. useState Issues

#### Problem: State không update ngay lập tức

**Symptoms:**

- Logging ngay sau setState shows old value
- Multiple setState calls chỉ apply cuối cùng
- State updates bị batch

**Solution:**

```javascript
// WRONG - Logging ngay sau setState
const handleClick = () => {
  setState(newValue);
  console.log(state); // Old value!
};

// CORRECT - Dùng useEffect để log sau khi update
useEffect(() => {
  console.log(state); // Updated value
}, [state]);

// CORRECT - Dùng functional update để tránh stale state
const handleClick = () => {
  setState(prev => prev + 1);
  setState(prev => prev + 1);
  // Sẽ increment 2 lần
};

// WRONG - Multiple setState calls bị overwrite
const handleClick = () => {
  setState(count + 1);
  setState(count + 1); // Still increment 1 lần
};
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Log state changes
useEffect(() => {
  console.log('[DEBUG] State changed:', {
    count,
    previousValue: usePrevious(count)
  });
}, [count]);

// Custom hook cho previous value
const usePrevious = (value) => {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
};
```

#### Problem: Object state merge không work

**Symptoms:**

- Updating object property loses other properties
- Spread operator không hoạt động như expected

**Solution:**

```javascript
// WRONG - Direct assignment loses other properties
const updateName = (name) => {
  setUser({ name }); // Mất age, email, etc.
};

// CORRECT - Spread existing state
const updateName = (name) => {
  setUser(prev => ({ ...prev, name }));
};

// CORRECT - Dùng useReducer cho complex state
const userReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_NAME':
      return { ...state, name: action.payload };
    case 'UPDATE_AGE':
      return { ...state, age: action.payload };
    default:
      return state;
  }
};
```

### 3. Event Handler Issues

#### Problem: Event listeners with stale state

**Symptoms:**

- Event handlers dùng outdated state
- Cleanup functions capture old values
- Multiple event listeners bị registered

**Solution:**

```javascript
// WRONG - Stale closure
useEffect(() => {
  const handleMove = (e) => {
    if (canMove) { // canMove luôn là giá trị ban đầu
      setPosition({ x: e.clientX, y: e.clientY });
    }
  };
  window.addEventListener('pointermove', handleMove);
  return () => window.removeEventListener('pointermove', handleMove);
}, []); // Missing canMove dependency

// CORRECT 1: Add dependency
useEffect(() => {
  const handleMove = (e) => {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  };
  window.addEventListener('pointermove', handleMove);
  return () => window.removeEventListener('pointermove', handleMove);
}, [canMove]);

// CORRECT 2: useRef cho mutable flag
const canMoveRef = useRef(canMove);
useEffect(() => {
  canMoveRef.current = canMove;
}, [canMove]);

useEffect(() => {
  const handleMove = (e) => {
    if (canMoveRef.current) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  };
  window.addEventListener('pointermove', handleMove);
  return () => window.removeEventListener('pointermove', handleMove);
}, []); // Không cần dependencies
```

### 4. Performance Issues

#### Problem: Unnecessary re-renders

**Symptoms:**

- Component re-render khi parent re-render
- Expensive calculations chạy mỗi render
- List items re-render khi list thay đổi

**Solution:**

```javascript
// SOLUTION 1: React.memo cho component
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* expensive rendering */}</div>;
});

// SOLUTION 2: useMemo cho expensive calculations
const ExpensiveCalculation = ({ items }) => {
  const sorted = useMemo(() => {
    return items.sort((a, b) => a.value - b.value);
  }, [items]);

  return <div>{sorted.map(...)}</div>;
};

// SOLUTION 3: useCallback cho event handlers
const ParentComponent = () => {
  const handleClick = useCallback((id) => {
    // Handler logic
  }, []); // Dependencies array

  return <ChildComponent onClick={handleClick} />;
};

// SOLUTION 4: Keys cho lists
const ListComponent = ({ items }) => {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
        // NOT: key={index}
      ))}
    </ul>
  );
};
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Track renders
const renderCount = useRef(0);
useEffect(() => {
  renderCount.current += 1;
  console.log('[DEBUG] Component rendered:', renderCount.current);
});

// TODO:DEBUG - Check why component re-rendered
useEffect(() => {
  console.log('[DEBUG] Props changed:', { prop1, prop2, prop3 });
}, [prop1, prop2, prop3]);
```

### 5. Context Issues

#### Problem: Context updates causing unnecessary re-renders

**Symptoms:**

- All consumers re-render khi context value thay đổi
- Performance issues với large context trees
- Deep updates cause full re-render

**Solution:**

```javascript
// WRONG - Single context cho tất cả
const AppContext = createContext();
const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [settings, setSettings] = useState({});

  // Mọi consumer re-render khi BẤT KỲ value thay đổi
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme, settings, setSettings }}>
      {children}
    </AppContext.Provider>
  );
};

// CORRECT - Split contexts
const UserContext = createContext();
const ThemeContext = createContext();
const SettingsContext = createContext();

// Chỉ consumers cần thiết sẽ re-render
const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

// CORRECT 2: Dùng useMemo cho context value
const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  const value = useMemo(() => ({ user, setUser }), [user]);
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
};
```

## Next.js

### 1. Server vs Client Components

#### Problem: "use client" missing hoặc wrong

**Symptoms:**

- "Window is not defined" error
- Hydration failed
- Interactivity không work

**Solution:**

```typescript
// Server Component (default)
const Page = () => {
  return <div>{/* Server-side only */}</div>;
};

export default Page;

// Client Component (needs 'use client')
'use client';

const InteractiveComponent = () => {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
};

export default InteractiveComponent;

// CORRECT - Import Client Component vào Server Component
// components/InteractiveComponent.tsx
'use client';

const InteractiveComponent = () => { /* ... */ };
export { InteractiveComponent };

// app/page.tsx (Server Component)
import { InteractiveComponent } from './components/InteractiveComponent';

const Page = () => {
  return (
    <div>
      <h1>Server Rendered</h1>
      <InteractiveComponent />
    </div>
  );
};

export default Page;
```

### 2. Data Fetching

#### Problem: Data not loading hoặc caching issues

**Symptoms:**

- Stale data
- Loading states không work
- Error handling missing

**Solution:**

```typescript
// Server-side fetching (Server Component)
const getUserData = async () => {
  const res = await fetch('https://api.example.com/user', {
    // Force fresh data every request
    cache: 'no-store'
    // OR cache for specific duration
    // next: { revalidate: 60 }
  });

  if (!res.ok) {
    throw new Error('Failed to fetch data');
  }

  return res.json();
};

const Page = async () => {
  const user = await getUserData();

  return <div>{user.name}</div>;
};

export default Page;

// Client-side fetching (Client Component)
'use client';

import useSWR from 'swr';

const UserProfile = () => {
  const { data, error, isLoading } = useSWR(
    '/api/user',
    fetcher,
    {
      revalidateOnFocus: false,
      dedupingInterval: 60000
    }
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading user</div>;

  return <div>{data.name}</div>;
};

export { UserProfile };
```

### 3. Route Transitions

#### Problem: Page navigation không smooth

**Symptoms:**

- Flash of unstyled content
- Lost scroll position
- Loading states missing

**Solution:**

```typescript
// app/page.tsx
import Link from 'next/link';

const Page = () => {
  return (
    <Link
      href="/dashboard"
      className="transition-all"
    >
      Go to Dashboard
    </Link>
  );
};

export default Page;

// Programmatic navigation
'use client';

import { useRouter } from 'next/navigation';

const NavigationButton = () => {
  const router = useRouter();

  const handleClick = () => {
    router.push('/dashboard');
    // OR with options
    router.push('/dashboard', { scroll: false });
  };

  return <button onClick={handleClick}>Go</button>;
};

export { NavigationButton };

// Loading UI
// app/loading.tsx
const Loading = () => {
  return <div>Loading...</div>;
};

export default Loading;
```

## Common Debug Patterns

### React

```javascript
// TODO:DEBUG - Component mount/unmount
useEffect(() => {
  console.log('[DEBUG] Component mounted');
  return () => {
    console.log('[DEBUG] Component unmounted');
  };
}, []);

// TODO:DEBUG - Props/State changes
useEffect(() => {
  console.log('[DEBUG] Props/State changed:', { prop1, prop2, state1 });
}, [prop1, prop2, state1]);

// TODO:DEBUG - Custom hook debugging
const useDebug = (label, value) => {
  useEffect(() => {
    console.log(`[DEBUG] ${label}:`, value);
  }, [label, value]);
};
```

### Next.js

```typescript
// TODO:DEBUG - Server vs Client component
console.log('[DEBUG] Component type:', typeof window !== 'undefined' ? 'Client' : 'Server');

// TODO:DEBUG - Data fetching
console.log('[DEBUG] Fetching data:', { url, cacheStrategy });

// TODO:DEBUG - Route changes
'use client';
import { usePathname } from 'next/navigation';

const DebugRoute = () => {
  const pathname = usePathname();
  useEffect(() => {
    console.log('[DEBUG] Route changed:', pathname);
  }, [pathname]);
  return null;
};

export { DebugRoute };
```

## Context7 Integration

Khi debug framework issues, LUÔN dùng Context7 để fetch latest documentation:

```javascript
// Step 1: Resolve library
mcp__context7__resolve-library-id({
  libraryName: "react" // hoặc "next.js"
})

// Step 2: Get specific docs
mcp__context7__query-docs({
  libraryId: "/facebook/react",
  query: "useEffect dependency array missing dependencies"
})
```