---
title: Express Authentication App
author: cuong
date: 2025-07-16 12:00:00 +0700
categories: [ExpressAuth]
tags: [TypeScript, Intermediate, Authentication, JWT, Express, MongoDB, React]
description: Learn how to build a secure authentication system using Express, MongoDB, React, and TypeScript
comments: false
pin: true
media_subpath: /assets/tutorials/express-auth
image: /thumb-1920-1358310.png
---

## About the project

Welcome to the Express Authentication tutorial! In this project, you will learn how to build a complete full-stack contact management application with secure user authentication.

![Authentication Demo](/Animation.gif)

**Prerequisites**:

- Basic understanding of how the web works, APIs, sending and receiving requests
- Basic coding skills in JS. React skills are preferable

**What you will learn:**

- How to set up a Node.js backend with Express and TypeScript
- Creating and managing MongoDB database models with Mongoose
- Implementing secure authentication with JWT (JSON Web Tokens)
- Building a React frontend with TypeScript, hooks, and context
- Proper error handling and user feedback with notifications
- Creating protected routes that require authentication

**What you will make:**

By the end of this tutorial, you will have built a fully functional application where users can:

- Register a new account with validation
- Log in securely with JWT authentication
- Create and view contact information
- Experience proper session management with token persistence
- Navigate through a protected application with proper routing

**Further possibilities:**

You can add more to this project:

- Implement refresh/access token
- Add "forget password" field and send email to user to verify
- Add edit/delete functionalities for your contact
- Improve styling
- Use a different database

## Project Overview

Our application consists of two parts:

1. **Backend**: Node.js with Express and TypeScript, connected to MongoDB
2. **Frontend**: React with TypeScript

The application will allow users to:

- Register a new account
- Log in with secure authentication
- View their contacts
- Add new contacts

Let's get started!

## Part 1: Backend setup

We'll begin by setting up our TypeScript configuration and creating the basic structure of our backend. 

### Basic setup

First we create our backend:

```
mkdir backend && cd backend
npm init -y
```

Then install dependencies:
(try to understand their purposes in the scope of the project)

```bash
npm install bcrypt jsonwebtoken mongoose express cors dotenv
npm install --save-dev typescript@^5.9.3 ts-node nodemon tsconfig-paths @types/bcrypt @types/jsonwebtoken @types/express @types/cors @types/node eslint prettier
```

Next, we will create a `tsconfig.json` file. It is used to manage TypeScript in our project. Run

```
npx tsc --init
```

