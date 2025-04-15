The **React Context API** is a powerful feature that allows you to share state across multiple components in a React application without having to pass props manually through every level of the component tree. It helps in scenarios where you need to pass data through many layers of components, like themes, user authentication, or language preferences, avoiding prop drilling.

Here’s a step-by-step guide on how to use the **React Context API**:

### 1. Create a Context

The first step is to create a context object. This object will hold the data you want to share across the application.

```js
import React from 'react';

const MyContext = React.createContext();
```

### 2. Create a Provider

Next, create a provider component. The `Provider` component is used to pass the context value to its children. This component will wrap your app or any component tree where you want to share state.

```js
const MyProvider = ({ children }) => {
  const [state, setState] = React.useState('Hello World');
  
  return (
    <MyContext.Provider value={{ state, setState }}>
      {children}
    </MyContext.Provider>
  );
};
```

Here, the context's value is an object containing `state` and a `setState` function. This value will be available to all the children components inside `MyProvider`.

### 3. Consume the Context Value

To consume the context value inside any child component, you can use either:

- **`useContext` Hook** (Functional Components)
- **`Context.Consumer`** (Class Components)

#### Using `useContext` (Functional Components)

```js
import React, { useContext } from 'react';

const MyComponent = () => {
  const { state, setState } = useContext(MyContext);

  return (
    <div>
      <p>{state}</p>
      <button onClick={() => setState('Updated State!')}>Update State</button>
    </div>
  );
};
```

The `useContext` hook gives you access to the context value. You can destructure and use it like any other state variable.

#### Using `Context.Consumer` (Class Components)

If you're using a class component, you can consume the context using the `Context.Consumer`:

```js
import React from 'react';

class MyComponent extends React.Component {
  render() {
    return (
      <MyContext.Consumer>
        {({ state, setState }) => (
          <div>
            <p>{state}</p>
            <button onClick={() => setState('Updated State!')}>Update State</button>
          </div>
        )}
      </MyContext.Consumer>
    );
  }
}
```

### 4. Wrapping Your Application with the Provider

To allow all components to access the context, wrap your app with the `Provider` component that holds the state.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { MyProvider } from './MyProvider'; // import provider
import MyComponent from './MyComponent';

const App = () => {
  return (
    <MyProvider>
      <MyComponent />
    </MyProvider>
  );
};

ReactDOM.render(<App />, document.getElementById('root'));
```

### Key Points

- **Provider**: This is where you set the value of the context, making it available to the components in its subtree.
- **Consumer or useContext**: This is how you access the value provided by the `Provider`.

### Benefits of Using Context API

- Avoids prop drilling: No need to pass props through multiple levels of components.
- Centralized state management for specific sections of your app without needing external libraries like Redux.
- Simple to use and lightweight compared to more complex state management solutions.

### Example: Theme Context

Here's a simple example where we create a theme context to manage a dark or light theme:

```js
// ThemeContext.js
import React, { createContext, useState, useContext } from 'react';

const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => setTheme(theme === 'light' ? 'dark' : 'light');
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => useContext(ThemeContext);
```

In your components, you can use the context like this:

```js
// App.js
import React from 'react';
import { ThemeProvider, useTheme } from './ThemeContext';

const ThemedComponent = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff' }}>
      <p>The current theme is {theme}</p>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
};

const App = () => (
  <ThemeProvider>
    <ThemedComponent />
  </ThemeProvider>
);

export default App;
```

### Exam Power-Up:
When using Context API:
- It’s crucial to remember that the value provided by the context is **re-rendered** every time the state changes.
- It's a good practice to **memoize** the value passed to the `Provider` if the value is expensive to compute, to avoid unnecessary renders.
- Use **`useMemo`** and **`useCallback`** for optimizing performance.

Sample MCQ:
1. Which of the following is used to share data across components without prop drilling in React?
    - A) Redux
    - B) React Router
    - C) Context API
    - D) useState

Answer: C) Context API


**React Router** is a standard library for routing in React applications. It enables the navigation between different components or views based on the URL, without requiring a full page reload. This allows for a seamless, single-page application (SPA) experience.

React Router is used to handle the mapping between URL paths and the corresponding React components. It also provides navigation features like back, forward, and dynamic routing, making it an essential tool for building React apps that require routing.

### Core Concepts of React Router

1. **Router**: The top-level component that contains the routing logic and renders different views based on the URL. The most commonly used router is `BrowserRouter`.

2. **Route**: The `Route` component is used to define a mapping between a path (URL) and a component. When the current URL matches the path defined in the `Route`, it renders the specified component.

3. **Switch**: The `Switch` component ensures that only the first matching route is rendered. It’s useful to prevent multiple routes from being displayed at the same time.

4. **Link**: The `Link` component allows navigation between different routes in the app without reloading the page. It’s used in place of traditional anchor tags (`<a>`).

5. **useHistory** / **useNavigate**: These hooks are used for programmatic navigation within your app.

### How to Use React Router

Here’s a simple guide on how to set up React Router in a React app.

#### Step 1: Install React Router

If you're using React Router in your project, you first need to install it:

```bash
npm install react-router-dom
```

#### Step 2: Setup the Router

Wrap your app in a `BrowserRouter` component to enable routing.

```js
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Home from './Home';
import About from './About';
import NotFound from './NotFound';

const App = () => {
  return (
    <Router>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route component={NotFound} /> {/* Fallback route for unmatched paths */}
      </Switch>
    </Router>
  );
};

export default App;
```

- **`<BrowserRouter>`**: This component is responsible for handling the history of navigation.
- **`<Route>`**: This is used to declare what component should be rendered for a specific path.
- **`<Switch>`**: It ensures that only one route is rendered at a time, even if multiple routes match the current URL.

#### Step 3: Add Links for Navigation

Use the `Link` component to navigate between different views.

```js
import React from 'react';
import { Link } from 'react-router-dom';

const Navbar = () => {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
    </nav>
  );
};

export default Navbar;
```

- **`<Link>`**: This component works like a traditional anchor tag but prevents a full page reload.

#### Step 4: Programmatic Navigation

Sometimes, you might need to navigate based on certain conditions or actions (like after a form submission). React Router provides hooks like `useHistory` (v5) or `useNavigate` (v6) for this.

```js
import { useNavigate } from 'react-router-dom';

const SubmitForm = () => {
  const navigate = useNavigate();
  
  const handleSubmit = () => {
    // Logic here
    navigate('/thank-you'); // Redirect after form submission
  };
  
  return (
    <button onClick={handleSubmit}>Submit</button>
  );
};
```

- **`useNavigate()`**: This hook provides a `navigate` function to programmatically change the route.

#### Dynamic Routing

You can pass dynamic values in the URL using route parameters. For example, displaying user details based on their ID.

```js
import React from 'react';
import { useParams } from 'react-router-dom';

const UserProfile = () => {
  const { userId } = useParams(); // Retrieves userId from the URL
  
  return <div>User Profile: {userId}</div>;
};

// In your route
<Route path="/user/:userId" component={UserProfile} />
```

- **`useParams()`**: This hook gives you access to the dynamic parts of the route, such as `:userId`.

#### Example Application

Let's build a small app with React Router.

```js
// Home.js
import React from 'react';

const Home = () => {
  return <h1>Welcome to the Home Page</h1>;
};

export default Home;
```

```js
// About.js
import React from 'react';

const About = () => {
  return <h1>Welcome to the About Page</h1>;
};

export default About;
```

```js
// NotFound.js
import React from 'react';

const NotFound = () => {
  return <h1>404 - Page Not Found</h1>;
};

export default NotFound;
```

```js
// App.js
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Home from './Home';
import About from './About';
import NotFound from './NotFound';
import Navbar from './Navbar';

const App = () => {
  return (
    <Router>
      <Navbar />
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route component={NotFound} />
      </Switch>
    </Router>
  );
};

export default App;
```

### React Router v6 Update

In React Router v6, there are some key changes:
1. **`Switch` is replaced by `Routes`**: The `Switch` component is now `Routes`, which is used to wrap route definitions.
2. **Route `element` prop**: Instead of `component` in `Route`, React Router v6 uses `element` to pass JSX elements directly.

```js
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';

const App = () => {
  return (
    <Router>
      <Navbar />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </Router>
  );
};
```

### Exam Power-Up:
- React Router allows you to build **single-page applications** (SPAs) with navigation without reloading the entire page.
- In v6, the major change is the use of the `Routes` component instead of `Switch`, and the `element` prop replaces `component`.
- **`useNavigate()`** in React Router v6 allows you to navigate programmatically.

Sample MCQ:
1. In React Router v6, what is used to define multiple routes in an application?
   - A) Switch
   - B) Routes
   - C) Link
   - D) Route

Answer: B) Routes

An **Authenticated Route** is a concept in React where certain routes in your application are only accessible to authenticated users. This is useful for protecting pages like user profiles, dashboards, or settings, ensuring that only users who are logged in can access them.

The React Router, along with context or a global state management tool (like Redux or React Context), can be used to create authenticated routes.

### Steps to Create Authenticated Routes

1. **Set up authentication logic**: You'll typically store user authentication state (e.g., whether the user is logged in or not) in a context or a global state.

2. **Create a PrivateRoute component**: This component will check if the user is authenticated before rendering the requested route. If the user is not authenticated, it can redirect to a login page.

3. **Protect routes**: Wrap routes that should only be accessible to authenticated users in the `PrivateRoute` component.

### Example: Authenticated Route Using React Context

1. **Create an Authentication Context**

First, set up a context that will manage the authentication state.

```js
// AuthContext.js
import React, { createContext, useState, useContext } from 'react';

