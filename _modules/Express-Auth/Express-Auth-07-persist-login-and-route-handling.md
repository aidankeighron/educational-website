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
    // ... 

    try {
      const response = await loginService.login(credentials);
      setJwt(response.token);
      window.localStorage.setItem("JwtAccessToken", response.token);
    } catch (error) {
      console.error("Login failed:", error);
      return false;
    }
  };
```
{: file="frontend/src/hooks/useLogin.ts"}
{: .nolineno}

Then add another `useEffect` to handle the case when the page is refreshed: 

```tsx
export function useLogin() {
  const [jwt, setJwt] = useState<string | null>(null);
  const [contacts, setContacts] = useState<Contact[]>([]);
  const { showNotification } = useNotification();

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

  useEffect(() => {
    //..
  })

  // ...
```
{: file="frontend/src/hooks/useLogin.ts"}
{: .nolineno}

> Note that *refresh* and *rerender* means two different things. *Rerender* is just React updating some parts of the UI, and the local variables stays the same (unless you changed them, of course). For *refreshing*, however, we are making an entirely new request to the server, and all state stored in memory will be lost unless stored elsewhere.
{: .prompt-info}

It works like this: First the user is logged in, then the JWT is stored inside `localStorage` (see `handleLogin`). Then, after we refresh, all the state will be refreshed (so our `jwt` variable would be null), but then `useEffect` is called, and it retrieves the JWT we stored earlier in the browser, call `setJwt`, and then set the token locally inside `contactService` (more on that later). Since we call `setJwt`, the page is rerendered again, but now we have our `jwt` variable set up, so our app should be able to run smoothly. 

For `contactService`, just use 

```ts
let token: string;
export const setToken = (newToken: string) => {
  token = newToken;
};
```
{: file="frontend/src/services/contactService.ts"}
{: .nolineno}

This will persist the token directly inside `contactService` and eliminates any necessity to pass the token from outside. 

> Task: In the above part we did not validate if the JWT extracted from localStorage is valid or not (in particular, its expiry time). Try to validate the JWT after it is retrieved from the browser. If it's not valid, do not continue, but rather delete the token from `localStorage`. You can definitely look up on how to do this - I did the same. To test, go back to backend and change `expiresIn` to a small number and try to refresh the website after.
{: .prompt-tip}

After you're done we can continue working on the logout part. 

> Task: Implement logout function. You should put it inside `useLogin`. The logic is pretty simple: since the contact will not render without `jwt`, you can just clear up all of them. 
{: .prompt-tip}

**Answer (click to unblur):**

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
{: .blur}

`payload` will also be cleared after this since we call `setJwt` and `setContacts`.

### Better routes handling

Currently we have `/register` for the register page. However, we want a better separation: `/login` for login page, `/home` for home page. We also want some logic handling: for example, when user logged in successfully, we want to immediately go to `/home`. To do that we will be upgrading our `App.tsx` file with more routes and logic. 

> Task: Upgrade your `App.tsx` so that it has three routes: `/login`, `/register`, and `/home`. The `/login` endpoint should only contain `LoginForm`, `/home` should only contain `Homepage` (rename `ContactDisplay` into this), and `/register` to only contain the `RegisterForm`. When the user attempts to go to the default endpoint `/`, you should check if the user is logged in or not and then redirect correspondingly (same goes for `/login` and `/home`).  Use `<Navigate>` to redirect. 
{: .prompt-tip}

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