You will see a newly created `tsconfig.json` file. You can try playing around with the settings. Mine look like this for reference:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "..",  
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "paths": {
      "@shared/*": ["../shared/*"]
    },
    "composite": true,
    "sourceMap": true
  },
  "include": ["src/**/*", "../shared/**/*"],  
  "exclude": ["node_modules"]
}
```
{: file="backend/tsconfig.json" }
{: .nolineno }

The `rootDir` property has been changed from `./src` to `..` since we will also need a `frontend` folder later. Also, later on, we will also need a `types.ts` file to put all our custom types inside and use it. Those types will be used in both backend and frontend. You can create a `types.ts` file inside both of them, but I will use a shared folder instead. That's why we need to set up the `paths` and `include` property to include `shared` to our project.

You can create it right now:

```
cd ..           // if you are currently inside backend
mkdir shared
```

Then, create a `types.ts` file under this folder. We will revisit it later.

Next, our `backend` folder should be set up like this:

```
backend/
├── src/
│   ├── config.ts                 // config file for .env
│   ├── app.ts                    // app configuration
│   ├── index.ts                  // server entry point
│   ├── middlewares/              // middlewares
│   │   ├── errorHandler.ts          
│   │   ├── jwtAuth.ts               
│   │   ├── modifyToken.ts
│   │   └── unknownEndpoint.ts
│   ├── models/                   // models
│   │   ├── user.ts
│   │   └── contact.ts 
│   ├── routers/                  // route handlers
│   │   ├── contactRouter.ts
│   │   ├── loginRouter.ts
│   │   ├── registerRouter.ts
│   │   └── userRouter.ts
├── .env                          // environment variables
├── package.json                
└── tsconfig.json      
```

### Creating the models 

Now we will create our models.

A *model* or *schema* is basically how our data is stored inside the database. It is a blueprint to tell us how should the data look like (e.g. which fields should the data have, the restrictions to each field, etc). The two main types of database are *SQL* and *NoSQL*. Basically, a *SQL* database require the data to follow the schema as strictly as possible, and invalid data (which does not follow the schema) will not allowed to be persisted. On the other hand, *NoSQL* database are databases that are more flexible, allowing users to store data that does not have a fixed schema. 

In this guide we will use MongoDB - a NoSQL database. 

Now think about what your models need. In this application, we need two entities: `User` and `Contact`. User will have name, username, email, password, and a list of contact. Contact will have name, number, and belongsTo (which user). 

(Guiding tips: Try to understand how User and Contact work in tandem with each other, and how the contacts are stored in the database.)

First, for our `User`: 

```typescript
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  username: {
    type: String, 
    required: true,
    unique: true, 
    minLength: 3, 
    maxLength: 15, 
    validate: {
      validator: function (v: string) {
        return /^[a-zA-Z][a-zA-Z0-9_]{2,15}$/.test(v);
      }, 
      message: () => "Wrong username format: Begin with letters, alphanumeric only"
                    + "(with underscores), no spaces.",
    }
  },
  name: {
    type: String, 
    required: true,
    minLength: 3
  }, 
  email: {
    type: String, 
    required: true, 
    unique: true,
  },
  passwordHash: String,
  contacts: [ 
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Contact'
    }
  ]
});

userSchema.set("toJSON", {
  transform: (doc, ret: any) => {
    ret.id = ret._id.toString();
    delete ret._id;
    delete ret.__v;

    // DO NOT REVEAL PASSWORD HASH!!!!
    delete ret.passwordHash;
  },
});

export default mongoose.model("User", userSchema);

```
{: file="backend/src/models/user.ts" }
{: .nolineno }

> **QUESTION:** Why do we store a hashed password (`passwordHash`) in the database instead of the raw plaintext password? What security implications arise if an attacker gains access to a database table storing unhashed passwords?
{: .prompt-tip }

The `validate` part above is to validate our name against regex - and if it doesn't match, the database will refuse to save the user to the database. For the `contacts` part, we're using `mongoose.Schema.Types.ObjectId` as type. When we store objects into MongoDb, each object will have its own id. Think of this as an array of id of `Contact`s, so that we can convert them back to actual `Contact` later. 

Also, the "toJSON" part at the end of our file is defining what will the object be like when transformed into JSON. We *absolutely* don't want to reveal an user's `passwordHash`, so we must delete that from the returned result. There are two more fields: `_id` and `__v`, in which we don't need `__v`, and for `_id`, I chose to rename it to just `id`. 

Next, let's create our `Contact` model. A user can own many contacts (a list of contacts), but each contact belongs to one user (`ObjectId` reference):

```typescript
import mongoose from "mongoose";

const contactSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    minLength: 3,
  },
  number: {
    type: String,
    required: true,
    validate: {
      validator: (v: string) => /^\d{2,3}-\d{7,}$/.test(v),
      message: (props: { value: string }) => `${props.value} is not a valid phone number!`,
    },
  },
  belongsTo: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
  },
});

contactSchema.set("toJSON", {
  transform: (document, returnedObject: any) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
  },
});