const AuthContext = createContext();

export const useAuth = () => useContext(AuthContext);

export const AuthProvider = ({ children }) => {
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const login = () => setIsAuthenticated(true);
  const logout = () => setIsAuthenticated(false);

  return (
    <AuthContext.Provider value={{ isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
```

Here, we are using `useState` to track the `isAuthenticated` state, and we provide `login` and `logout` functions that will update this state.

2. **Create the PrivateRoute Component**

This component will check if the user is authenticated. If the user is not authenticated, they are redirected to a login page.

```js
// PrivateRoute.js
import React from 'react';
import { Route, Redirect } from 'react-router-dom';
import { useAuth } from './AuthContext';

const PrivateRoute = ({ component: Component, ...rest }) => {
  const { isAuthenticated } = useAuth();

  return (
    <Route
      {...rest}
      render={(props) =>
        isAuthenticated ? (
          <Component {...props} />
        ) : (
          <Redirect to="/login" />
        )
      }
    />
  );
};

export default PrivateRoute;
```

- `PrivateRoute` is a wrapper around the `Route` component. It checks if `isAuthenticated` is true.
- If the user is authenticated, it renders the desired component.
- If not, it redirects them to the `/login` page.

3. **Set Up Routing with PrivateRoute**

Now, let's set up routing where we use `PrivateRoute` to protect authenticated routes.

```js
// App.js
import React from 'react';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import { AuthProvider, useAuth } from './AuthContext';
import PrivateRoute from './PrivateRoute';

const Home = () => <h1>Home Page</h1>;
const Dashboard = () => <h1>Dashboard (Protected)</h1>;
const Login = () => {
  const { login } = useAuth();
  return (
    <div>
      <h1>Login Page</h1>
      <button onClick={login}>Login</button>
    </div>
  );
};

const App = () => {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          <Route path="/dashboard" element={<PrivateRoute component={Dashboard} />} />
        </Routes>
      </Router>
    </AuthProvider>
  );
};

export default App;
```

In this example:
- `/dashboard` is a protected route. The user will be redirected to the login page if they are not authenticated.
- `/login` is the page where the user can log in.
- The `login` function in `Login` simulates a login and sets `isAuthenticated` to `true`.

### Explanation of Components

- **`AuthContext`**: Manages the authentication state of the application and provides the necessary functions (`login`, `logout`) to update the state.
- **`PrivateRoute`**: A custom route component that checks if the user is authenticated before rendering the protected component. If not authenticated, it redirects to the login page.
- **`Routes` in `App.js`**: We use `Route` for both public and protected routes. The `PrivateRoute` ensures only authenticated users can access the `/dashboard`.

### React Router v6 Update (for Private Routes)

In React Router v6, `PrivateRoute` is simplified because `Route` now uses the `element` prop directly, and `Redirect` is replaced by `Navigate`. Here’s how the `PrivateRoute` component can be implemented:

```js
// PrivateRoute.js (for React Router v6)
import React from 'react';
import { Route, Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

const PrivateRoute = ({ element, ...rest }) => {
  const { isAuthenticated } = useAuth();

  return isAuthenticated ? element : <Navigate to="/login" />;
};

export default PrivateRoute;
```

And in your `App.js`:

```js
// App.js (for React Router v6)
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider, useAuth } from './AuthContext';
import PrivateRoute from './PrivateRoute';

const Home = () => <h1>Home Page</h1>;
const Dashboard = () => <h1>Dashboard (Protected)</h1>;
const Login = () => {
  const { login } = useAuth();
  return (
    <div>
      <h1>Login Page</h1>
      <button onClick={login}>Login</button>
    </div>
  );
};

const App = () => {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          <Route path="/dashboard" element={<PrivateRoute element={<Dashboard />} />} />
        </Routes>
      </Router>
    </AuthProvider>
  );
};

export default App;
```

### Benefits of Using Authenticated Routes

- **Security**: Ensures only authorized users can access sensitive or protected resources.
- **Seamless User Experience**: Users are automatically redirected to the login page if they try to access protected routes without authentication.
- **Centralized Auth Logic**: By using context, you centralize your authentication logic, making it easier to manage and modify.

### Exam Power-Up:
- React Router allows you to create **authenticated routes** by combining route guarding with context or state management.
- Use **`PrivateRoute`** to protect specific routes, redirecting unauthorized users to the login page.
- React Router v6 has replaced **`Redirect`** with **`Navigate`** and **`component`** with **`element`** for defining components.

Sample MCQ:
1. In React Router v6, what component is used to redirect the user to another route?
   - A) Redirect
   - B) Navigate
   - C) Route
   - D) RedirectTo

Answer: B) Navigate

Would you like to explore more about handling different types of authentication (e.g., JWT tokens) in your app?


**React Redux** is a library that helps you manage the state of your application globally, making it easier to share state between components that don't have a direct parent-child relationship. Redux follows a strict unidirectional data flow and provides a predictable state container for JavaScript applications.

### Core Concepts of Redux

1. **Store**: The store is the central repository that holds the entire state of your application. It is created using the `createStore()` function and contains all of your app’s state.

2. **Actions**: Actions are plain JavaScript objects that describe an event that has occurred in your application. Actions are sent to the store using **dispatch()**.

3. **Reducers**: Reducers are pure functions that specify how the state of the application changes in response to an action. Reducers take the current state and an action, and return a new state.

4. **Dispatch**: The **dispatch()** function sends an action to the store, which then updates the state based on the action type.

5. **Selectors**: Selectors are functions that read values from the store. They can be used to get specific slices of the state or perform calculations.

6. **Provider**: The `Provider` component from `react-redux` makes the Redux store available to all components in the app, allowing components to access the store’s state.

### How Redux Works with React

1. **Setting Up Redux**:
   To use Redux with React, you need to install `react-redux` and `redux`:

   ```bash
   npm install redux react-redux
   ```

2. **Create a Redux Store**:
   First, you need to define your actions, reducers, and store.

   **Define Action Types and Actions**:

   ```js
   // actions.js
   export const INCREMENT = 'INCREMENT';
   export const DECREMENT = 'DECREMENT';

   export const increment = () => ({
     type: INCREMENT,
   });

   export const decrement = () => ({
     type: DECREMENT,
   });
   ```

   **Create Reducers**:

   Reducers define how the state should change in response to actions. Here’s a simple counter reducer:

   ```js
   // counterReducer.js
   import { INCREMENT, DECREMENT } from './actions';

   const initialState = {
     count: 0,
   };

   const counterReducer = (state = initialState, action) => {
     switch (action.type) {
       case INCREMENT:
         return {
           ...state,
           count: state.count + 1,
         };
       case DECREMENT:
         return {
           ...state,
           count: state.count - 1,
         };
       default:
         return state;
     }
   };

   export default counterReducer;
   ```

   **Create the Redux Store**:

   Now, combine all your reducers (if you have multiple) and create the store:

   ```js
   // store.js
   import { createStore } from 'redux';
   import counterReducer from './counterReducer';

   const store = createStore(counterReducer);

   export default store;
   ```

3. **Wrap the App with `Provider`**:
   The `Provider` component from `react-redux` will make the Redux store available to all components in your app:

   ```js
   // index.js
   import React from 'react';
   import ReactDOM from 'react-dom';
   import { Provider } from 'react-redux';
   import App from './App';
   import store from './store';

   ReactDOM.render(
     <Provider store={store}>
       <App />
     </Provider>,
     document.getElementById('root')
   );
   ```

4. **Connect Components to Redux Store**:
   Use the `useSelector` hook to access state and `useDispatch` hook to dispatch actions.

   **Accessing the State**:

   ```js
   // Counter.js
   import React from 'react';
   import { useSelector, useDispatch } from 'react-redux';
   import { increment, decrement } from './actions';

   const Counter = () => {
     const count = useSelector((state) => state.count); // Get the count from the Redux store
     const dispatch = useDispatch(); // Access the dispatch function

     return (
       <div>
         <h1>Count: {count}</h1>
         <button onClick={() => dispatch(increment())}>Increment</button>
         <button onClick={() => dispatch(decrement())}>Decrement</button>
       </div>
     );
   };

   export default Counter;
   ```

5. **Dispatching Actions**:

   - **`useDispatch`**: This hook provides access to the `dispatch` function, allowing you to send actions to the store.
   - **`useSelector`**: This hook allows you to select pieces of state from the Redux store and make your component re-render when that state changes.

6. **Creating More Complex State**:

   For larger applications, you might want to split your state into multiple slices (reducers). You can use **combineReducers** to combine different reducers:

   ```js
   // rootReducer.js
   import { combineReducers } from 'redux';
   import counterReducer from './counterReducer';

   const rootReducer = combineReducers({
     counter: counterReducer,
   });

   export default rootReducer;
   ```

   Then, update the store to use `rootReducer`:

   ```js
   // store.js
   import { createStore } from 'redux';
   import rootReducer from './rootReducer';

   const store = createStore(rootReducer);

   export default store;
   ```

### Example Application with Redux

Let’s put everything together in a small app that uses Redux to manage the counter state.

```js
// actions.js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';

export const increment = () => ({ type: INCREMENT });
export const decrement = () => ({ type: DECREMENT });
```

```js
// counterReducer.js
import { INCREMENT, DECREMENT } from './actions';

const initialState = { count: 0 };

const counterReducer = (state = initialState, action) => {
  switch (action.type) {
    case INCREMENT:
      return { ...state, count: state.count + 1 };
    case DECREMENT:
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
};

export default counterReducer;
```

```js
// store.js
import { createStore } from 'redux';
import counterReducer from './counterReducer';

const store = createStore(counterReducer);

export default store;
```

```js
// App.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from './actions';

const App = () => {
  const count = useSelector((state) => state.count);
  const dispatch = useDispatch();

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
    </div>
  );
};

export default App;
```

```js
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import App from './App';
import store from './store';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

### Benefits of Using Redux with React

- **Predictable State**: Redux provides a centralized state container, which ensures that the state is predictable and follows a strict flow.
- **Debugging**: You can use Redux DevTools to inspect every action that is dispatched and see how the state changes over time.
- **Middleware**: Redux supports middleware, allowing you to intercept dispatched actions for logging, async actions, or custom logic (e.g., Redux Thunk or Redux Saga).

### Exam Power-Up:
- Redux manages state globally, ensuring consistency across the app.
- You should always use **pure functions** for reducers and avoid side effects.
- Use **`useDispatch`** to dispatch actions and **`useSelector`** to access state from the store.

Sample MCQ:
1. Which of the following is the function used to update the Redux state based on dispatched actions?
   - A) useSelector
   - B) dispatch
   - C) reducer
   - D) setState

