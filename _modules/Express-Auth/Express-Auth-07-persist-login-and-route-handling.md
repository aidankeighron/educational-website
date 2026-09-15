---
title: "Persist login and route handling"
parent_post: Express-Auth
module_number: 7
layout: module
media_subpath: /assets/tutorials/express-auth
---

### Persist login between refreshes

If you refresh the page (hard-refresh by F5) in the current state, you will be immediately logged out. The reason is that we have not stored our token in the browser to use, so it can only persist until the page is reloaded. To solve that we will use [`window.localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) in order to store our token on our browser. 

First, right after we're logged in, we're going to store the token inside a field named `JwtAccessToken` directly inside the browser: 

```tsx
export function useLogin() {
  // ...

  const handleLogin = async (username: string, password: string) => {
    try {
      const response = await loginService.login({ username, password });
      setJwt(response.token);
      window.localStorage.setItem("JwtAccessToken", response.token);
      return true;
    } catch (error) {
      console.error("Login failed:", error);
      return false;
    }
  };
```
{: file="frontend/src/hooks/useLogin.ts"}
{: .nolineno}

Then add an effect to restore the token and user session when the page is refreshed: 

```tsx
export function useLogin() {
  const [jwt, setJwt] = useState<string | null>(null);
  const [contacts, setContacts] = useState<Contact[]>([]);

  const payload = jwt !== null 
    ? jwtDecode<JwtPayload>(jwt)
    : null;

  useEffect(() => {
    const jwtAccessToken = window.localStorage.getItem("JwtAccessToken");

    if (jwtAccessToken) {
      setJwt(jwtAccessToken);
      contactService.setToken(jwtAccessToken);
    }
  }, []);

  // ...
```
{: file="frontend/src/hooks/useLogin.ts"}
{: .nolineno}

> Note that *refresh* and *rerender* means two different things. *Rerender* is just React updating some parts of the UI, and the local variables stays the same (unless you changed them, of course). For *refreshing*, however, we are making an entirely new request to the server, and all state stored in memory will be lost unless stored elsewhere.
{: .prompt-info}

It works like this: First the user is logged in, then the JWT is stored inside `localStorage` (see `handleLogin`). Then, after we refresh, all the state will be refreshed (so our `jwt` variable would be null), but then `useEffect` is called, and it retrieves the JWT we stored earlier in the browser, call `setJwt`, and then set the token locally inside `contactService` (more on that later). Since we call `setJwt`, the page is rerendered again, but now we have our `jwt` variable set up, so our app should be able to run smoothly. 

For logging out, we implement `handleLogout` inside `useLogin` to clear `localStorage`, reset state variables, and clear the token from `contactService`:

```tsx
  const handleLogout = () => {
    window.localStorage.removeItem("JwtAccessToken");
    setJwt(null);
    setContacts([]);
    contactService.setToken("");
  };
```
{: file="frontend/src/hooks/useLogin.ts"}
{: .nolineno}

`payload` will also be cleared after this since we call `setJwt` and `setContacts`.

> **QUESTION:** For educational purposes, storing JWTs in `localStorage` is convenient. In production applications, what security trade-offs (such as XSS vs. CSRF vulnerabilities) differentiate storing authentication tokens in `localStorage` versus `HttpOnly` cookies?
{: .prompt-tip }

### Better routes handling

Currently we have `/register` for the register page. However, we want a better separation: `/login` for login page, `/home` for home page. We also want some logic handling: for example, when user logged in successfully, we want to immediately go to `/home`. To do that we will be upgrading our `App.tsx` file with more routes and logic. 

> **TASK:** Upgrade `App.tsx` so that it has three routes: `/login`, `/register`, and `/home`. If a logged-in user accesses `/` or `/login`, redirect them to `/home` using `<Navigate replace />`. If an unauthenticated user accesses `/home`, redirect them to `/login`.
{: .prompt-warning }

**Hint 1 (login endpoint)**

```tsx
function App() {
  // useLogin()

  return (
    <Router>
      <Routes>
        {/* login route: go to /home if logged in*/}
        <Route
          path="/login"
          element={
            payload ? (
              <Navigate to="/home" replace />
            ) : (
              <>
                <h1>Login</h1>
                <LoginForm handleLogin={handleLogin} />
              </>
            )
          }
        />

	// ...
  );
}
```
{: file="frontend/src/App.tsx"}
{: .nolineno}
{: .blur}

**Answer (click to unblur):**
```tsx
	// ... login endpoint in above hint
	{/* home route: stay if logged in, else redirect to /login */}
        <Route
          path="/home"
          element={
            payload ? (
              <Homepage
                contacts={contacts}
                username={payload.username}
                handleLogout={handleLogout}
              />
            ) : (
              <Navigate to="/login" replace />
            )
          }
        />

        {/* register: always open, only accessible via /login */}
        <Route path="/register" element={<RegisterForm />} />

        {/* default route "/": redirect */}
        <Route
          path="/"
          element={
            payload ? (
              <Navigate to="/home" replace />
            ) : (
              <Navigate to="/login" replace />
            )
          }
        />

        {/* 404 not found: create your own NotFoundPage */}
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </Router>
```
{: file="frontend/src/App.tsx"}
{: .nolineno}
{: .blur}

Let's create our `Homepage` and `NotFoundPage` components:

```tsx
import { useState, useEffect, type FormEvent } from "react";
import type { Contact } from "@shared/types";
import * as contactService from "../services/contactService";

interface HomepageProps {
  contacts: Contact[];
  username: string;
  handleLogout: () => void;
}

export const Homepage = ({ contacts, username, handleLogout }: HomepageProps) => {
  const [name, setName] = useState("");
  const [number, setNumber] = useState("");
  const [contactList, setContactList] = useState<Contact[]>(contacts);

  useEffect(() => {
    setContactList(contacts);
  }, [contacts]);

  const handleAddContact = async (e: FormEvent) => {
    e.preventDefault();
    if (!name || !number) return;

    try {
      const added = await contactService.create({ name, number });
      setContactList(contactList.concat(added));
      setName('');
      setNumber('');
    } catch (err) {
      console.error('Failed to add contact', err);
    }
  };

  return (
    <div className="homepage-container">
      <div className="homepage-header">
        <span className="homepage-user">Logged in as {username}</span>
        <button className="homepage-logout" onClick={handleLogout}>Logout</button>
      </div>

      <h1 className="homepage-title">Your Contacts</h1>
      <div className="contacts-list">
        {contactList.map((contact) => (
          <div key={contact.id} className="contact-card">
            <span className="contact-name">{contact.name}</span>
            <span className="contact-number">{contact.number}</span>
          </div>
        ))}
      </div>

      <form className="add-contact-form" onSubmit={handleAddContact}>
        <h3>Add New Contact</h3>
        <div className="form-group">
          <label>Name</label>
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
          />
        </div>
        <div className="form-group">
          <label>Number</label>
          <input
            type="text"
            value={number}
            onChange={(e) => setNumber(e.target.value)}
          />
        </div>
        <button type="submit">Add Contact</button>
      </form>
    </div>
  );
};

export default Homepage;
```
{: file="frontend/src/components/Homepage.tsx"}
{: .nolineno}

```tsx
import { Link } from 'react-router-dom';

export const NotFoundPage = () => {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <p>The page you are looking for does not exist.</p>
      <Link to="/">Go Home</Link>
    </div>
  );
};

export default NotFoundPage;
```
{: file="frontend/src/components/NotFoundPage.tsx"}
{: .nolineno}

The `replace` part in `<Navigate>` is for the new endpoint to replace the old endpoint in your browser history. Without `replace`, you could click the backwards button in your browser and you would go back to `/login` when you are at `/home`, while we don't really want that. 

#### NotFoundPage on backend

Looking back at our `unknownEndpoint`: 

```ts
const unknownEndpoint = (req: Request, res: Response) => {
  return void res.status(404).send({ error: "unknown endpoint" });
};
```
{: file="backend/src/middlewares/unknownEndpoint.ts"}
{: .nolineno}

We have a conflict between the frontend and backend: When we go to an unknown endpoint, for example `/abcde`, the `unknownEndpoint` middleware in backend will override the frontend, which means that our `NotFoundPage` will not be displayed. 

One way to fix it is to separate the API calls with frontend calls:

```tsx
import path from "path";

const unknownEndpoint = (req: Request, res: Response) => {
  if (req.path.startsWith("/api/")) {
    return void res.status(404).send({ error: "unknown endpoint" });
  }

  return void res.sendFile(path.resolve(__dirname, "../../dist/index.html"));
};
```
{: file="backend/src/middlewares/unknownEndpoint.ts"}
{: .nolineno}

But what the heck is the last line? 
#### Frontend production build

So far we've been developing our frontend in *development* mode. However when we actually ship the product, we should use the *production* build, as it is more optimized for deployment. 

To build your frontend for production, run:

```bash
cd frontend
npm run build
```

This creates a `dist` folder containing optimized static files (HTML, CSS, JavaScript) that can be served by your Express server. 

Now, to use the frontend production build with the backend, one option is to copy the `dist` folder directly from the frontend to the backend. You can automate this with a script in the backend `package.json`: 

```json
"scripts": {
    "build:fe": "rm -rf dist && cd ../frontend && npm run build && cp -r dist ../backend"
  }
```
{: file="backend/package.json"}
{: .nolineno}

> **NOTE:** On Windows PowerShell or Command Prompt, run the build command manually (`cd ../frontend; npm run build; Copy-Item -Recurse dist ..\backend`) or use WSL/Git Bash to run the chained shell commands.
{: .prompt-info }

This will delete the current `dist` folder (if present), go to frontend and build, then copy the entire folder back to the backend folder. (hence the path `"../../dist/index.html"` in `unknownEndpoint` above  - it tries to load `dist/index.html`).

Next, go back to backend `app.ts`, and add one line:

```ts
app.use(express.static("dist")); // add it right here
app.use(express.json());

app.use(modifyToken); 

// ...

```
{: file="backend/src/app.ts"}
{: .nolineno}

This will allow the backend to serve the static`dist` folder.
