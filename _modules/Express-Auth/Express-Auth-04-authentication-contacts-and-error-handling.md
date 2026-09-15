---
title: "Authentication, contacts, and error handling"
parent_post: Express-Auth
module_number: 4
layout: module
media_subpath: /assets/tutorials/express-auth
---

### Authentication

Next, let's implement authentication with JWT (Json Web Token). Watch [this](https://www.youtube.com/watch?v=7Q17ubqLfaM) first in order to understand what is JWT and how does JWT work. 

> In practice, JWT is often implemented with a *refresh-access token model*, in which both the access token - the actual JWT that is used for authentication - have a short-lived lifecycle (typically about 15 minutes), and a refresh token that have a longer lifecycle (about a few days) are utilized. When a user connects to a server, if the access token has expired, their refresh token will be used instead, and if the refresh token is still valid, it will generate another access token, allowing the user to continuously use the service without having to log in repeatedly. 
>
> In this project I will only do the basic access token method. You can do your own research on the refresh token. Practically speaking, in a real project, unless you're working in cybersecurity, you would end up using a library for authentication anyway. 
{: .prompt-info}

After that you can explore the debugger on [jwt.io](https://jwt.io/). Notice it has three parts: header, payload, and signature. To sign tokens, create a `SECRET_KEY` field in your `.env` file and configure it in `config.ts`. Use [jwt-keys.21no.de](https://jwt-keys.21no.de/) to generate a cryptographically strong secret string.

> **NOTE:** In enterprise JWT setups, asymmetric cryptography (public/private key pairs) is commonly used so identity providers sign tokens that services verify independently. In this tutorial, we will use symmetric cryptography (a single shared secret key).
{: .prompt-info }

#### Which endpoints need protection?

First let's think about it for a second: which endpoints need to be protected? In our contact management app, we want to protect endpoints that deal with user-specific data:

- **Public endpoints** (no authentication needed):
  - `POST /api/register` - Anyone can register
  - `POST /api/login` - Anyone can attempt to login
  
- **Protected endpoints** (authentication required):
  - `GET /api/users` - View user information
  - `GET /api/users/:id` - View specific user
  - `GET /api/contacts` - View user's contacts
  - `POST /api/contacts` - Create new contacts

#### Application Flow

Now we will understand how JWT is used. 

First, the user log in with credentials. If the credentials match, JWT is generated. The user can then use the JWT to perform authorized-only operations (e.g. adding a contact to an user's contact list). So we need an endpoint to perform just that.  

First, let's create the login controller that generates JWT tokens:

```typescript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';
import User from '../models/user';
import config from '../config';

export const login = async (req: Request, res: Response, next: NextFunction) => {
  const { username, password } = req.body;

  const user = await User.findOne({ username });

  if (!user || !(await bcrypt.compare(password, String(user!.passwordHash)))) {
    res.status(401).send({ err: "Invalid credentials" });
  }

  const payload = {
    username: user!.username,
    name: user!.name,
    id: user!._id
  };

  const token = jwt.sign(payload, config.SECRET_KEY, { expiresIn: 60*60 });

  res.status(200).send({ token });
} 
```
{: file="backend/src/controllers/loginController.ts" }
{: .nolineno }

> **BUG HUNT:** If you test this controller with invalid credentials in Postman, your server will crash with `Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client`! Why does Express keep running down to line 18 after sending the 401 response? What keyword is missing inside the `if` statement to immediately halt execution?
{: .prompt-danger }

> **QUESTION:** What is the purpose of `{ expiresIn: 60 * 60 }`? Why is token expiration set to 1 hour instead of never expiring? What security risks exist if an access token has no expiration date?
{: .prompt-tip }

### Fixing the Response Flow

In Express, calling `res.send()` or `res.json()` transmits the HTTP response payload to the client, but **it does not automatically exit the JavaScript function**. If execution continues, Express will attempt to send a second response on the same closed connection, triggering `ERR_HTTP_HEADERS_SENT`. Always prefix early error responses with `return`:

```typescript
  if (!user || !(await bcrypt.compare(password, String(user!.passwordHash)))) {
    return void res.status(401).send({ err: "Invalid credentials" });
  }
```
{: file="backend/src/controllers/loginController.ts" }
{: .nolineno }

Next, create the router for user login:

```typescript
import express from 'express';
import { login } from '../controllers/loginController';

const loginRouter = express.Router();

loginRouter.post('/', login);

export default loginRouter;
```
{: file="backend/src/routers/loginRouter.ts" }
{: .nolineno }


#### Handling JWT 

Now that we have a way to generate JWTs. What about storing them and using them for authorization, e.g. to create contacts? In the frontend, the code used to send requests may look like this:

```typescript
const someFunction = async () => {
  const config = {
    headers: { Authorization: `Bearer ${token}` },
  };

  const response = await axios.get(baseUrl, config);
  return response.data;
};
```
{: .nolineno }

Typically, the JWT token will be sent through the `Authorized` header, as we seen above. For now, just use Postman to login first, get the token, and then send the token manually in the `Authorization` header when we want authorized access. We will persist and automatically use the JWT when we develop the frontend. 

> Also, it is the standard to send the `Authorization` header with the format `Bearer {token}` instead of just your token. Just send it like that. 
{: .prompt-info}

Next we will cover how the JWT is used. 

##### 1. Token extraction middleware

When the user is logged in and attempts to perform restricted operations, the JWT will be extracted from the request to validate it. This middleware will extract the token from the `Authorization` header:

```typescript
import { Request, Response, NextFunction } from 'express';
import '@shared/types';

const modifyToken = (req: Request, res: Response, next: NextFunction) => {
  const authorization = req.get("authorization");

  if (authorization && authorization.startsWith("Bearer ")) {
    // delete 'Bearer' and add new field 'token'
    req.token = authorization.substring(7);
  }

  console.log(req.token);
  next();
}

export default modifyToken;
```
{: file="backend/src/middlewares/modifyToken.ts" }
{: .nolineno }

This middleware:

- Checks for the `Authorization` header
- Extracts the token part from `Bearer <token>` format
- Attaches the token to the request object for later use

You will probably notice TypeScript throwing an error: type `Request` does not have field `token`. This is correct - the `Request` type typically does not have that field, we're adding it into the request. So how can we fix this? This is when we use the `types.ts` file. Go to the `shared` folder (outside of `backend`) and add this to `types.ts`: 

```typescript
// extend express.Request
declare global {
  namespace Express {
    interface Request {
      user: {
        id: string;
        username: string;
        name: string;
      }
      token?: string;
    }
  }
}
```
{: file="@shared/types.ts" }
{: .nolineno }

This will extend the `Request` type to also contain the field `user` and `token`. Note that you will have to import `@shared/types.ts` every time you want to extend the `Request`. 

##### 2. JWT Authentication middleware

Now that the JWT is extracted, the next step is to validate it. This middleware validates the JWT token and extracts user information:

```typescript
import jwt from "jsonwebtoken";
import { Request, Response, NextFunction } from "express";
import config from "../config";
import type { JwtPayload } from "@shared/types";


export const jwtAuth = (req: Request, res: Response, next: NextFunction) => {
  const token = req.token;

  try {
    if (!token) {
      return void res.status(401).json({ error: "No token provided" });
    }

    const payload = jwt.verify(token, config.SECRET_KEY) as JwtPayload;
    if (!payload) {
      return void res.status(401).json({ error: "Invalid token" });
    }

    req.user = {
      id: payload.id,
      username: payload.username,
      name: payload.name
    };

    next();
  } catch (error) {
    return void res.status(401).json({ error: "Token invalid or expired" });
  }
};

```
{: file="backend/src/middlewares/jwtAuth.ts" }
{: .nolineno }

This middleware:

- Checks if token exists on the request
- Verifies the token using our secret key
- Extracts user information from the token payload
- Attaches user info to the request object
- Handles token verification errors

You will need to declare a `JwtPayload` type in order to stop TypeScript from throwing errors:

```ts
export interface JwtPayload {
  id: string;
  username: string;
  name: string;
  exp?: number;
  iat?: number;
}
```
{: file="@shared/types.ts" }
{: .nolineno }

Aside from `username`, `name` and `id`, the `iat` and `exp` means issued time and expire time of a JWT in Unix epoch, respectively. These two are pretty standard fields inside a JWT. 

To summarize: the first middleware extracts the JWT and attaches it to the request. The second one validates the token, and if the token is valid, it attaches the username and id of the user to the request. 

> You might be wondering why we attach the username, name and id to the request after decoding the JWT - would that expose the username and id? Well, the thing is that the JWT payload is not securely encrypted in the first place. JWT use base64 encoding, which is easily reversible, and pretty much everybody can decrypt a JWT once they obtain it. The core part of JWT is to prevent tampering - since only a slight alteration of the content will create a completely different JWT. Read more [here](https://softwareengineering.stackexchange.com/questions/280257/json-web-token-why-is-the-payload-public). 
{: .prompt-info}

##### 3. Adding middleware to protected endpoints 

Finally, we configure the middleware in our `app.ts` file. Notice how we apply `jwtAuth` selectively to protect `/api/users` while keeping login and registration endpoints public:

```typescript
// ...
app.use(express.json());

app.use(modifyToken); // add the jwtToken to request

// Public routes (no authentication required)
app.use("/api/login", loginRouter);
app.use("/api/register", registerRouter);

// Apply JWT authentication for protected routes
app.use("/api/users", jwtAuth, userRouter);

export default app;
```
{: file="backend/src/app.ts"}
{: .nolineno}

When a user logs in or registers, there is no JWT present, so the `modifyToken` middleware will do nothing. Once authenticated, subsequent requests contain the `Authorization: Bearer <token>` header, allowing `modifyToken` and `jwtAuth` to validate the token before reaching protected routes.

### Creating Contact Controller & Router

The final part of our backend is setting up contact controllers and routes:

```typescript
import Contact from '../models/contact';
import User from "../models/user";
import { Request, Response, NextFunction } from 'express';
import '@shared/types';

export const getAllContacts = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const contacts = await Contact.find({}).populate("belongsTo", { username: 1, name: 1 });
    res.json(contacts);
  } catch (err) {
    next(err);
  }
};

export const getById = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const contact = await Contact.findById(req.params.id);
    if (!contact) {
      return void res.status(404).send({ error: "Contact not found" });
    }
    res.json(contact);
  } catch (err) {
    next(err);
  }
};

export const addNewContacts = async (req: Request, res: Response, next: NextFunction) => { 
  const { name, number } = req.body;
  const userId = req.user.id;

  if (!userId) {
    return void res.status(401).send({ error: "Invalid token" });
  }
  
  if (!name) {
    return void res.status(400).send({ error: "Name is required" });
  }
  if (!number) {
    return void res.status(400).send({ error: "Number is required" });
  }

  try {
    const user = await User.findById(userId);
    if (!user) {
      return void res.status(400).send({ error: "User not found" });
    }
    
    const contact = new Contact({
      name,
      number,
      belongsTo: userId
    });

    const newContact = await contact.save();
    user.contacts = user.contacts.concat(newContact._id as any);
    await user.save();

    res.status(201).json(newContact);
  } catch (err) {
    next(err);
  }
};

export const deleteById = async (req: Request, res: Response, next: NextFunction) => {
  const userId = req.user.id;

  if (!userId) {
    return void res.status(401).send({ error: "Authentication required" });
  }

  try {
    const user = await User.findById(userId);
    if (!user) {
      return void res.status(400).send({ error: "User not found" });
    }

    await Contact.findByIdAndDelete(req.params.id);
    user.contacts = user.contacts.filter(c => c.toString() !== req.params.id);
    await user.save();

    res.status(204).end();
  } catch (err) {
    next(err);
  }
};
```
{: file="backend/src/controllers/contactController.ts"}
{: .nolineno }

Next, create the router to expose these contact endpoints:

```typescript
import express from 'express';
import {
  getAllContacts,
  getById,
  addNewContacts,
  deleteById
} from '../controllers/contactController';

const contactRouter = express.Router();

contactRouter.get('/', getAllContacts);
contactRouter.get('/:id', getById);
contactRouter.post('/', addNewContacts);
contactRouter.delete('/:id', deleteById);

export default contactRouter;
```
{: file="backend/src/routers/contactRouter.ts"}
{: .nolineno } 

### Error handling

Currently we write our error handling code inside our backend code. However, controllers should only be used to receive requests, not to handle errors. If in the future we have multiple controllers, then we will have to repeat our error handling code across multiple controllers, which not only is not good practice, but also a pain to maintain and update. 

First, for a basic error scenario: if an user try to access the wrong endpoint, we want to return a basic unknown endpoint error. 

```typescript
import { Request, Response } from 'express';

const unknownEndpoint = (req: Request, res: Response) => {
  return void res.status(404).send({ error: "unknown endpoint" });
};

export default unknownEndpoint;
```

Then we want to tackle more specific errors. For example, we only want our database to only contain unique email and username. If we send an invalid register request (duplicate username), the error will look like this (in the console): 

```
MongoServerError: E11000 duplicate key error collection: 2weekproj.users index: username_1 dup key: { username: "root_user" }
[0]     at InsertOneOperation.execute (/home/cuongdang/projects/Imagine/2weekproject/backend/node_modules/mongodb/src/operations/insert.ts:88:13)
[0]     at processTicksAndRejections (node:internal/process/task_queues:95:5)
[0]     at async tryOperation (/home/cuongdang/projects/Imagine/2weekproject/backend/node_modules/mongodb/src/operations/execute_operation.ts:283:14)
[0]     at async executeOperation (/home/cuongdang/projects/Imagine/2weekproject/backend/node_modules/mongodb/src/operations/execute_operation.ts:115:12)
[0]     at async Collection.insertOne (/home/cuongdang/projects/Imagine/2weekproject/backend/node_modules/mongodb/src/collection.ts:285:12) {
[0]   errorLabelSet: Set(0) {},
[0]   errorResponse: {
[0]     index: 0,
[0]     code: 11000,
[0]     errmsg: 'E11000 duplicate key error collection: 2weekproj.users index: username_1 dup key: { username: "root_user" }',
[0]     keyPattern: { username: 1 },
[0]     keyValue: { username: 'root_user' }
[0]   },
[0]   index: 0,
[0]   code: 11000,
[0]   keyPattern: { username: 1 },
[0]   keyValue: { username: 'root_user' }
[0] }
```
{: .nolineno}

We will handle it in our `errorHandler` file: 

```typescript
import { Request, Response, NextFunction } from 'express';

const errorHandler = (error: Error, req: Request, res: Response, next: NextFunction) => {
  console.log("ErrorHandler intercepted: ", error);

  if (error.name === "MongoServerError" && error.message.includes("E11000 duplicate key error")) {
    const duplicate = error.message.includes("email")
      ? "Email"
      : "Username";
    return void res.status(400).json({ error: `${duplicate} has already existed` });
  }

  if (error.name === "CastError") {
    return void res.status(400).send({ error: "Invalid id" });
  }

  if (error.name === "ValidationError") {
    return void res.status(400).json({ error: error.message });
  }

  next(error);
};

export default errorHandler;
```
{: file="backend/src/middlewares/errorHandler.ts"}
{: .nolineno}

Finally, connect your new routes and error-handling middlewares in `app.ts`:

```typescript
// ...
import contactRouter from './routers/contactRouter';
import unknownEndpoint from './middlewares/unknownEndpoint';
import errorHandler from './middlewares/errorHandler';

// Protected contact routes
app.use("/api/contacts", jwtAuth, contactRouter);

// Unknown endpoint & error handler
app.use(unknownEndpoint);
app.use(errorHandler);

export default app;
```
{: file="backend/src/app.ts"}
{: .nolineno}