Answer: C) reducer

Would you like to explore middleware like **Redux Thunk** for handling async actions or dive into more complex state management in Redux?

### Middleware in Redux: Redux Thunk

**Redux Thunk** is a middleware that allows you to write action creators that return **functions** instead of plain objects. This is particularly useful for handling asynchronous logic (like fetching data from an API) within Redux actions.

By default, Redux actions are plain objects. However, sometimes we need to perform asynchronous tasks (e.g., API calls, delays) before dispatching actions to the store. **Redux Thunk** allows you to write action creators that return a function, which can dispatch actions after completing an async operation.

### Why Use Redux Thunk?

1. **Async Actions**: It allows you to handle async operations like API calls, timeouts, or conditional actions in your Redux flow.
2. **Dispatch Delayed Actions**: You can dispatch multiple actions based on the result of the async operation (e.g., dispatch a loading action, then dispatch a success/failure action based on the result).
3. **Access Redux store within actions**: The function passed to `dispatch()` can have access to the Redux store, allowing you to interact with state if needed before dispatching another action.

### How to Use Redux Thunk

#### 1. Install Redux Thunk

First, you need to install the `redux-thunk` middleware.

```bash
npm install redux-thunk
```

#### 2. Apply Middleware to the Redux Store

Next, apply Redux Thunk middleware when creating the store using `applyMiddleware`.

```js
// store.js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk'; // Import thunk middleware
import rootReducer from './rootReducer';

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;
```

#### 3. Create Async Actions Using Redux Thunk

Redux Thunk allows you to write action creators that return a function. This function will receive the `dispatch` and `getState` as arguments, which allows you to dispatch actions asynchronously.

Here’s an example of fetching data asynchronously:

```js
// actions.js
import { FETCH_DATA_REQUEST, FETCH_DATA_SUCCESS, FETCH_DATA_FAILURE } from './actionTypes';

// Action creator for making an async API call
export const fetchData = () => {
  return (dispatch) => {
    dispatch({ type: FETCH_DATA_REQUEST }); // Dispatch request action to indicate loading state

    fetch('https://api.example.com/data')
      .then((response) => response.json())
      .then((data) => {
        dispatch({ type: FETCH_DATA_SUCCESS, payload: data }); // Dispatch success action with data
      })
      .catch((error) => {
        dispatch({ type: FETCH_DATA_FAILURE, payload: error }); // Dispatch failure action with error
      });
  };
};
```

Here, the action `fetchData` is a **thunk**, which returns a function instead of a plain action. The function performs the async operation (fetching data from an API) and then dispatches different actions based on the result.

#### 4. Reducers to Handle Async Actions

Your reducers will handle different action types for loading, success, and failure states.

```js
// dataReducer.js
import {
  FETCH_DATA_REQUEST,
  FETCH_DATA_SUCCESS,
  FETCH_DATA_FAILURE,
} from './actionTypes';

const initialState = {
  loading: false,
  data: null,
  error: null,
};

const dataReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_DATA_REQUEST:
      return { ...state, loading: true };
    case FETCH_DATA_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_DATA_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};

export default dataReducer;
```

In this example, the reducer listens for three action types:
- **FETCH_DATA_REQUEST**: This action is dispatched when the request starts (for loading state).
- **FETCH_DATA_SUCCESS**: This action is dispatched when the request succeeds, with the fetched data.
- **FETCH_DATA_FAILURE**: This action is dispatched if the request fails, with the error.

#### 5. Using the Action in a Component

Now, you can dispatch the `fetchData` action in your component and use the state to show loading, error, or success states.

```js
// DataComponent.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchData } from './actions';

const DataComponent = () => {
  const dispatch = useDispatch();
  const { loading, data, error } = useSelector((state) => state.data);

  useEffect(() => {
    dispatch(fetchData());
  }, [dispatch]);

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <div>
      <h1>Data:</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};

export default DataComponent;
```

### Full Example of Redux Thunk in Action

Let’s combine everything into a complete example.

#### 1. **Action Types**

```js
// actionTypes.js
export const FETCH_DATA_REQUEST = 'FETCH_DATA_REQUEST';
export const FETCH_DATA_SUCCESS = 'FETCH_DATA_SUCCESS';
export const FETCH_DATA_FAILURE = 'FETCH_DATA_FAILURE';
```

#### 2. **Actions**

```js
// actions.js
import { FETCH_DATA_REQUEST, FETCH_DATA_SUCCESS, FETCH_DATA_FAILURE } from './actionTypes';

export const fetchData = () => {
  return (dispatch) => {
    dispatch({ type: FETCH_DATA_REQUEST });

    fetch('https://api.example.com/data')
      .then((response) => response.json())
      .then((data) => {
        dispatch({ type: FETCH_DATA_SUCCESS, payload: data });
      })
      .catch((error) => {
        dispatch({ type: FETCH_DATA_FAILURE, payload: error });
      });
  };
};
```

#### 3. **Reducer**

```js
// dataReducer.js
import { FETCH_DATA_REQUEST, FETCH_DATA_SUCCESS, FETCH_DATA_FAILURE } from './actionTypes';

const initialState = {
  loading: false,
  data: null,
  error: null,
};

const dataReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_DATA_REQUEST:
      return { ...state, loading: true };
    case FETCH_DATA_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_DATA_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};

export default dataReducer;
```

#### 4. **Store**

```js
// store.js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import dataReducer from './dataReducer';

const store = createStore(dataReducer, applyMiddleware(thunk));

export default store;
```

#### 5. **App Component**

```js
// App.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchData } from './actions';

const App = () => {
  const dispatch = useDispatch();
  const { loading, data, error } = useSelector((state) => state);

  useEffect(() => {
    dispatch(fetchData());
  }, [dispatch]);

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <div>
      <h1>Fetched Data:</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};

export default App;
```

### Benefits of Using Redux Thunk

- **Handling Async Logic**: It helps manage async actions like API calls, handling the state transitions for loading, success, and failure.
- **Predictable State**: Even with async actions, Redux keeps your state predictable by dispatching multiple actions (e.g., loading, success, error).
- **Composable**: You can chain multiple async actions or even dispatch additional actions based on certain conditions.

### Exam Power-Up:
- **Redux Thunk** allows you to handle **async logic** inside your actions.
- It's a middleware that **enhances Redux** by enabling action creators to return **functions** instead of objects.
- **Dispatch async actions** and track loading states using Redux Thunk.

Sample MCQ:
1. Which middleware allows you to handle async actions in Redux?
   - A) Redux Logger
   - B) Redux DevTools
   - C) Redux Thunk
   - D) Redux Saga

Answer: C) Redux Thunk



### Middleware in Redux: Redux Thunk

**Redux Thunk** is a middleware that allows you to write action creators that return **functions** instead of plain objects. This is particularly useful for handling asynchronous logic (like fetching data from an API) within Redux actions.

