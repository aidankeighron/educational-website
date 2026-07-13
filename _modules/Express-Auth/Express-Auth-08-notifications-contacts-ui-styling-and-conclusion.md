---
title: "Notifications, contacts UI, styling, and conclusion"
parent_post: Express-Auth
module_number: 8
layout: module
media_subpath: /assets/tutorials/express-auth
---

### Notification and React Context

As our application grows, we want to provide user feedback for various actions - success messages when contacts are added, error messages when operations fail, login confirmations, etc. We want notifications that can appear from anywhere in our application: login forms, contact management, registration, and more.

#### Prop drilling

If we tried to implement notifications the traditional way, we'd face a problem called **prop drilling**. Here's what it would look like:

```tsx
// App.tsx - top level component
function App() {
  const [notification, setNotification] = useState(null);
  
  return (
    <LoginForm 
      handleLogin={handleLogin} 
      setNotification={setNotification}  // Pass down
    />
  );
}

// LoginForm.tsx - needs to pass it further down
const LoginForm = ({ handleLogin, setNotification }) => {
  return (
    <SomeChildComponent 
      setNotification={setNotification}  // Pass down again
    />
  );
}

// And this continues for every component that needs notifications...
```

This becomes messy quickly. Every component in the chain needs to accept and pass down notification props, even if they don't use them themselves. 

#### React Context

React Context provides a way to share data across components without prop drilling. It's like creating a "global" state that any component can access directly. It consists of three main parts: 

1. **Context**: A "container" that holds the data you want to share
2. **Provider**: A component that supplies the data to its children
3. **Consumer**: Components that use the shared data (via hooks like `useContext`)

Here's how the pattern works:

```tsx
// 1. Create the Context 
const MyContext = createContext();

// 2. Create a Provider 
function MyProvider({ children }) {
  const [data, setData] = useState("some data");
  
  return (
    <MyContext.Provider value={{ data, setData }}>
      {children}  {/* All children can now access this data */}
    </MyContext.Provider>
  );
}

// 3. Use the Context in any child component 
function SomeChildComponent() {
  // need this line to access data even if component is wrapped inside provider
  const { data, setData } = useContext(MyContext);
  return <div>{data}</div>;
}
```

The key insight is that **any component wrapped by the Provider can access the context data**, no matter how deeply nested it is. You should also look up the `{children}` property if you don't know what it is - this is valid code. 

Now let's implement our notification context:

```tsx
import { createContext } from 'react';
import type { NotificationType } from '@shared/types';

interface NotificationContextType {
  notification: NotificationType | null;
  setNotification: React.Dispatch<React.SetStateAction<NotificationType | null>>;
}

export const NotificationContext = createContext<NotificationContextType>({
  notification: null,
  setNotification: () => {}
});
```
{: file="frontend/src/contexts/NotificationContext.tsx"}
{: .nolineno}

Don't be bothered by the long types - those are just boilerplates to avoid warnings. And by the way, for `NotificationType`: 

```ts
export interface NotificationType {
  msg: string,
  type: string
}
```
{: file="@shared/types.ts"}
{: .nolineno}

Next, create a provider component that manages the notification state and renders notifications:

```tsx
import { useState } from 'react';
import type { NotificationType } from '@shared/types';
import { NotificationContext } from '../contexts/NotificationContext';
import '../styles/index.css';

export const NotificationContextProvider = ({ children }: { children: React.ReactNode }) => {
  const [notification, setNotification] = useState<NotificationType | null>(null);

  return (
    <NotificationContext.Provider value={{ notification, setNotification }}>
      {notification && (
        <div className={notification.type}>
          {notification.msg}
        </div>
      )}
      {children}
    </NotificationContext.Provider>
  );
};
```
{: file="frontend/src/components/Notification.tsx"}
{: .nolineno}

In order to utilize this provider, we need to wrap components inside `NotificationContextProvider`. In our simple app, everywhere needs notification. So the easiest way can be wrap it around our `App` in `main.tsx`: 

