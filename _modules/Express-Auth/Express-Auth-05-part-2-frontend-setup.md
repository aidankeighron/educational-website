---
title: "Part 2: Frontend setup"
parent_post: Express-Auth
module_number: 5
layout: module
media_subpath: /assets/tutorials/express-auth
---

## Part 2: Frontend setup 

Now that our backend is set up, let's move on to creating the frontend of our application. We will use Vite as frontend template.

### Basic setup 

First, outside the backend folder, run 

```
npm create vite@latest
```
{: .nolineno}

Then, enter your project name, choose React and TypeScript. After that, you can run 

```
cd frontend 
npm install 
npm run dev
```
{: .nolineno}

#### Login page

Now your app should run. However, we won't need the app template file. You can delete the css import in `main.tsx`, and edit the `App.tsx` file into 

```tsx
import { useState, useEffect } from "react";
import axios from "axios";

function App() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  interface Credentials {
    username: string;
    password: string;
  }

  const handleLogin = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    const credentials: Credentials = {
      username,
      password,
    };

    await handleLoginBackend(credentials);
  };

  return (
    <>
      <h1>login</h1>
      <form onSubmit={handleLogin}>
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
    </>
  );
}

export default App;
```
{: file="frontend/src/App.tsx"}
{: .nolineno}

After this we can have a simple login form that look like this (the `register` button is not present here, but overall the login should look like this):

![](Pasted image 20250708233009.png)

First the form have two states: `username` and `password`, contained within a form, and set up to change as the user edit the text fields. Then, the submit button is named `login` and linked to `handleLogin`. `event.preventDefault()` is to prevent the page from reloading. Notice that `handleLogin` is currently missing `handleLoginBackend`. 

> Task: Create function `handleLoginBackend` that will send the request (username and password) from the frontend from the backend we set up above. If the credentials is valid, the backend will return the JWT and you should persist it within a state. 
> You will need to look up how to send request from frontend. I used [Axios](https://github.com/axios/axios). 
{: .prompt-tip}

**Answer (click to unblur):**

```tsx
function App() {
  // ...
  const [jwt, setJwt] = useState(null);

  // ...

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

  // ...
}

export default App;
```
{: file="frontend/src/App.tsx"}
{: .nolineno}
{: .blur}

#### Backend proxy

Before we move on, if you just send requests from frontend to backend like right now, chances are it will not work. If you open the console, it would be filled with errors. This is because of something called the same origin policy. To explain shortly, it's a security feature: your frontend is running default on port 5173 (Vite default), and backend on port 3000, so they cannot communicate since they're not on the same origin. 

To mitigate this, you can install `cors` directly on backend and enable it, or add this to your `vites.config.ts` (assuming your backend is running on port 3000):

```ts
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@shared': path.resolve(__dirname, '../shared')
    }
  },
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:3000", 
        changeOrigin: true,
      },
    }
  }
})
```

With this, you can communicate directly with the server. If you want to test your frontend code in real-time, first run your backend, then run your frontend, then test directly on your frontend port (in this case 5173) and your requests will go through. 

Also, the `alias` part is to make sure your files recognizes the `@shared/types.ts` syntax. 

#### Displaying contacts

Then, after the user is logged in, we should display the contacts. 

> Task: Implement displaying the list of contacts after the user is logged in. To do it, you can check if the JWT is not null. 
{: .prompt-tip}

**Answer (click to unblur):**

```tsx
function App() {
  // ...
  const [contacts, setContacts] = useState([]);

  useEffect(() => {
    if (jwt !== null) {
      console.log(jwt);
      const contactUrl = "/api/contacts";
      const token = jwt.token;

      const config = {
        headers: { Authorization: `Bearer ${token}` },
      };

      axios.get(contactUrl, config).then((response) => setContacts(response.data));
    }
  }, [user]); // Add dependency array to prevent infinite re-renders

  return (
    <>
      // ... 
      {jwt !== null && (
        <div>
          <h2>Your Contacts</h2>
          {contacts.map((contact) => (
            <div>
              {contact!.name} {contact!.number}
            </div>
          ))}
        </div>
      )}
    </>
  );
}

export default App;
```
{: file="frontend/src/App.tsx"}
{: .nolineno}
{: .blur}

If you didn't know `useEffect` already you should look it up *immediately*. Also, here we add another variable `config` after `contactUrl` in order to send the JWT with the request.

However, If you test this code right now, you'll notice a problem: **all contacts in the database are being displayed**, regardless of which user is logged in. This is a security issue! Each user should only see their own contacts.

> Task: Fix so that only contacts belong to the authenticated user are displayed. To do that you'll first need to decode your JWT in order to get the username. Use [jwt-decode](https://www.npmjs.com/package/jwt-decode).
{: .prompt-tip}

**Answer (click to unblur):**

```tsx
function App() {
	const [jwt, setJwt] = useState(null);
	const [contacts, setContacts] = useState([]);

	const payload = jwt !== null 
    ? jwtDecode<JwtPayload>(jwt)
    : null;

	useEffect(() => {
		if (payload !== null) {
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

	// ...
}
```
{: file="frontend/src/App.tsx"}
{: .nolineno }
{: .blur }

The approach works like this: When jwt is `null`, nothing happens. But then if `jwt` is not null, then the entire function runs again, and then `payload` will run first before `useEffect` runs. After that, when `useEffect` runs, it will get the token, send it, and filter the response by payload data. 

> ...or maybe you can change it in the backend so that the resposne already contains the filtered data? :) That approach is better but I'll let you figure out that yourself. 
{: .prompt-tip}

Also notice `JwtPayload`. It is yet another defined custom types in `types.ts`. We will cover it right in the next part. 