By default, Redux actions are plain objects. However, sometimes we need to perform asynchronous tasks (e.g., API calls, delays) before dispatching actions to the store. **Redux Thunk** allows you to write action creators that return a function, which can dispatch actions after completing an async operation.

### Why Use Redux Thunk?

1. **Async Actions**: It allows you to handle async operations like API calls, timeouts, or conditional actions in your Redux flow.
2. **Dispatch Delayed Actions**: You can dispatch multiple actions based on the result of the async operation (e.g., dispatch a loading action, then dispatch a success/failure action based on the result).
3. **Access Redux store within actions**: The function passed to `dispatch()` can have access to the Redux store, allowing you to interact with state if needed before dispatching another action.

### How to Use Redux Thunk

#### 1. Install Redux Thunk

First, you need to install the `redux-thunk` middleware.

```bash
npm install redux-thunk
```

#### 2. Apply Middleware to the Redux Store

Next, apply Redux Thunk middleware when creating the store using `applyMiddleware`.

```js
// store.js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk'; // Import thunk middleware
import rootReducer from './rootReducer';

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;
```

#### 3. Create Async Actions Using Redux Thunk

Redux Thunk allows you to write action creators that return a function. This function will receive the `dispatch` and `getState` as arguments, which allows you to dispatch actions asynchronously.

Here’s an example of fetching data asynchronously:

```js
// actions.js
import { FETCH_DATA_REQUEST, FETCH_DATA_SUCCESS, FETCH_DATA_FAILURE } from './actionTypes';

// Action creator for making an async API call
export const fetchData = () => {
  return (dispatch) => {
    dispatch({ type: FETCH_DATA_REQUEST }); // Dispatch request action to indicate loading state

    fetch('https://api.example.com/data')
      .then((response) => response.json())
      .then((data) => {
        dispatch({ type: FETCH_DATA_SUCCESS, payload: data }); // Dispatch success action with data
      })
      .catch((error) => {
        dispatch({ type: FETCH_DATA_FAILURE, payload: error }); // Dispatch failure action with error
      });
  };
};
```

Here, the action `fetchData` is a **thunk**, which returns a function instead of a plain action. The function performs the async operation (fetching data from an API) and then dispatches different actions based on the result.

#### 4. Reducers to Handle Async Actions

Your reducers will handle different action types for loading, success, and failure states.

```js
// dataReducer.js
import {
  FETCH_DATA_REQUEST,
  FETCH_DATA_SUCCESS,
  FETCH_DATA_FAILURE,
} from './actionTypes';

const initialState = {
  loading: false,
  data: null,
  error: null,
};

const dataReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_DATA_REQUEST:
      return { ...state, loading: true };
    case FETCH_DATA_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_DATA_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};

export default dataReducer;
```

In this example, the reducer listens for three action types:
- **FETCH_DATA_REQUEST**: This action is dispatched when the request starts (for loading state).
- **FETCH_DATA_SUCCESS**: This action is dispatched when the request succeeds, with the fetched data.
- **FETCH_DATA_FAILURE**: This action is dispatched if the request fails, with the error.

#### 5. Using the Action in a Component

Now, you can dispatch the `fetchData` action in your component and use the state to show loading, error, or success states.

```js
// DataComponent.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchData } from './actions';

const DataComponent = () => {
  const dispatch = useDispatch();
  const { loading, data, error } = useSelector((state) => state.data);

  useEffect(() => {
    dispatch(fetchData());
  }, [dispatch]);

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <div>
      <h1>Data:</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};

export default DataComponent;
```

### Full Example of Redux Thunk in Action

Let’s combine everything into a complete example.

#### 1. **Action Types**

```js
// actionTypes.js
export const FETCH_DATA_REQUEST = 'FETCH_DATA_REQUEST';
export const FETCH_DATA_SUCCESS = 'FETCH_DATA_SUCCESS';
export const FETCH_DATA_FAILURE = 'FETCH_DATA_FAILURE';
```

#### 2. **Actions**

```js
// actions.js
import { FETCH_DATA_REQUEST, FETCH_DATA_SUCCESS, FETCH_DATA_FAILURE } from './actionTypes';

export const fetchData = () => {
  return (dispatch) => {
    dispatch({ type: FETCH_DATA_REQUEST });

    fetch('https://api.example.com/data')
      .then((response) => response.json())
      .then((data) => {
        dispatch({ type: FETCH_DATA_SUCCESS, payload: data });
      })
      .catch((error) => {
        dispatch({ type: FETCH_DATA_FAILURE, payload: error });
      });
  };
};
```

#### 3. **Reducer**

```js
// dataReducer.js
import { FETCH_DATA_REQUEST, FETCH_DATA_SUCCESS, FETCH_DATA_FAILURE } from './actionTypes';

const initialState = {
  loading: false,
  data: null,
  error: null,
};

const dataReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_DATA_REQUEST:
      return { ...state, loading: true };
    case FETCH_DATA_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_DATA_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};

export default dataReducer;
```

#### 4. **Store**

```js
// store.js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import dataReducer from './dataReducer';

const store = createStore(dataReducer, applyMiddleware(thunk));

export default store;
```

#### 5. **App Component**

```js
// App.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchData } from './actions';

const App = () => {
  const dispatch = useDispatch();
  const { loading, data, error } = useSelector((state) => state);

  useEffect(() => {
    dispatch(fetchData());
  }, [dispatch]);

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <div>
      <h1>Fetched Data:</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};

export default App;
```

### Benefits of Using Redux Thunk

- **Handling Async Logic**: It helps manage async actions like API calls, handling the state transitions for loading, success, and failure.
- **Predictable State**: Even with async actions, Redux keeps your state predictable by dispatching multiple actions (e.g., loading, success, error).
- **Composable**: You can chain multiple async actions or even dispatch additional actions based on certain conditions.

### Exam Power-Up:
- **Redux Thunk** allows you to handle **async logic** inside your actions.
- It's a middleware that **enhances Redux** by enabling action creators to return **functions** instead of objects.
- **Dispatch async actions** and track loading states using Redux Thunk.

Sample MCQ:
1. Which middleware allows you to handle async actions in Redux?
   - A) Redux Logger
   - B) Redux DevTools
   - C) Redux Thunk
   - D) Redux Saga

Answer: C) Redux Thunk

React provides several built-in hooks that allow you to manage state, side effects, and lifecycle methods within functional components. Here's a breakdown of the most commonly used hooks in React:

### 1. **useState**
The `useState` hook allows you to add state to your functional components.

#### Syntax:
```js
const [state, setState] = useState(initialValue);
```

- `state`: The current state value.
- `setState`: A function that updates the state.

#### Example:
```js
import React, { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0); // Initializes state with 0

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

export default Counter;
```

### 2. **useEffect**
The `useEffect` hook allows you to perform side effects in your components, such as fetching data, subscribing to a data stream, or manually changing the DOM.

#### Syntax:
```js
useEffect(() => {
  // Your side effect logic here
}, [dependencies]);
```

- The **effect function** is executed after every render, and the **dependency array** controls when the effect should run (if empty, it runs once when the component mounts).

#### Example:
```js
import React, { useState, useEffect } from 'react';

const Timer = () => {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);

    // Cleanup on component unmount
    return () => clearInterval(interval);
  }, []); // Empty dependency array means it only runs on mount

  return <p>Seconds: {seconds}</p>;
};

export default Timer;
```

### 3. **useContext**
The `useContext` hook allows you to access the value of a React Context directly without having to use the `Context.Consumer` component.

#### Syntax:
```js
const value = useContext(MyContext);
```

- `MyContext`: The context you want to access.
- `value`: The current context value.

#### Example:
```js
import React, { useContext } from 'react';

const ThemeContext = React.createContext('light');

const ThemedComponent = () => {
  const theme = useContext(ThemeContext);
  return <div>Current theme: {theme}</div>;
};

const App = () => {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedComponent />
    </ThemeContext.Provider>
  );
};

export default App;
```

### 4. **useReducer**
The `useReducer` hook is an alternative to `useState` for managing more complex state logic, such as when state depends on previous state or when there are multiple sub-values in the state.

#### Syntax:
```js
const [state, dispatch] = useReducer(reducer, initialState);
```

- `reducer`: A function that handles state updates based on actions.
- `dispatch`: A function to dispatch actions.

#### Example:
```js
import React, { useReducer } from 'react';

const initialState = { count: 0 };

const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
    </div>
  );
};

export default Counter;
```

### 5. **useCallback**
The `useCallback` hook returns a memoized version of a function that only changes if one of the dependencies has changed. It's useful for optimizing performance by preventing unnecessary re-renders.

#### Syntax:
```js
const memoizedCallback = useCallback(() => {
  // function body
}, [dependencies]);
```

- `dependencies`: The values that, if changed, will trigger the recreation of the callback function.

#### Example:
```js
import React, { useState, useCallback } from 'react';

const Button = ({ onClick }) => {
  console.log('Button rendered');
  return <button onClick={onClick}>Click me</button>;
};

const App = () => {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => {
    setCount((prev) => prev + 1);
  }, [setCount]);

  return (
    <div>
      <Button onClick={increment} />
      <p>Count: {count}</p>
    </div>
  );
};

export default App;
```