```tsx
// ...

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <NotificationContextProvider>
      <App />
    </NotificationContextProvider>
  </StrictMode>,
)
```
{: file="frontend/src/main.tsx"}
{: .nolineno}

By wrapping our entire app with `NotificationContextProvider`, we create this component hierarchy:

```
NotificationContextProvider (provides notification state)
└── App
    ├── Router
    │   ├── LoginForm (can use notifications)
    │   ├── RegisterForm (can use notifications)
    │   └── Homepage
    │       ├── ContactForm (can use notifications)
    │       └── ContactList (can use notifications)
    └── Any other components (all can use notifications)
```

Next, create a custom hook to make using notifications easier:

```tsx
import { useContext } from "react";
import { NotificationContext } from "../contexts/NotificationContext";

export const useNotification = () => {
  // this extracts [notification, setNotification] from NotificationContext
  const { notification, setNotification } = useContext(NotificationContext);
  
  const showNotification = (msg: string, type: string) => {
      setNotification({ msg, type });
      setTimeout(() => setNotification(null), 5000);
  };
  
  return { notification, setNotification, showNotification };
}
```
{: file="frontend/src/hooks/useNotification.ts"}
{: .nolineno}

Now any component can show notifications without prop drilling:

```tsx
import { useNotification } from '../hooks/useNotification';

const LoginForm = ({ handleLogin }) => {
  const { showNotification } = useNotification();
  
  const onSubmit = async (event) => {
    try {
      await handleLogin(username, password);
      showNotification('Login successful!', 'success');
    } catch (error) {
      showNotification('Login failed', 'error');
    }
  };
  
  // ... rest of component
};
```

### Adding new contacts

The final part is to add a small field to add new contacts to an user like this:

![](Pasted image 20250716002227.png)

This is no different from the login form so you should do it yourself :)

### Add styling 

Currently our app have no styling at all. You can improve it by adding more CSS/Tailwind/MUI/ etc. in order to improve the appearance of the app. 

There is no right answer to this. But for example, mine look like this:

```css
/* Homepage Styles */
.homepage-container {
  max-width: 600px;
  margin: 40px auto;
  background: #fff;
  border-radius: var(--border-radius);
  box-shadow: var(--shadow-light);
  padding: 32px 24px;
}
.homepage-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}
.homepage-user {
  font-weight: 600;
  color: var(--primary-color);
}
.homepage-logout {
  background: var(--error-color);
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 8px 16px;
  cursor: pointer;
  transition: var(--transition);
}
.homepage-logout:hover {
  background: #b91c1c;
}
.homepage-title {
  margin-top: 0;
  color: var(--primary-color);
}
.contacts-list {
  margin-bottom: 24px;
}
.contact-card {
  display: flex;
  justify-content: space-between;
  background: #f3f4f6;
  border-radius: 8px;
  padding: 10px 16px;
  margin-bottom: 8px;
  box-shadow: var(--shadow-light);
}
.contact-name {
  font-weight: 500;
}
.contact-number {
  color: #6b7280;
}
.add-contact-form {
  background: #f9fafb;
  border-radius: 8px;
  padding: 16px;
  box-shadow: var(--shadow-light);
}
.form-group {
  margin-bottom: 16px;
  display: flex;
  flex-direction: column;
}
.form-group label {
  font-weight: 500;
  margin-bottom: 6px;
}

/* etc. */
```
{: file="frontend/src/styles/index.css"}
{: .nolineno}

Refer back to the gif at the beginning of the guide to see the full design.

## Conclusion

Congratulations! You've built a complete full-stack contact management application with TypeScript, React, and Express. This application demonstrates several important concepts:

- **Authentication**: Secure login and registration with JWT tokens
- **Data Management**: Creating and retrieving contacts from a MongoDB database
- **Type Safety**: Using TypeScript for type checking across the stack
- **User Experience**: Notifications, form validation, and proper navigation
- **Code Organization**: Clean separation of concerns with components, hooks, and services

Happy coding!
