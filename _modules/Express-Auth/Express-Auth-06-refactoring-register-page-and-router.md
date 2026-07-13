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
import React, { useState } from "react";

interface LoginFormProps {
  handleLogin: (username: string, password: string) => void;
}

const LoginForm = ({ handleLogin }: LoginFormProps ) => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const onSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    handleLogin(username, password);
  };

  return (
    <>
      <form onSubmit={onSubmit}>
        {/* input */}
        <button type="submit">Login</button>
      </form>
      <button onClick={registerRedirect}>Register</button>
    </>
  );
};

export default LoginForm;

```
{: file="frontend/src/components/LoginForm.tsx}
{: .nolineno}

But then how about the backend handling part (`handleLogin`)? We are also going to refactor it into another file, `useLogin`: 

```tsx
import { useState, useEffect } from "react";
import type { LoginRequest, Contact, JwtPayload } from "@shared/types";
import axios from "axios";
import { jwtDecode } from "jwt-decode";

export function useLogin() {
  const [jwt, setJwt] = useState(null);
  const [contacts, setContacts] = useState<Contact[]>([]);

  const payload = jwt !== null 
    ? jwtDecode<JwtPayload>(jwt)
    : null;

  useEffect(() => {
    if (payload !== null) {
      console.log(jwt);
      const contactUrl = "/api/contacts";
      const token = jwt.token;

      const config = {
      headers: { Authorization: `Bearer ${token}` },
      };

      axios.get(contactUrl, config).then((response) => {
      setContacts(response.data.filter(
        contact => contact.belongsTo.username === payload.username
      ))
	  }) 
	}
   }, [payload]); 

  const handleLogin = async (username: string, password: string) => {
    // ...
  };


  return {
    payload,
    contacts,
    handleLogin,
  };
}

```
{: file="frontend/src/hooks/useLogin.tsx"}
{: .nolineno}

> Task: Define the types used in this file that you have not defined in `types.ts`.
{: .prompt-tip}

**Answer (click to unblur):**

```tsx
export interface LoginRequest {
  username: string, 
  password: string 
};

export interface Contact {
  id: string,
  name: string, 
  number: string,
  belongsTo: {
    username: string
  }
};

```
{: file="@shared/types.ts}
{: .nolineno}
{: .blur}

Although not specifying `LoginRequest` for the credentials does not result in warning, it is good practice to do so. Imagine having hundreds of types of request: `ContactRequest`, `DeleteRequest`, `UpdateRequest`, etc., you will quickly be overwhelmed and lose track of what are which if the types are not concrete. You should also do another `LoginResponse`. 

Next, refactor the contact displaying part into its own component: 

```tsx
import React from 'react';
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
import LoginForm from "./components/LoginForm";
import ContactDisplay from "./components/ContactDisplay";
import { useLogin } from "./hooks/useLogin";

function App() {
  const {payload, contacts, handleLogin} = useLogin();

  return (
    <>
      <h1>login</h1>
      <LoginForm handleLogin={handleLogin} />
      {payload !== null && (
        <ContactDisplay contacts={contacts} username={payload.username} />
      )}
    </>
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
{: file="frontend/src/services/loginService.ts" }
{: .nolineno }

Notice the `Promise<LoginResponse>` return type annotation. This is a best practice - you should always define strict data types for your function inputs and outputs. You may want to refer back to your `loginController` to define the appropriate data type structure for `LoginResponse`. After that you should refactor the whole application before moving on. 

> **Task**: Refactor your `Contact` API calls using the same service pattern, and create a dedicated service file for any place where you're making direct API calls in your current code.
{: .prompt-tip}

### Register page and React Router

Now we can create a register page for new users to sign up.

Let's start with a basic register form component:

```tsx
import type { RegisterRequest } from "@shared/types";
import React, { useState } from 'react';
import * as registerService from '../services/registerService';

const RegisterForm = () => { 
  // ... states
  
  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    try {
      const registerData: RegisterRequest = {
        username, 
        password,
        name, 
        email
      }

      await registerService.register(registerData);
    } catch (err) {
      console.error(err);
    }
  }
  
  return (
    <>
      <h1>Register</h1>
      <form onSubmit={handleSubmit}>
        {/* name, email, username, password */}
        <button>Register</button>
      </form>
    </>
  )
}

export default RegisterForm;
```
{: file="frontend/src/components/RegisterForm.tsx}
{: .nolineno}

The question now is: where do we put this page? Using conditional rendering for multiple pages becomes very complicated as our app grows. Instead, we're going to develop our app to use multiple endpoints in the frontend: `/login` for login page, `/register` for register page, and `/home` for the main page (after logged in). 

> Note that in an old school web app this means sending a request to the server, refresh the page, and then we arrive at our destination. In our app, we are in fact still on the same page. We're just simply utilizing Javascript to perform conditional rendering based on endpoints. And by the way, those endpoints are also completely unrelated to the backend. 
{: .prompt-info}

In order to achieve this we will use React Router. First, install the dependencies:

```
npm install react-router-dom
```

Then make the following changes to `RegisterForm`: 

```tsx
// ...
import { useNavigate } from 'react-router-dom';

const RegisterForm = () => { 
  // ... states
  const navigate = useNavigate();

  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    try {
      // ...

      await registerService.register(registerData);
      navigate("/");
    } catch (err) {
      console.error(err);
    }
  }
  
  return (
    <>
      {/* ... */}
      <button onClick={() => navigate("/")}>Cancel</button>
    </>
  )
}
```
{: file="frontend/src/components/RegisterForm.tsx}
{: .nolineno}

The `useNavigate` hook is used to navigate to a different page. In our logic, after the registration success, we will be redirected to the default page `/` (which is currently where our login page is located). We also added another cancel button at the end for users to return to homepage.

> Task: Do the same thing in `LoginForm`: Create a `Register` button that navigates to `/register`. 
{: .prompt-tip}

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
              <ContactDisplay contacts={contacts} username={user.username} />
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