### 6. **useMemo**
The `useMemo` hook returns a memoized value and only recomputes it when one of its dependencies changes. It’s useful for optimizing performance, especially for expensive calculations.

#### Syntax:
```js
const memoizedValue = useMemo(() => {
  // computation
}, [dependencies]);
```

- `dependencies`: The values that, if changed, will trigger the recalculation of the value.

#### Example:
```js
import React, { useState, useMemo } from 'react';

const ExpensiveComputation = ({ num }) => {
  console.log('Computing...');
  return <p>Result: {num * 2}</p>;
};

const App = () => {
  const [count, setCount] = useState(0);
  const [num, setNum] = useState(10);

  const computedValue = useMemo(() => num * 2, [num]);

  return (
    <div>
      <ExpensiveComputation num={computedValue} />
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setNum(num + 1)}>Change Num</button>
    </div>
  );
};

export default App;
```

### 7. **useRef**
The `useRef` hook is used to persist values between renders. It can be used to access a DOM element or store a mutable value that doesn't cause re-renders when it changes.

#### Syntax:
```js
const myRef = useRef(initialValue);
```

- `myRef`: A mutable object where `.current` holds the value.
- `initialValue`: The initial value of the ref.

#### Example:
```js
import React, { useRef } from 'react';

const FocusInput = () => {
  const inputRef = useRef(null);

  const handleClick = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={handleClick}>Focus Input</button>
    </div>
  );
};

export default FocusInput;
```

### 8. **useLayoutEffect**
The `useLayoutEffect` hook is similar to `useEffect`, but it runs synchronously after all DOM mutations. This makes it useful for reading layout from the DOM and synchronously re-rendering.

#### Syntax:
```js
useLayoutEffect(() => {
  // DOM mutations and layout code
}, [dependencies]);
```

- It runs after the DOM is painted, but before the browser has a chance to paint.

#### Example:
```js
import React, { useLayoutEffect, useState } from 'react';

const LayoutEffectExample = () => {
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    setWidth(window.innerWidth);
  }, []);

  return <p>Window width: {width}</p>;
};

export default LayoutEffectExample;
```

### 9. **useImperativeHandle**
The `useImperativeHandle` hook customizes the instance value that is exposed when using `ref`. This allows parent components to access specific methods or properties on the child component.

#### Syntax:
```js
useImperativeHandle(ref, () => {
  return {
    // Methods or properties
  };
}, [dependencies]);
```

- `ref`: A `ref` that the parent component will use to access the child's methods.

#### Example:
```js
import React, { useImperativeHandle, useRef } from 'react';

const CustomInput = React.forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
  }));

  return <input ref={inputRef} />;
});

const Parent = () => {
  const inputRef = useRef();

  const handleFocus = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <CustomInput ref={inputRef} />
      <button onClick={handleFocus}>Focus Input</button>
    </div>
  );
};

export default Parent;
```

### Summary of Hooks:
1. **`useState`**: Manages state in functional components.
2. **`useEffect`**: Performs side effects like fetching data.
3. **`useContext`**: Accesses values from context.
4. **`useReducer`**: Manages complex state logic.
5. **`useCallback`**: Memoizes functions to prevent unnecessary re-renders.
6. **`useMemo`**: Memoizes values to optimize expensive calculations.
7. **`useRef`**: Accesses DOM elements and persists values between renders.
8. **`useLayoutEffect`**: Performs DOM mutations synchronously after layout.
9. **`useImperativeHandle`**: Customizes ref behavior for child components.

### Exam Power-Up:
- **useState** manages local component state, while **useReducer** is better for complex state logic.
- **useEffect** and **useLayoutEffect** are both used for side effects, but **useLayoutEffect** is synchronous and can block the render process.
- **useRef** persists values without causing re-renders, and **useImperativeHandle** can expose methods to parent components.

Sample MCQ:
1. Which of the following hooks allows you to perform side effects in React?
   - A) useState
   - B) useEffect
   - C) useReducer
   - D) useMemo

Answer: B) useEffect

Would you like to dive deeper into any specific hook or an advanced usage example?

Both **`useCallback`** and **`useMemo`** are hooks in React used to optimize performance by memoizing values or functions, but they serve different purposes. Let's break down the differences between these two hooks.

### 1. **`useCallback`**: Memoizes Functions

- **Purpose**: The `useCallback` hook is used to memoize functions, ensuring that the function reference does not change unless one of its dependencies changes.
  
- **Use Case**: It is useful when passing functions to child components to prevent unnecessary re-renders of the child component that relies on function identity (for example, if the function is passed as a prop).

#### Syntax:
```js
const memoizedCallback = useCallback(() => {
  // function logic
}, [dependencies]);
```

- The function is only recreated if the dependencies change.
- It prevents a new instance of the function from being created on every render.

#### Example:

```js
import React, { useState, useCallback } from 'react';

const Button = ({ onClick }) => {
  console.log('Button rendered');
  return <button onClick={onClick}>Click me</button>;
};

const App = () => {
  const [count, setCount] = useState(0);

  // useCallback ensures that the function doesn't get recreated unless 'setCount' changes
  const increment = useCallback(() => {
    setCount((prev) => prev + 1);
  }, [setCount]);

  return (
    <div>
      <Button onClick={increment} />
      <p>Count: {count}</p>
    </div>
  );
};

export default App;
```

- In the above example, `increment` is memoized with `useCallback`. If the `increment` function is passed to child components, it will not be recreated unless the `setCount` dependency changes. This is beneficial if the child component does not need to re-render unless `increment` changes.

### 2. **`useMemo`**: Memoizes Values

- **Purpose**: The `useMemo` hook is used to memoize the result of a computation or a value. It returns a memoized version of the value that only recomputes when one of its dependencies has changed.

- **Use Case**: It is useful for expensive computations that you don't want to re-run unless necessary. For example, you might use `useMemo` to memoize a calculated value that involves heavy computation (e.g., filtering or sorting data).

#### Syntax:
```js
const memoizedValue = useMemo(() => {
  // expensive calculation or value
}, [dependencies]);
```

- The value is only recomputed if the dependencies change.

#### Example:

```js
import React, { useState, useMemo } from 'react';

const ExpensiveComputation = ({ num }) => {
  console.log('Computing...');
  return <p>Result: {num * 2}</p>;
};

const App = () => {
  const [count, setCount] = useState(0);
  const [num, setNum] = useState(10);

  // useMemo ensures that the computation only happens when 'num' changes
  const computedValue = useMemo(() => num * 2, [num]);

  return (
    <div>
      <ExpensiveComputation num={computedValue} />
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setNum(num + 1)}>Change Num</button>
    </div>
  );
};

export default App;
```

- In this example, the multiplication (`num * 2`) is an expensive calculation that only recomputes when the `num` value changes. By using `useMemo`, we prevent unnecessary re-calculations on every render.

### Key Differences

| Feature             | `useCallback`                                       | `useMemo`                                           |
|---------------------|-----------------------------------------------------|-----------------------------------------------------|
| **Purpose**          | Memoizes a function to prevent unnecessary re-creation. | Memoizes a computed value or result of a function to prevent unnecessary recalculations. |
| **Return Type**      | Returns a memoized function.                       | Returns a memoized value (result of a function).    |
| **Use Case**         | When passing functions as props to child components to avoid re-renders. | When computing expensive values or results that don't need to be recomputed unless certain dependencies change. |
| **Typical Usage**    | Useful with event handlers or callback functions.   | Useful with expensive computations like sorting or filtering large arrays. |
| **Example**          | Memoizing event handlers or callbacks.             | Memoizing computed values or heavy calculations.    |

### When to Use `useCallback` vs `useMemo`

- **Use `useCallback`** when you need to **memoize a function** (usually when passing functions as props to child components). This is important to prevent child components from re-rendering when the function reference doesn't change.
- **Use `useMemo`** when you need to **memoize a value** (especially when the value is the result of an expensive calculation). This helps avoid re-running the calculation on every render unless necessary.

### Example with Both `useMemo` and `useCallback`

```js
import React, { useState, useMemo, useCallback } from 'react';

const ExpensiveComputation = ({ value }) => {
  console.log('Computing...');
  return <p>Computed Value: {value}</p>;
};

const App = () => {
  const [count, setCount] = useState(0);
  const [num, setNum] = useState(10);

  // Memoize the computed value to avoid recomputing unless 'num' changes
  const computedValue = useMemo(() => num * 2, [num]);

  // Memoize the callback to avoid re-creating it on every render
  const increment = useCallback(() => {
    setCount(count + 1);
  }, [count]);

  return (
    <div>
      <button onClick={increment}>Increment Count</button>
      <p>Count: {count}</p>
      <ExpensiveComputation value={computedValue} />
      <button onClick={() => setNum(num + 1)}>Change Num</button>
    </div>
  );
};

export default App;
```

