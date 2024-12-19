# Using PrivateRoute with Redux Toolkit in React.js: A Complete Guide

When building React applications, you often need to protect certain routes so only authenticated users can access them. By combining **React Router**, **Redux Toolkit**, and **Redux Persist**, you can efficiently manage authentication and create a robust private routing solution.

This guide demonstrates how to implement a `PrivateRoute` component using **Redux Toolkit** for state management. We'll provide a clean and scalable approach for React projects.

---

## **Why Use PrivateRoute with Redux Toolkit?**

1. **Separation of Concerns**: Routes are protected at the component level, keeping your logic clean.
2. **Centralized State Management**: Redux Toolkit handles the authentication state across your app.
3. **Persistence**: Redux Persist ensures your authentication state persists after refreshing the page.
4. **Scalable and Maintainable**: This approach works for apps of all sizes and integrates smoothly with React Router.

---

## **Folder Structure**

Ensure your project has the following structure:

```bash
src/
├── api/
│   └── API.js
├── app/
│   └── store.js
├── features/
│   └── authSlice.js
├── components/
│   └── PrivateRoute.js
└── pages/
    └── Contact.js
```

---

## **Step 1: Set Up the Redux Auth Slice**

The `authSlice.js` file manages the authentication state. For this example, we'll focus only on the `signin` functionality.

### **authSlice.js**

```jsx
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";
import API from "../api";

// Async thunk for user sign-in
export const signin = createAsyncThunk("auth/signin", async (loginData, { rejectWithValue }) => {
    try {
        const response = await API.post("/users/signin", loginData);
        return response.data;
    } catch (error) {
        return rejectWithValue(error.response.data || { message: error.message });
    }
});

const initialState = {
    user: null,
    isAuthenticated: false,
    loading: false,
    error: null,
};

const authSlice = createSlice({
    name: "auth",
    initialState,
    reducers: {
        clearError: (state) => {
            state.error = null;
        },
    },
    extraReducers: (builder) => {
        builder
            .addCase(signin.pending, (state) => {
                state.loading = true;
                state.error = null;
            })
            .addCase(signin.fulfilled, (state, action) => {
                state.loading = false;
                state.isAuthenticated = true;
                state.user = action.payload;
            })
            .addCase(signin.rejected, (state, action) => {
                state.loading = false;
                state.error = action.payload.message || "Login failed.";
            });
    },
});

export const { clearError } = authSlice.actions;
export default authSlice.reducer;
```

---

## **Step 2: API Configuration**

The `API.js` file sets up an Axios instance for HTTP requests with a base URL. This configuration allows you to reuse the API client across your project.

### **api.js**

```jsx
import axios from "axios";

const API = axios.create({
    baseURL: import.meta.env.VITE_SERVER_URL, // Environment variable for the server URL
    headers: {
        "Content-Type": "application/json",
    },
    withCredentials: true, // Include cookies for authentication
});

export default API;
```

---

## **Step 3: Configure the Store with Redux Persist**

The store ensures the auth state persists across page reloads.

### **store.js**

```jsx
import { configureStore } from "@reduxjs/toolkit";
import { persistStore, persistReducer } from "redux-persist";
import storage from "redux-persist/lib/storage";
import authReducer from "../features/authSlice";

const authPersistConfig = {
    key: "auth",
    storage,
    whitelist: ["isAuthenticated", "user"],
};

const persistedAuthReducer = persistReducer(authPersistConfig, authReducer);

const store = configureStore({
    reducer: {
        auth: persistedAuthReducer,
    },
    middleware: (getDefaultMiddleware) =>
        getDefaultMiddleware({
            serializableCheck: false, // Ignore Redux Persist actions
        }),
});

export const persistor = persistStore(store);
export default store;
```

---

## **Step 4: Create the PrivateRoute Component**

The `PrivateRoute` component checks if the user is authenticated before rendering the desired route. If not, it redirects to the home page.

### **PrivateRoute.js**

```jsx
import React from "react";
import { useSelector } from "react-redux";
import { Navigate } from "react-router-dom";

const PrivateRoute = ({ children }) => {
    const isAuthenticated = useSelector((state) => state.auth.isAuthenticated);
    return isAuthenticated ? children : <Navigate to="/" replace />;
};

export default PrivateRoute;
```

---

## **Step 5: Protect Routes in App.js**

Use the `PrivateRoute` component to wrap protected routes in your app.

### **App.js**

```jsx
import React from "react";
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import { Provider } from "react-redux";
import { PersistGate } from "redux-persist/integration/react";
import store, { persistor } from "./app/store";
import PrivateRoute from "./components/PrivateRoute";
import Home from "./pages/Home";
import Contact from "./pages/Contact";

const App = () => {
    return (
        <Provider store={store}>
            <PersistGate loading={null} persistor={persistor}>
                <Router>
                    <Routes>
                        <Route path="/" element={<Home />} />
                        <Route
                            path="/contact"
                            element={
                                <PrivateRoute>
                                    <Contact />
                                </PrivateRoute>
                            }
                        />
                    </Routes>
                </Router>
            </PersistGate>
        </Provider>
    );
};

export default App;
```

---

## **How It Works**

1. **Redux Toolkit** manages the authentication state (`authSlice`).
2. **Redux Persist** ensures the authentication state is saved in `localStorage`.
3. The **PrivateRoute** component checks if the user is authenticated using Redux state.
4. If the user is not authenticated, they are redirected to the home page (`"/"`).
5. Routes requiring authentication are wrapped in the `PrivateRoute` component.

---

## **Benefits of This Approach**

1. **Clean and Reusable**: The `PrivateRoute` component is simple and reusable.
2. **Persistent Authentication**: Redux Persist keeps the auth state even after refreshing.
3. **Centralized State Management**: Authentication logic is managed in a single slice.
4. **Scalable**: Works seamlessly as your app grows.

---

## **Conclusion**

By combining React Router, Redux Toolkit, and Redux Persist, you can easily implement a clean and robust private routing system in your React app. This approach is scalable, maintainable, and ensures a seamless user experience.

If you follow the steps above, your routes will be protected effectively, and your authentication state will persist across sessions.

**Happy Coding! 🚀**