export default mongoose.model("Contact", contactSchema);
```
{: file="backend/src/models/contact.ts" }
{: .nolineno }

If you were able to understand the `User` file above, this file should be pretty similar. One difference is that the `belongsTo` field is not an array but instead one object - which make sense, because contacts can only be created when a user is logged in, which means that the contact can only belong to one user only. Note that the phone number validator regex `/^\d{2,3}-\d{7,}$/` expects 2–3 digits followed by a hyphen and at least 7 digits (e.g. `09-1234567` or `012-12345678`).

### Creating controllers for our models 

Controllers are functions that will handle the logic of our application. Let's create `userController.ts` in `backend/src/controllers`: 

```typescript
import User from '../models/user';
import Contact from '../models/contact';
import { Request, Response, NextFunction } from 'express';

export const getAll = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const users = await User.find({}).populate("contacts", { name: 1, number: 1 });
    res.json(users);
  } catch (err) {
    next(err);
  }
}

export const getById = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const user = await User.findById(req.params.id).populate("contacts", { name: 1, number: 1 });
    if (!user) {
      return void res.status(404).send({ error: "User not found" });
    }
    res.json(user);
  } catch (err) {
    next(err);
  }
}
```

> **QUESTION:** In asynchronous controller functions (such as `User.findById()`), what happens if a database query rejects or the network connection drops without a `try/catch` block wrapping the `await` call? How does passing errors to `next(err)` protect our Express application from unhandled promise rejections?
{: .prompt-tip }

`Request, Response, NextFunction` are types required for our `req, res, next` arguments. `User.find({})` is used to get all users from the database. 

Remember about the `Contact`s we said earlier that are stored as ObjectId? `populate` here is used to actually display the content of the `Contact` - instead of just as an `ObjectId` (this is the "convert back to `Contact` part we discussed earlier when we were writing model for `User`). First, we have `populate("contacts")` to tell MongoDB to populate the `contacts` field in the `User` object. Then, the `{name: 1, number: 1}` is to include name and number in a `Contact` entity. If you don't want to include name for example, you can leave the field out. 

This file only consists of GET-ing users. For adding users, we will handle that in a different file, `registerController`. But I'll hand that to you. 

> **TASK:** Write the `registerController` to validate username, email, and password, and hash the password before saving using `bcrypt.hash(password, 10)`.
{: .prompt-warning }

**Answer (click to unblur):**

```typescript
import User from '../models/user';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';