### Exam Power-Up:
- **`useCallback`**: Memoizes functions to prevent re-creation unless dependencies change.
- **`useMemo`**: Memoizes expensive calculations or values to avoid unnecessary re-computations.

Sample MCQ:
1. Which hook is used to optimize the performance of a function in React by memoizing it?
   - A) `useState`
   - B) `useEffect`
   - C) `useCallback`
   - D) `useMemo`

Answer: C) `useCallback`

Would you like more examples or further details on performance optimization with hooks?

**Internationalization (i18n)** is the process of designing an application so that it can be adapted to different languages, regions, and cultures. It prepares your application for **localization (l10n)**, which is the actual process of adapting the application to a specific language or region by translating the content and formatting it according to local conventions.

In the context of React, internationalization involves:
1. **Displaying content in multiple languages**.
2. **Formatting dates, numbers, and currencies according to local conventions**.
3. **Handling pluralization** and other language-specific grammar rules.

There are libraries available to help with internationalization in React, with the most popular one being **react-i18next**.

### Steps to Implement Internationalization (i18n) in a React App

#### 1. **Install Dependencies**

To get started with internationalization in React, you can use `react-i18next`, a popular library for internationalizing React apps. It supports features like language switching, translation, and formatting.

Install the necessary packages:

```bash
npm install react-i18next i18next
```

You may also need `i18next-browser-languagedetector` for automatic language detection.

```bash
npm install i18next-browser-languagedetector
```

#### 2. **Set Up i18next**

Create a file to configure i18next. For example, create an `i18n.js` file.

```js
// src/i18n.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

// Import translations
import enTranslations from './locales/en/translation.json';
import frTranslations from './locales/fr/translation.json';

i18n
  .use(LanguageDetector) // Detect language automatically
  .use(initReactI18next) // Pass i18n instance to react-i18next
  .init({
    resources: {
      en: {
        translation: enTranslations,
      },
      fr: {
        translation: frTranslations,
      },
    },
    fallbackLng: 'en', // Default language if detection fails
    interpolation: {
      escapeValue: false, // React already escapes variables
    },
  });

export default i18n;
```

Here, `resources` holds the translation data for each language (e.g., English and French in this case).

#### 3. **Organize Translation Files**

You need to create separate files for each language that hold the translations. For example:

```json
// src/locales/en/translation.json
{
  "welcome": "Welcome to our application",
  "login": "Login",
  "logout": "Logout"
}
```

```json
// src/locales/fr/translation.json
{
  "welcome": "Bienvenue dans notre application",
  "login": "Se connecter",
  "logout": "Se déconnecter"
}
```

#### 4. **Wrap Your Application with `I18nextProvider`**

In your main `index.js` or `App.js` file, import `i18n.js` and ensure your app is wrapped with `I18nextProvider` to make the translation data available throughout your app.

```js
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './i18n'; // Import the i18n configuration

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

#### 5. **Using Translations in Components**

To use translations in your components, you can use the `useTranslation` hook provided by `react-i18next`.

```js
// src/components/MyComponent.js
import React from 'react';
import { useTranslation } from 'react-i18next';

const MyComponent = () => {
  const { t } = useTranslation(); // `t` is the translation function

  return (
    <div>
      <h1>{t('welcome')}</h1> {/* Display translated text */}
      <button>{t('login')}</button>
    </div>
  );
};

export default MyComponent;
```

- **`t('key')`**: This is the translation function. It looks up the key in the current language and returns the corresponding translation.

#### 6. **Switching Languages**

You can switch languages by using the `i18n.changeLanguage()` function.

```js
// src/components/LanguageSwitcher.js
import React from 'react';
import { useTranslation } from 'react-i18next';

const LanguageSwitcher = () => {
  const { i18n } = useTranslation();

  const changeLanguage = (lng) => {
    i18n.changeLanguage(lng);
  };

  return (
    <div>
      <button onClick={() => changeLanguage('en')}>English</button>
      <button onClick={() => changeLanguage('fr')}>Français</button>
    </div>
  );
};

export default LanguageSwitcher;
```

Here, the `changeLanguage` function changes the language when the buttons are clicked.

#### 7. **Formatting Dates, Numbers, and Currencies**

You can also format dates, numbers, and currencies based on the selected language using **`react-i18next`** in conjunction with **`i18next`'s formatting features**.

Example of date formatting:

```js
import { useTranslation } from 'react-i18next';
import { useEffect, useState } from 'react';

const DateComponent = () => {
  const { i18n } = useTranslation();
  const [formattedDate, setFormattedDate] = useState('');

  useEffect(() => {
    const date = new Date();
    setFormattedDate(new Intl.DateTimeFormat(i18n.language).format(date));
  }, [i18n.language]);

  return <div>{formattedDate}</div>;
};

export default DateComponent;
```

### Key Features of `react-i18next`

1. **Language Switching**: Allows for dynamic language changes and supports multiple languages.
2. **Translation Loading**: Supports loading translations from various sources (local files, API, etc.).
3. **Pluralization**: Handles pluralization rules for different languages.
4. **Interpolation**: Supports variable interpolation in translations (e.g., `t('hello', { name: 'John' })`).
5. **Formatting**: Handles formatting of dates, numbers, and currencies based on the selected locale.

### Example Application Structure for i18n

```plaintext
src/
  ├── components/
  │   ├── MyComponent.js
  │   └── LanguageSwitcher.js
  ├── locales/
  │   ├── en/
  │   │   └── translation.json
  │   └── fr/
  │       └── translation.json
  ├── i18n.js
  ├── App.js
  ├── index.js
  └── styles.css
```

### Exam Power-Up:
- **i18next** is a powerful library for **internationalizing React apps**, allowing easy switching of languages and formatting of dates, numbers, and currencies.
- It supports **language detection**, **fallback languages**, and even advanced features like **pluralization** and **gender-based translations**.
- **`useTranslation()`** hook is the primary method for accessing translation functionality in React components.

Sample MCQ:
1. Which of the following is a commonly used library for internationalizing React apps?
   - A) react-router
   - B) react-i18next
   - C) redux-thunk
   - D) axios

Answer: B) react-i18next


In React, **lifecycle methods** are special methods that are automatically called at various stages of a component's existence. These methods allow you to run code at specific times during the lifecycle of a component, such as when it is created, updated, or destroyed.

In **React Functional Components**, lifecycle methods are handled using **React Hooks** like `useEffect`. In **Class Components**, they are handled through explicit lifecycle methods.

### React Class Component Lifecycle Methods

In **class components**, React provides several lifecycle methods that allow you to hook into different stages of a component's life. These are:

1. **Mounting** (When the component is being created and inserted into the DOM)
2. **Updating** (When the component is being re-rendered due to changes in props or state)
3. **Unmounting** (When the component is being removed from the DOM)

Here's an overview of the key lifecycle methods and their execution flow:

### 1. **Mounting Phase**
This is when the component is being created and inserted into the DOM.

- **constructor(props)**: Called when the component is created. It’s used to initialize state and bind methods.
- **getDerivedStateFromProps(nextProps, nextState)**: Called before every render, both on initial mount and on subsequent updates. It can be used to update state based on props. This method is static, meaning it doesn't have access to `this` (i.e., the instance of the component).
- **render()**: The core method that returns the JSX to render. It is required in every class component.
- **componentDidMount()**: Called after the component is inserted into the DOM. It is typically used for fetching data, setting up subscriptions, or triggering side effects.

### 2. **Updating Phase**
This phase occurs when the component’s state or props change.

- **getDerivedStateFromProps(nextProps, nextState)**: This method is called every time the component's props or state change before the render.
- **shouldComponentUpdate(nextProps, nextState)**: Allows you to optimize performance by preventing unnecessary re-renders. If it returns `false`, the component will not re-render.
- **render()**: Same as in the mounting phase, it returns the JSX for the component.
- **getSnapshotBeforeUpdate(prevProps, prevState)**: Called right before changes from the virtual DOM are applied to the DOM. It is often used to capture information (e.g., scroll position) before the update occurs.
- **componentDidUpdate(prevProps, prevState, snapshot)**: Called after the component updates and re-renders. It's useful for operations like network requests based on prop/state changes, or interacting with the DOM after updates.

### 3. **Unmounting Phase**
This phase occurs when the component is being removed from the DOM.

- **componentWillUnmount()**: Called before the component is removed from the DOM. It’s typically used for cleanup tasks like invalidating timers, canceling network requests, or cleaning up subscriptions.

### Example of Lifecycle Methods in Class Components

```js
import React, { Component } from 'react';

class Example extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    console.log('Constructor: Initializing state');
  }

  static getDerivedStateFromProps(nextProps, nextState) {
    console.log('getDerivedStateFromProps: Checking if state should be updated');
    return null; // No state update in this case
  }

  shouldComponentUpdate(nextProps, nextState) {
    console.log('shouldComponentUpdate: Should the component update?');
    return true; // Component will re-render
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('getSnapshotBeforeUpdate: Capturing info before changes are applied');
    return null; // No snapshot needed
  }

  componentDidMount() {
    console.log('componentDidMount: Component mounted');
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('componentDidUpdate: Component updated');
  }

  componentWillUnmount() {
    console.log('componentWillUnmount: Cleanup before unmounting');
  }

  render() {
    console.log('render: Rendering JSX');
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Increment
        </button>
      </div>
    );
  }
}

