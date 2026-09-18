---
title: "Refactoring, register page, and router"
parent_post: Express-Auth
module_number: 6
layout: module
media_subpath: /assets/tutorials/express-auth
---

### Refactoring and shared types 

As our application grows, you might notice that our `App.tsx` is becoming quite large and doing many things at once. Let's refactor our application to be more maintainable and scalable. 

#### Component refactoring

First move the login form into its own component: 

```tsx
import { useState, type FormEvent } from "react";
import { useNavigate } from "react-router-dom";

interface LoginFormProps {
  handleLogin: (username: string, password: string) => void;
}

const LoginForm = ({ handleLogin }: LoginFormProps) => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const navigate = useNavigate();

  const onSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    handleLogin(username, password);
  };

  return (
    <>
      <form onSubmit={onSubmit}>
        <div>
          username
          <input
            type="text"
            value={username}
            name="Username"
            onChange={(e) => setUsername(e.target.value)}
          />
        </div>
        <div>
          password
          <input
            type="password"
            value={password}
            name="Password"
            onChange={(e) => setPassword(e.target.value)}
          />
        </div>
        <button type="submit">Login</button>
      </form>
      <button type="button" onClick={() => navigate("/register")}>Register</button>
    </>
  );
};

export default LoginForm;
```
{: file="frontend/src/components/LoginForm.tsx"}
{: .nolineno}

Next, let's encapsulate authentication state and logic into a custom hook `useLogin`: 

```tsx
import { useState, useEffect } from "react";
import type { Contact, JwtPayload } from "@shared/types";
import * as loginService from "../services/loginService";
import * as contactService from "../services/contactService";
import { jwtDecode } from "jwt-decode";

export function useLogin() {
  const [jwt, setJwt] = useState<string | null>(null);
  const [contacts, setContacts] = useState<Contact[]>([]);

  const payload = jwt !== null 
    ? jwtDecode<JwtPayload>(jwt)
    : null;

  useEffect(() => {
    if (payload !== null && jwt) {
      contactService.setToken(jwt);
      contactService.getAll().then((data) => {
        setContacts(data.filter(
          contact => contact.belongsTo?.username === payload.username
        ));
      });
    }
  }, [payload, jwt]);

  const handleLogin = async (username: string, password: string) => {
    try {
      const response = await loginService.login({ username, password });
      setJwt(response.token);
      contactService.setToken(response.token);
      window.localStorage.setItem("JwtAccessToken", response.token);
    } catch (error) {
      console.error("Login failed:", error);
    }
  };

  return {
    jwt,
    payload,
    contacts,
    handleLogin,
  };
}
```
{: file="frontend/src/hooks/useLogin.ts"}
{: .nolineno}

Now, let's declare concrete TypeScript interfaces in `@shared/types.ts` to ensure type safety across our backend, hooks, and services:

```tsx
export interface LoginRequest {
  username: string;
  password: string;
}

export interface LoginResponse {
  token: string;
}

export interface RegisterRequest {
  username: string;
  password: string;
  name: string;
  email: string;
}

export interface Contact {
  id: string;
  name: string;
  number: string;
  belongsTo: {
    username: string;
    name?: string;
    id?: string;
  };
}
```
{: file="@shared/types.ts"}
{: .nolineno}

Although not specifying `LoginRequest` for the credentials does not result in warning, it is good practice to do so. Imagine having hundreds of types of request: `ContactRequest`, `DeleteRequest`, `UpdateRequest`, etc., you will quickly be overwhelmed and lose track of what are which if the types are not concrete. You should also do another `LoginResponse`. 

Next, refactor the contact displaying part into its own component: 

```tsx
import type { Contact } from '@shared/types';

interface ContactDisplayProps {
  contacts: Contact[];
  username: string;
}

const ContactDisplay = ({ contacts }: ContactDisplayProps) => {
  return (
    <div>
      <h2>Your Contacts</h2>
      {contacts.map((contact, index) => (
        <div key={index}>
          {contact.name} {contact.number}
        </div>
      ))}
    </div>
  );
};

export default ContactDisplay;
```
{: file="frontend/src/components/ContactDisplay.tsx"}
{: .nolineno }

Here notice that `ContactDisplayProps` is directly defined inside the file. We certainly know that this interface is only needed inside this function and this file (where else in the code we need to display contacts like this?), so we can declare the interface straight in the file. This is mostly up to personal taste. 

Finally, after refactoring, our `App.tsx` will be much cleaner:

```tsx
import { BrowserRouter as Router } from "react-router-dom";
import LoginForm from "./components/LoginForm";
import ContactDisplay from "./components/ContactDisplay";
import { useLogin } from "./hooks/useLogin";

function App() {
  const { payload, contacts, handleLogin } = useLogin();

  return (
    <Router>
      <h1>login</h1>
      <LoginForm handleLogin={handleLogin} />
      {payload !== null && (
        <ContactDisplay contacts={contacts} username={payload.username} />
      )}
    </Router>
  );
}

export default App;
```
{: file="frontend/src/App.tsx"}
{: .nolineno}

#### API refactoring 

Before we move on to the next part, let's refactor our service code for better organization. The main idea is to separate all Axios-related API calls into dedicated *service* modules. This creates a clean separation between our UI logic and API communication.