export const register = async (req: Request, res: Response, next: NextFunction) => {
  const { username, name, email, password } = req.body;

  if (!username || username.length < 3) {
    return void res.status(400).send({
      error: "Username must be at least 3 characters"
    });
  }

  if (!password || password.length < 8) {
    return void res.status(400).send({
      error: "Password must be at least 8 characters"
    });
  }

  if (!name) {
    return void res.status(400).send({
      error: "Name is required"
    });
  }

  const emailRegex = /^[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?$/;
  if (!email || !emailRegex.test(email)) {
    return void res.status(400).send({
      error: "Invalid email address"
    });
  }

  try {
    const passwordHash = await bcrypt.hash(password, 10);

    const user = new User({
      username,
      name,
      email,
      passwordHash
    });

    const savedUser = await user.save();
    res.status(201).json(savedUser);
  } catch (err) {
    next(err);
  }
};
```
{: file="backend/src/controllers/registerController.ts"}
{: .nolineno }
{: .blur }

> **QUESTION:** Look at the execution order in our register controller: we validate user inputs before calling `bcrypt.hash()`. What potential server performance, reliability, and security issues could arise if we performed password hashing before verifying input formats?
{: .prompt-tip }

### Creating the Express Application

You are not supposed to write everything at once (maybe except for models, those are the first thing you should think about before you do any coding, and should be the first thing you ever set up in a backend application). Now we have written some controllers for our `User` entity, let's test them out by building a test application. 
#### Main setup

First we have to establish our database connection and configure our environment variables.

We will use MongoDB for our database. Setup your database according to this [short video](https://www.youtube.com/shorts/pIHvoXkwmq4). Then, create a .env file in your backend directory:

```bash
MONGODB_URI={your_mongodb_url}
PORT=3001
SECRET_KEY=your_secret_jwt_key
```
{: file="backend/.env" }

> **NOTE:** Never commit your `.env` file to version control! Add it to your `.gitignore` file. Use [jwt-keys.21no.de](https://jwt-keys.21no.de/) to generate a cryptographically strong secret string for `SECRET_KEY`.
{: .prompt-info }

Next, create a configuration file to handle environment variables:

{: file="backend/src/config.ts" }
{: .nolineno }

```typescript
import dotenv from 'dotenv';
dotenv.config();

const PORT = process.env.PORT || 3001;
const MONGODB_URI = process.env.MONGODB_URI || '';
const SECRET_KEY = process.env.SECRET_KEY || '';

export default {
  PORT,
  MONGODB_URI,
  SECRET_KEY
};
```

Next, set up the routers for our endpoints. It makes the function we defined in the controller to be accessible in certain endpoints. For example: 

```typescript
import express from 'express';
import { getAll, getById } from '../controllers/userController';

const userRouter = express.Router();

userRouter.get('/', getAll);
userRouter.get('/:id', getById);

export default userRouter;
```
{: file="backend/src/routers/userRouter.ts" }
{: .nolineno }

Similarly, let's create a router for user registration:

```typescript
import express from 'express';
import { register } from '../controllers/registerController';

const registerRouter = express.Router();

registerRouter.post('/', register);

export default registerRouter;
```
{: file="backend/src/routers/registerRouter.ts" }
{: .nolineno }

Next, create the main Express application file:

```typescript
import express, { Request, Response } from 'express';
import mongoose from 'mongoose';
import config from './config';
import cors from 'cors';

import registerRouter from './routers/registerRouter';
import userRouter from './routers/userRouter';

const app = express();

// Enable CORS for frontend communication
app.use(cors());

// Connect to MongoDB
console.log("connecting to ", config.MONGODB_URI);
mongoose
  .connect(config.MONGODB_URI)
  .then(() => console.log("connected to MongoDB"))
  .catch((error) =>
    console.log("error connecting to MongoDB: ", error.message)
  );

// Middleware for parsing JSON
app.use(express.json());

// Routes
app.use("/api/register", registerRouter);
app.use("/api/users", userRouter);

export default app;
```
{: file="backend/src/app.ts" }
{: .nolineno }

These two lines 

```typescript
app.use("/api/register", registerRouter);
app.use("/api/users", userRouter);
```
{: file="backend/src/app.ts" }
{: .nolineno }

are used to connect your routers. Think of it this way: you connect to the `userRouter` via the top 
domain `/api/users`. Then, to ask it to perform `getById` (refer to router setup part), we send a GET request to `/api/users/{id}`. 

Finally, let's create the entry point for our application:

```typescript
import app from './app';
import config from './config';

app.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT}`);
});
```
{: file="backend/src/index.ts" }
{: .nolineno }

Now our basic backend application should be done. First, configure your `package.json` file:

```json
"scripts": {
    "dev": "nodemon --watch src --exec \"ts-node -r tsconfig-paths/register\" src/index.ts"
  }
```
{: file="backend/package.json"}
{: .nolineno}

The important part here is the `-r tsconfig-paths/register` part. This will enable path mapping support (like `@shared/types`) and without this your `@shared/*` imports won't work. You can look up the rest if you don't understand.

Then start your server:

```bash
cd backend
npm run dev
```

You should see the message "Server running on port 3001" and "connected to MongoDB".

#### Testing