export default Example;
```

### React Functional Component Lifecycle with Hooks

In **functional components**, the lifecycle is managed with **React Hooks**. The primary hook for managing side effects (such as fetching data, subscribing to a service, or performing cleanup) is **`useEffect`**.

Here’s how the lifecycle methods translate in functional components:

1. **`useEffect`**: This hook can be used to replicate the behavior of `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.

   - **componentDidMount**: This is achieved by passing an empty dependency array (`[]`), so the effect runs only once, after the initial render.
   - **componentDidUpdate**: The effect runs after every render, by default, but you can pass a dependency array to control when it should run.
   - **componentWillUnmount**: This is done by returning a cleanup function inside the `useEffect` hook.

### Example of `useEffect` for Lifecycle in Functional Components

```js
import React, { useState, useEffect } from 'react';

const Example = () => {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate
  useEffect(() => {
    console.log('Component mounted or updated');
    document.title = `Count: ${count}`;

    // Cleanup similar to componentWillUnmount
    return () => {
      console.log('Cleanup before component unmounts or before next update');
    };
  }, [count]); // Effect depends on 'count'

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

export default Example;
```

### Key Points:

- **Mounting**: The component is created and inserted into the DOM. This phase includes `constructor`, `getDerivedStateFromProps`, `render`, and `componentDidMount`.
- **Updating**: The component is re-rendered due to state or prop changes. This phase includes `getDerivedStateFromProps`, `shouldComponentUpdate`, `render`, `getSnapshotBeforeUpdate`, and `componentDidUpdate`.
- **Unmounting**: The component is removed from the DOM. This phase includes `componentWillUnmount`.

### React Functional Component Lifecycle Summary

| Lifecycle Phase      | Class Component Methods            | Functional Component (Hooks)         |
|----------------------|-------------------------------------|--------------------------------------|
| **Mounting**          | `constructor`, `getDerivedStateFromProps`, `render`, `componentDidMount` | `useEffect` (with empty dependency array) |
| **Updating**          | `getDerivedStateFromProps`, `shouldComponentUpdate`, `render`, `getSnapshotBeforeUpdate`, `componentDidUpdate` | `useEffect` (with state/props as dependencies) |
| **Unmounting**        | `componentWillUnmount`             | `useEffect` (with cleanup function) |

### Execution Flow in React

Here’s the typical flow when a React component is mounted:

1. **Initial Render**:
   - `constructor()` (for class components) initializes state.
   - `getDerivedStateFromProps()` is called before the render method (for class components).
   - `render()` returns JSX.
   - `componentDidMount()` is called after the initial render (for class components).

2. **Subsequent Updates**:
   - If state or props change, React will re-render the component.
   - `getDerivedStateFromProps()` is called (for class components) before `render()`.
   - `shouldComponentUpdate()` is called to decide whether to re-render.
   - `render()` returns the updated JSX.
   - `getSnapshotBeforeUpdate()` is called (before the changes are applied to the DOM).
   - `componentDidUpdate()` is called after the component updates (for class components).

3. **Unmounting**:
   - `componentWillUnmount()` is called just before the component is removed from the DOM (for class components).
   - Cleanup activities (like timers, subscriptions, etc.) are handled inside the return function of `useEffect` in functional components.

### Exam Power-Up:
- **React Lifecycle Methods**: Class components have methods like `componentDidMount`, `shouldComponentUpdate`, and `componentWillUnmount` for managing component behavior during the mounting, updating, and unmounting phases.
- **`useEffect` Hook**: In functional components, `useEffect` replaces lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.
- **Performance Optimization**: Use `shouldComponentUpdate` (in class components) or `React.memo` (in functional components) to prevent unnecessary re-renders.

Sample MCQ:
1. Which React lifecycle method is called after the component is inserted into the DOM?
   - A) componentDidMount
   - B) shouldComponentUpdate
   - C) componentWillUnmount
   - D) componentDidUpdate

Answer: A) componentDidMount



    Let's dive into **Jest** and **React Testing Library** to write a **complex test** for a React component. This example will demonstrate how to test a **form component** that interacts with API data, handles form validation, and updates state based on user interaction.

### **Scenario: Testing a Registration Form**

We will build and test a **registration form** that:
- Takes **username** and **password** as input.
- Validates the inputs (e.g., username must be at least 5 characters long, and password must be at least 8 characters long).
- Simulates submitting the form.
- Displays success or error messages based on the form submission result (which mimics an API call).

### 1. **Creating the Registration Form Component**

#### `RegistrationForm.js`

```js
// src/components/RegistrationForm.js
import React, { useState } from 'react';

const RegistrationForm = ({ onSubmit }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);

  const validateForm = () => {
    if (username.length < 5) return 'Username must be at least 5 characters';
    if (password.length < 8) return 'Password must be at least 8 characters';
    return '';
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const validationError = validateForm();
    if (validationError) {
      setError(validationError);
      return;
    }

    setError('');
    setLoading(true);
    try {
      await onSubmit(username, password); // Mimic API call
      setSuccess(true);
    } catch (err) {
      setError('Registration failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h1>Registration Form</h1>
      <form onSubmit={handleSubmit}>
        <div>
          <label>Username:</label>
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            aria-label="username"
          />
        </div>
        <div>
          <label>Password:</label>
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            aria-label="password"
          />
        </div>
        {error && <p role="alert">{error}</p>}
        <button type="submit" disabled={loading}>
          {loading ? 'Submitting...' : 'Submit'}
        </button>
        {success && <p role="alert">Registration successful!</p>}
      </form>
    </div>
  );
};

export default RegistrationForm;
```

### 2. **Test File for `RegistrationForm`**

Now, let's write the test for this component using **Jest** and **React Testing Library**.

#### `RegistrationForm.test.js`

```js
// src/components/RegistrationForm.test.js
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import RegistrationForm from './RegistrationForm';

describe('RegistrationForm Component', () => {
  test('renders the form with username and password fields', () => {
    render(<RegistrationForm onSubmit={() => {}} />);

    // Check if the form fields are in the document
    expect(screen.getByLabelText(/username/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  test('shows validation error if username is less than 5 characters', async () => {
    render(<RegistrationForm onSubmit={() => {}} />);

    // Fill the form with invalid username
    fireEvent.change(screen.getByLabelText(/username/i), { target: { value: 'user' } });
    fireEvent.change(screen.getByLabelText(/password/i), { target: { value: 'password123' } });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));

    // Check for validation error
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Username must be at least 5 characters');
    });
  });

  test('shows validation error if password is less than 8 characters', async () => {
    render(<RegistrationForm onSubmit={() => {}} />);

    // Fill the form with invalid password
    fireEvent.change(screen.getByLabelText(/username/i), { target: { value: 'validuser' } });
    fireEvent.change(screen.getByLabelText(/password/i), { target: { value: 'short' } });

    fireEvent.click(screen.getByRole('button', { name: /submit/i }));

    // Check for validation error
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Password must be at least 8 characters');
    });
  });

  test('calls the onSubmit function with correct parameters when form is valid', async () => {
    const onSubmitMock = jest.fn().mockResolvedValue('success');
    
    render(<RegistrationForm onSubmit={onSubmitMock} />);
    
    fireEvent.change(screen.getByLabelText(/username/i), { target: { value: 'validuser' } });
    fireEvent.change(screen.getByLabelText(/password/i), { target: { value: 'validpassword123' } });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));

    await waitFor(() => {
      expect(onSubmitMock).toHaveBeenCalledTimes(1);
      expect(onSubmitMock).toHaveBeenCalledWith('validuser', 'validpassword123');
    });
  });

  test('shows success message after successful registration', async () => {
    const onSubmitMock = jest.fn().mockResolvedValue('success');
    
    render(<RegistrationForm onSubmit={onSubmitMock} />);
    
    fireEvent.change(screen.getByLabelText(/username/i), { target: { value: 'validuser' } });
    fireEvent.change(screen.getByLabelText(/password/i), { target: { value: 'validpassword123' } });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));

    // Check for the success message
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Registration successful!');
    });
  });

  test('shows error message if registration fails', async () => {
    const onSubmitMock = jest.fn().mockRejectedValue('error');
    
    render(<RegistrationForm onSubmit={onSubmitMock} />);
    
    fireEvent.change(screen.getByLabelText(/username/i), { target: { value: 'validuser' } });
    fireEvent.change(screen.getByLabelText(/password/i), { target: { value: 'validpassword123' } });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));

    // Check for error message
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Registration failed');
    });
  });
});
```

### **Explanation of the Tests**

1. **Rendering the Form**: 
   - We use `render()` to render the `RegistrationForm` component and verify that the **username** and **password** fields, and the **submit** button, are correctly rendered using `screen.getByLabelText()` and `screen.getByRole()`.