For example, instead of handling API calls directly in our components like this:

```tsx
const handleLoginBackend = async (credentials: Credentials) => {
    const baseUrl = "/api/login";

    try {
      const response = await axios.post(baseUrl, credentials);
      const jwt = response.data;

      setJwt(jwt);
    } catch (error) {
      console.error("Login failed:", error);
    }
};
```
{: .nolineno}

We can refactor it to use a dedicated service like this:

```tsx
const handleLoginBackend = async (credentials: LoginRequest) => {
    try {
      const response = await loginService.login(credentials);
      setJwt(response);
    } catch (error) {
      console.error("Login failed:", error);
    }
};
```
{: .nolineno}

The refactored code is much cleaner and more descriptive. Instead of having to parse through implementation details to understand what a function does, we can immediately understand its purpose from the service method name. This follows a key principle in app design: **keep specific implementation details separate from general business logic**.

Let's create a new `loginService` file under `frontend/src/services`:

```tsx
import axios from "axios";
import type { LoginRequest, LoginResponse } from '@shared/types';

const baseUrl = "/api/login";

export const login = async (credentials: LoginRequest): Promise<LoginResponse> => {
  const response = await axios.post(baseUrl, credentials);
  return response.data;
};
```
Similarly, let's create `registerService.ts` for registration requests:

```tsx
import axios from "axios";
import type { RegisterRequest } from '@shared/types';

const baseUrl = "/api/register";

export const register = async (userData: RegisterRequest) => {
  const response = await axios.post(baseUrl, userData);
  return response.data;
};
```
{: file="frontend/src/services/registerService.ts" }
{: .nolineno }

And refactor all contact-related Axios calls into `contactService.ts` to keep API communication decoupled from UI rendering:

```tsx
import axios from 'axios';
import type { Contact } from '@shared/types';

const baseUrl = '/api/contacts';
let token: string = '';

export const setToken = (newToken: string) => {
  token = newToken;
};

export const getAll = async (): Promise<Contact[]> => {
  const config = {
    headers: { Authorization: `Bearer ${token}` }
  };
  const response = await axios.get(baseUrl, config);
  return response.data;
};

export const create = async (newContact: { name: string; number: string }): Promise<Contact> => {
  const config = {
    headers: { Authorization: `Bearer ${token}` }
  };
  const response = await axios.post(baseUrl, newContact, config);
  return response.data;
};

export const remove = async (id: string): Promise<void> => {
  const config = {
    headers: { Authorization: `Bearer ${token}` }
  };
  await axios.delete(`${baseUrl}/${id}`, config);
};
```
{: file="frontend/src/services/contactService.ts" }
{: .nolineno }

### Register page and React Router

Now we can create a register page for new users to sign up using `react-router-dom`:

```bash
npm install react-router-dom
```

Let's create our `RegisterForm` component:

```tsx
import type { RegisterRequest } from "@shared/types";
import { useState, type FormEvent } from "react";
import * as registerService from "../services/registerService";
import { useNavigate } from "react-router-dom";

const RegisterForm = () => { 
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const navigate = useNavigate();

  const handleSubmit = async (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    try {
      const registerData: RegisterRequest = {
        username, 
        password,
        name, 
        email
      };

      await registerService.register(registerData);
      navigate("/");
    } catch (err) {
      console.error(err);
    }
  };
  
  return (
    <>
      <h1>Register</h1>
      <form onSubmit={handleSubmit}>
        <div>
          username
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />
        </div>
        <div>
          name
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
          />
        </div>
        <div>
          email
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
          />
        </div>
        <div>
          password
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          />
        </div>
        <button type="submit">Register</button>
      </form>
      <button type="button" onClick={() => navigate("/")}>Cancel</button>
    </>
  );
};

export default RegisterForm;
```
{: file="frontend/src/components/RegisterForm.tsx"}
{: .nolineno}

The `useNavigate` hook is used to navigate to a different page. In our logic, after the registration success, we will be redirected to the default page `/` (which is currently where our login page is located). We also add a matching `Register` button inside `LoginForm` to allow users to navigate to `/register` using `useNavigate()`.

After that, in `App.tsx`: 

```tsx
// ... 
import { BrowserRouter as Router, Routes, Route, Navigate } from "react-router-dom";


function App() {
  const { payload, contacts, handleLogin } = useLogin();

  return (
    <>
      <Router>
      <Routes>
        <Route path="/" element={
          <>
            <h1>Login</h1>
            <LoginForm handleLogin={handleLogin} />
            {payload !== null && (
              <ContactDisplay contacts={contacts} username={payload.username} />
            )}
          </>
        } />
        <Route path="/register" element={<RegisterForm />} />
      </Routes>
    </Router>
    </>
  );
}
```
{: file="frontend/src/App.tsx"}
{: .nolineno}

Now we have three new keywords here: `Router`, `Routes`, and `Route`. `Router` (or actually `BrowserRouter`) wraps our entire application and enables routing, as well as managing the current endpoint and navigation history. The `Routes` is a container that group different `Route` into a collection, and ensure only one `Route` in the group will render at one time. Finally, `Route` should be pretty self-explanatory.