That was a lot of code. In order to check if our controllers are working properly, we have to test our controllers to see it is working as we expected. To do that we will use Postman. Watch this [video](https://www.youtube.com/watch?v=CLG0ha_a0q8) for an introduction to Postman. After that, you should be able to test all of the methods below.
##### 1. Register a New User

**POST** `http://localhost:3001/api/register`

Body (JSON):
```json
{
  "username": "johndoe",
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}
```

Expected Response (201 Created):
```json
{
  "username": "johndoe",
  "name": "John Doe",
  "email": "john@example.com",
  "contacts": [],
  "id": "60f7b3b3b3b3b3b3b3b3b3b3"
}
```

##### 2. Get All Users

**GET** `http://localhost:3001/api/users`

Expected Response (200 OK):
```json
[
  {
    "username": "johndoe",
    "name": "John Doe",
    "email": "john@example.com",
    "contacts": [],
    "id": "60f7b3b3b3b3b3b3b3b3b3b3"
  }
]
```

##### 3. Get User by ID

**GET** `http://localhost:3001/api/users/{id}`

(Replace the ID with the actual ID from your database)

Expected Response (200 OK):
```json
{
  "username": "johndoe",
  "name": "John Doe",
  "email": "john@example.com",
  "contacts": [],
  "id": "60f7b3b3b3b3b3b3b3b3b3b3"
}
```

> **Note**: Notice that the `passwordHash` field is not included in the response. This is because of our `toJSON` transformation in the User model that removes sensitive data.
{: .prompt-info }

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

## Part 2: Frontend setup 

Now that our backend is set up, let's move on to creating the frontend of our application. We will use Vite as frontend template.

### Basic setup 

First, outside the backend folder, run 

```
npm create vite@latest
```
{: .nolineno}

Then, enter your project name, choose React and TypeScript. After that, you can run 

```bash
cd frontend 
npm install 
npm install axios jwt-decode react-router-dom
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

  const handleLoginBackend = async (credentials: Credentials) => {
    console.log("Submitting credentials:", credentials);
  };

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

First the form have two states: `username` and `password`, contained within a form, and set up to change as the user edit the text fields. Then, the submit button is named `login` and linked to `handleLogin`. `event.preventDefault()` is to prevent the page from reloading. Notice that `handleLogin` is currently calling a placeholder `handleLoginBackend`. 

> **TASK:** Create function `handleLoginBackend` that will send the credentials (username and password) to the backend `/api/login`. If valid, persist the returned JWT within React state. (You can use [Axios](https://github.com/axios/axios)).
{: .prompt-warning }

**Answer (click to unblur):**

```tsx
function App() {
  // ...
  const [jwt, setJwt] = useState<string | null>(null);

  // ...

  const handleLoginBackend = async (credentials: Credentials) => {
    const baseUrl = "/api/login";

    try {
      const response = await axios.post(baseUrl, credentials);
      const token = response.data.token;

      setJwt(token);
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

To mitigate this, configure CORS on the backend or add a proxy in your `vite.config.ts` (pointing to your backend running on port 3001):

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

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
        target: "http://localhost:3001", 
        changeOrigin: true,
      },
    }
  }
});
```
{: file="frontend/vite.config.ts" }
{: .nolineno }

Also, ensure your `frontend/tsconfig.json` includes the `@shared/*` path mapping so TypeScript resolves the shared types:

```json
"paths": {
  "@shared/*": ["../shared/*"]
}
```
{: file="frontend/tsconfig.json" }
{: .nolineno }

With this, you can communicate directly with the server. If you want to test your frontend code in real-time, first run your backend, then run your frontend, and your requests to `/api` will be proxied automatically. 

#### Displaying contacts & JWT Decoding

Next, after the user logs in, we display their contacts. To ensure users only see their own contacts, we use [jwt-decode](https://www.npmjs.com/package/jwt-decode) to read the user's username directly from the client-side JWT payload:

```tsx
function App() {
  const [jwt, setJwt] = useState<string | null>(null);
  const [contacts, setContacts] = useState([]);

  const payload = jwt !== null 
    ? jwtDecode<JwtPayload>(jwt)
    : null;

  useEffect(() => {
    if (payload !== null && jwt) {
      const contactUrl = "/api/contacts";
      const token = jwt;

      const config = {
        headers: { Authorization: `Bearer ${token}` },
      };

      axios.get(contactUrl, config).then((response) => {
        setContacts(response.data.filter(
          contact => contact.belongsTo.username === payload.username
        ));
      });
    }
  }, [payload, jwt]);

  return (
    <>
      {/* login form */}
      {jwt !== null && (
        <div>
          <h2>Your Contacts</h2>
          {contacts.map((contact) => (
            <div key={contact.id}>
              {contact.name} {contact.number}
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
{: .nolineno }

The approach works like this: When jwt is `null`, nothing happens. But then if `jwt` is not null, then the entire function runs again, and then `payload` will run first before `useEffect` runs. After that, when `useEffect` runs, it will get the token, send it, and filter the response by payload data. 

> **NOTE:** You can also update the backend `/api/contacts` endpoint to return only contacts belonging to the authenticated user (`req.user.id`). This avoids sending all contacts across the network and filtering them in React.
{: .prompt-info }

Also notice `JwtPayload`. It is yet another defined custom types in `types.ts`. We will cover it right in the next part. 

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

(Guiding tips: Try simulating the loading state that you would normally see on websites (like a "Saving Profile..." with a loading wheel spinner)

### Add styling 

Currently our app have no styling at all. You can improve it by adding more CSS/Tailwind/MUI/ etc. in order to improve the appearance of the app. 

There is no right answer to this. But for example, mine look like this:

```css
:root {
  --primary-color: #2563eb;
  --error-color: #ef4444;
  --border-radius: 8px;
  --shadow-light: 0 1px 3px rgba(0, 0, 0, 0.1);
  --transition: all 0.2s ease;
}

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
Refer back to the gif at the beginning of the guide to see the full design.

## Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (9 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Response Header Crash (`ERR_HTTP_HEADERS_SENT`) | If you test this controller with invalid credentials, the server crashes with `Cannot set headers after they are sent`. Why does Express continue running after calling `res.send()`, and how does `return` fix it? |
| 2 | Question | Password Hashing Security | Why do we store a hashed password (`passwordHash`) in the database instead of the raw plaintext password? What security implications arise if an attacker accesses unhashed passwords? |
| 3 | Question | Asynchronous Controller Error Handling | In asynchronous controller functions (like `User.findById()`), what happens if a query fails without a `try/catch` block? How does passing errors to `next(err)` protect the application? |
| 4 | Question | Input Validation Timing & Cryptographic Cost | Look at the execution order in our register controller: we validate user inputs before calling `bcrypt.hash()`. What potential server performance, reliability, and security issues could arise if we performed password hashing before verifying input formats? |
| 5 | Question | JWT Expiration Window | What is the purpose of `{ expiresIn: 60 * 60 }`? Why is token expiration set to 1 hour instead of never expiring? What risks exist if an access token has no expiration date? |
| 6 | Question | Token Storage Security (`localStorage` vs. `HttpOnly`) | For educational purposes, storing JWTs in `localStorage` is convenient. In production applications, what security trade-offs (such as XSS vs. CSRF) differentiate storing tokens in `localStorage` versus `HttpOnly` cookies? |
| 7 | Task | User Registration Controller | Write the `registerController` to validate username, email, and password, and hash the password before saving using `bcrypt.hash(password, 10)`. |
| 8 | Task | Frontend Login & State Management | Create function `handleLoginBackend` that will send the credentials (username and password) to the backend `/api/login`. If valid, persist the returned JWT within React state. |
| 9 | Task | Client Route Protection Guards | Upgrade `App.tsx` so that it has three routes: `/login`, `/register`, and `/home`. If a logged-in user accesses `/` or `/login`, redirect them to `/home` using `<Navigate replace />`. If an unauthenticated user accesses `/home`, redirect them to `/login`. |

</details>

## Conclusion

Congratulations! You've built a complete full-stack contact management application with TypeScript, React, and Express. This application demonstrates several important concepts:

- **Authentication**: Secure login and registration with JWT tokens
- **Data Management**: Creating and retrieving contacts from a MongoDB database
- **Type Safety**: Using TypeScript for type checking across the stack
- **User Experience**: Notifications, form validation, and proper navigation
- **Code Organization**: Clean separation of concerns with components, hooks, and services

Happy coding!