2. **Validation Tests**: 
   - We simulate user input and test form validation. For instance, when the username is less than 5 characters, the form should display an appropriate error message. We use `fireEvent.change()` to simulate user input and `fireEvent.click()` to submit the form.

3. **Successful Form Submission**: 
   - We mock the `onSubmit` function (which mimics an API call) using `jest.fn()`. The mock function is set to resolve with a success value (`mockResolvedValue()`).
   - After filling in the form, we check that `onSubmit` was called with the correct parameters and verify that the success message appears.

4. **Failed Form Submission**: 
   - We mock the `onSubmit` function to reject with an error (`mockRejectedValue()`). We then check if the error message "Registration failed" is shown after the form submission.

5. **Using `waitFor`**: 
   - We use `waitFor` to ensure that asynchronous code (like form submission and state updates) has time to complete before making assertions. This is particularly useful when dealing with API calls or state updates that may take some time.

### 3. **Running the Tests**

You can run your tests using Jest by running the following command in your terminal:

```bash
npm test
```

Jest will automatically detect and run the test files, displaying the results in your terminal.

### Conclusion

This example demonstrates how to:
- Render a component and verify its presence in the DOM.
- Simulate user interactions like filling out a form and clicking buttons.
- Validate form inputs and show appropriate error/success messages.
- Mock API calls and check how the component handles success or failure.

### Power-Up:
- **`fireEvent`** simulates user interactions like typing, clicking, and changing values.
- **`waitFor`** ensures your tests wait for asynchronous tasks to complete before assertions are made.
- **Jest mocks** (`jest.fn()`) are powerful for testing functions that involve side effects, like API calls.

Sample MCQ:
1. Which function from React Testing Library is used to simulate user events in a component?
   - A) `render()`
   - B) `screen.getByText()`
   - C) `fireEvent()`
   - D) `waitFor()`

Answer: C) `fireEvent()`

### **Jest Mocking: An Overview**

**Jest** provides powerful mocking features that help in testing components, functions, and modules by replacing real implementations with mock implementations. Mocking allows you to isolate the part of your application you're testing and control the behavior of dependencies like API calls, modules, and functions.

### Types of Jest Mocking:
1. **Mock Functions (`jest.fn()`)**: Mock functions can be used to track calls to a function, return custom values, or simulate behavior.
2. **Mocking Modules (`jest.mock()`)**: Mocking modules allows you to replace the entire module with a mock implementation, often used for API calls, utilities, or third-party libraries.
3. **Mocking Timers (`jest.useFakeTimers()`)**: Jest allows you to mock timers to test functions that rely on `setTimeout`, `setInterval`, or other time-based APIs.
4. **Mocking Implementation of Functions**: Mocking the behavior of specific functions within a module using `jest.fn()`.

### 1. **Mocking Functions with `jest.fn()`**

The `jest.fn()` method creates a mock function that you can use to track its calls, arguments, and return values.

#### Example 1: Basic Mock Function

```js
test('should call mock function', () => {
  const mockFn = jest.fn();
  
  mockFn();
  
  // Check that mock function was called
  expect(mockFn).toHaveBeenCalledTimes(1);
});
```

#### Example 2: Mock Function with Return Values

You can control the return value of a mock function to simulate various scenarios:

```js
test('mock function should return a value', () => {
  const mockFn = jest.fn(() => 'mocked value');
  
  const result = mockFn();
  
  // Check the result of the mock function
  expect(result).toBe('mocked value');
});
```

#### Example 3: Mocking Implementation of a Function

You can mock the implementation of a function using `mockImplementation`.

```js
test('mock function should use a custom implementation', () => {
  const mockFn = jest.fn().mockImplementation(() => 'mocked implementation');
  
  const result = mockFn();
  
  expect(result).toBe('mocked implementation');
});
```

### 2. **Mocking Modules with `jest.mock()`**

You can mock entire modules, including external libraries like Axios or Firebase, to simulate their behavior without actually calling real endpoints.

#### Example 1: Mocking a Module

Let's say you have a module that fetches data from an API:

```js
// api.js
export const fetchData = async () => {
  const response = await fetch('https://api.example.com/data');
  return response.json();
};
```

You can mock this module in your test to avoid making actual API calls.

```js
// api.js
import { fetchData } from './api';

// Test using the mocked fetchData
test('should fetch data and return response', async () => {
  // Mock the fetchData function
  jest.mock('./api', () => ({
    fetchData: jest.fn().mockResolvedValue({ data: 'mocked data' })
  }));

  const result = await fetchData();

  expect(result).toEqual({ data: 'mocked data' });
});
```

#### Example 2: Mocking a Third-Party Module (e.g., Axios)

Let's say you're testing a component that makes an HTTP request using Axios. You can mock Axios to avoid actual network requests.

```js
import axios from 'axios';
import { fetchData } from './api'; // Assume fetchData uses axios internally

jest.mock('axios');

test('should fetch data using axios', async () => {
  axios.get.mockResolvedValue({ data: 'mocked data' });

  const result = await fetchData(); // Assume fetchData uses axios.get
  expect(result).toEqual('mocked data');
});
```

In this case:
- **`jest.mock('axios')`** automatically mocks Axios.
- **`axios.get.mockResolvedValue()`** tells Jest to return a resolved promise with the provided data when `axios.get` is called.

### 3. **Mocking Timers with `jest.useFakeTimers()`**

When testing functions that rely on JavaScript timers, such as `setTimeout`, `setInterval`, etc., you can mock these functions to control their execution timing.

#### Example 1: Using `jest.useFakeTimers()` with `setTimeout`

```js
test('should call setTimeout after 1 second', () => {
  jest.useFakeTimers();
  
  const mockCallback = jest.fn();

  setTimeout(mockCallback, 1000);
  
  // Fast-forward 1 second
  jest.advanceTimersByTime(1000);
  
  // Assert the mock function was called
  expect(mockCallback).toHaveBeenCalled();
});
```

- **`jest.useFakeTimers()`**: Replaces the global timer functions (`setTimeout`, `setInterval`) with mock implementations.
- **`jest.advanceTimersByTime()`**: Simulates the passing of time (e.g., for testing how timeouts behave).

#### Example 2: Mocking `setInterval`

```js
test('should call setInterval every 1 second', () => {
  jest.useFakeTimers();
  
  const mockCallback = jest.fn();

  setInterval(mockCallback, 1000);
  
  // Fast-forward time by 3 seconds
  jest.advanceTimersByTime(3000);
  
  // Assert that the function was called 3 times
  expect(mockCallback).toHaveBeenCalledTimes(3);
});
```

### 4. **Mocking Functions in Class Components**

If you are testing a class component with methods, you can mock instance methods directly.

#### Example 1: Mocking Class Method

```js
class MyClass {
  fetchData() {
    return 'real data';
  }
}

test('should mock class method', () => {
  const mockClass = new MyClass();
  mockClass.fetchData = jest.fn().mockReturnValue('mocked data');

  const result = mockClass.fetchData();

  expect(result).toBe('mocked data');
});
```

### 5. **Mocking Imported Functions**

You may need to mock functions that are imported into your component or module. Here's how you can do that:

#### Example 1: Mocking an Imported Function

```js
import * as api from './api';

jest.mock('./api'); // Automatically mocks all functions in './api'

test('should call fetchData from api', async () => {
  api.fetchData.mockResolvedValue({ data: 'mocked data' });

  const result = await api.fetchData();

  expect(result).toEqual({ data: 'mocked data' });
});
```

### 6. **Mocking React Components**

You can mock React components if you want to isolate certain parts of your application.

#### Example 1: Mocking a Child Component

```js
import React from 'react';
import { render } from '@testing-library/react';
import ParentComponent from './ParentComponent';

jest.mock('./ChildComponent', () => () => <div>Mocked Child</div>);

test('should render mocked child component', () => {
  const { getByText } = render(<ParentComponent />);
  expect(getByText('Mocked Child')).toBeInTheDocument();
});
```

### 7. **Resetting Mocks**

After a test, you may want to reset mocks to ensure no state is carried over to the next test. You can do this with `jest.clearAllMocks()` or `jest.resetAllMocks()`.

#### Example 1: Clearing Mocks After Each Test

```js
afterEach(() => {
  jest.clearAllMocks(); // Clears mock function calls and reset state
});
```

### Conclusion

Jest's mocking utilities provide great flexibility for isolating and testing individual components and functions, without the need to interact with external services, APIs, or modules.

### Exam Power-Up:
- **`jest.fn()`** is used to mock functions, track calls, and return custom values.
- **`jest.mock()`** is used to mock entire modules, including third-party libraries and internal utilities.
- **`jest.useFakeTimers()`** is used to mock JavaScript timers (`setTimeout`, `setInterval`).
- **`jest.clearAllMocks()`** and **`jest.resetAllMocks()`** help reset mocks between tests to avoid shared state.

Sample MCQ:
1. Which Jest function is used to create a mock function that can be tracked?
   - A) `jest.mock()`
   - B) `jest.fn()`
   - C) `jest.useFakeTimers()`
   - D) `jest.clearAllMocks()`

Answer: B) `jest.fn()`

