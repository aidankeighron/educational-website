---
title: Express Authentication App
author: Cuong Dang
date: 2025-06-15 12:00:00 +0700
categories: [TypeScript, ExpressAuth]
tags: [TypeScript, Intermediate, Authentication, JWT, Express, MongoDB, React]
description: Learn how to build a secure authentication system using Express, MongoDB, React, and TypeScript
comments: false
pin: true
media_subpath: /assets/tutorials/express-auth
image: /auth-header.png
---


## About the project

Welcome to the Express Authentication tutorial! In this project, you will learn how to build a complete full-stack contact management application with secure user authentication.

![Authentication Demo](/auth-demo.gif)

**What you will learn:**

- How to set up a Node.js backend with Express and TypeScript
- Creating and managing MongoDB database models with Mongoose
- Implementing secure authentication with JWT (JSON Web Tokens)
- Building a React frontend with TypeScript, hooks, and context
- Proper error handling and user feedback with notifications
- Creating protected routes that require authentication
- Type safety across the entire stack with TypeScript

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

## Table of Contents

- [Project Overview](#project-overview)
- [Part 1: Setting Up the Backend](#part-1-setting-up-the-backend)
- [Part 2: Building the Frontend Foundation](#part-2-building-the-frontend-foundation)
- [Part 3: Adding Authentication](#part-3-adding-authentication)
- [Part 4: Contact Management Features](#part-4-contact-management-features)
- [Part 5: Final Touches and Improvements](#part-5-final-touches-and-improvements)
- [Conclusion](#conclusion)

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

```
npm install bcrypt jsonwebtoken mongoose express cors dotenv
npm install --save-dev typescript ts-node nodemon @types/bcrypt @types/jsonwebtoken @types/express @types/cors @types/node eslint prettier
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
    "baseUrl": ".",
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

A *model* or *schema* is basically how our data is stored inside the database. It is a blueprint to tell us how should the data look like (e.g. which fields should the data have, the restrictions to each field, etc). The two main types of database are *SQL* and *NoSQL*. Basically, a *SQL* database require the data to follow the schema as strictly as possible, and invalid data (which does not follow the schema) will not allowed to be persisted. On the other hand, *NoSQL* database are database that are more flexible, allowing users to store data that does not have a fixed schema. 

In this guide we will use MongoDB - a NoSQL database. 

Now think about what your models need. In this application, we need two entities: `User` and `Contact`. User will have name, username, email, password, and a list of contact. Contact will have name, number, and belongsTo (which user). 

First, for our `User`: 

```typescript
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  username: {
    type: String, 
    required: true,
    unique: true, 
    minLength: 3, 
    maxLength: 10, 
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
  transform: (doc, ret) => {
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

The `validate` part above is to validate our name against regex - and if it doesn't match, the database will refuse to save the user to the database. For the `contacts` part, we're using `mongoose.Schema.Types.ObjectId` as type. When we store objects into MongoDb, each object will have its own id. Think of this as an array of id of `Contact`s, so that we can convert them back to actual `Contact` later. 

Also, the "toJSON" part at the end of our file is defining what will the object be like when transformed into JSON. We *absolutely* don't want to reveal an user's `passwordHash`, so we must delete that from the returned result. There are two more fields: `_id` and `__v`, in which we don't need `__v`, and for `_id`, I chose to rename it to just `id`. 

Next, for our `Contact`: 

> Task: Create our `Contact` model inside `backend/src/models`. It should have name, number, and a belongsTo field that reference back to an user. When referring to other objects, use its ObjectId. You should also add some validation of your choice - looking up some public regex can be a good idea.
{: .prompt-tip}

**Answer (click to unblur):**

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
    minLength: 8,
    maxLength: 11,
    validate: {
      validator: function (v: string) {
        return /^\d{2,3}-(\d+)$/.test(v);
      },
      message: () => "Wrong format (123-1234567).",
    },
  },

  belongsTo: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
  },
});

contactSchema.set("toJSON", {
  transform: (doc, ret) => {
    ret.id = ret._id.toString();
    delete ret._id;
    delete ret.__v;
  },
});

export default mongoose.model("Contact", contactSchema);
```
{: file="backend/src/models/contact.ts" }
{: .nolineno}
{: .blur }

If you were able to understand the `User` file above, this file should be pretty similar. One difference is that the `belongsTo` field is not an array but instead one object - which make sense, because contacts can only be created when a user is logged in, which means that the contact can only belong to one user only.

### Creating controllers 

After we have defined our models, we can move on to write controllers. 

A *controller* can generally be understood as your request handler. For example, if you create a GET request to `localhost:3001/api/users`, the controllers will handle that request, do various backend operations, such as talking/querying to database or getting the data, and then send back to you the response from the server.  For most applications, with each model, you should write all the [CRUD](https://www.codecademy.com/article/what-is-crud) controllers for each object. In RESTful applications, that translates to four types of request: GET, POST, DELETE, PUT/PATCH.

For the scope of this app, I'm going to simplify things a bit. For `User`, we just want a `POST` request (registering new users) and a GET request (for login). For `Contact`, we want a GET, POST, and DELETE. 

For `User`: 

{: file="backend/src/controllers/userController.ts"}
{: .nolineno }

```typescript 
import User from "../models/user";
import { Request, Response, NextFunction } from "express";

export const getAllUsers = async (req: Request, res: Response, next: NextFunction) => {
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

`Request, Response, NextFunction` are types required for our `req, res, next` arguments. `User.find({})` is used to get all users from the database. 

Remember about the `Contact`s we said earlier that are stored as ObjectId? `populate` here is used to actually display the content of the `Contact` - instead of just as an `ObjectId` (this is the "convert back to `Contact` part we discussed earlier when we were writing model for `User`). First, we have `populate("contacts")` to tell MongoDB to populate the `contacts` field in the `User` object. Then, the `{name: 1, number: 1}` is to include name and number in a `Contact` entity. If you don't want to include name for example, you can leave the field out. 

This file only consists of GET-ing users. For adding users, we will handle that in a different file, `registerController`. But I'll hand that to you. 

> Task: Write a controller that supports adding users. The request contains username, name, email, and password. You should try to validate your username, email, and password (just simple `if`s are sufficient). For email validation, you might want to see [this](https://uibakery.io/regex-library/email). And you will also want to hash our password before saving to our database, using [bcrypt](https://nordvpn.com/blog/what-is-bcrypt/).  Basically, just use this in your code
> ```typescript
> const passwordHash = await bcrypt.hash(password, 10); 
> ```
> and store the password hash along with the other details into your database. You would also want to look up how to store an object to the database, if you don't already know that. 
{: .prompt-tip}

**Answer (click to unblur):**

```typescript
import User from '../models/user';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';

export const register = async (req: Request, res: Response, next: NextFunction) => {
  const {username, name, email, password} = req.body;

  if (username.length <= 6)
    return void res.status(400).send({
      error: "Username must be at least 6 characters"
    });

  if (password.length <= 8) {
    return void res.status(400).send({
      error: "Password must be at least 8 characters"
    })
  }

  const emailRegex = /^[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?$/;
  if (!emailRegex.test(email)) {
    return void res.status(400).send({
      error: "Invalid email address"
    })
  };

  const passwordHash = await bcrypt.hash(password, 10);

  const user = new User({
    username, 
    name, 
    email, 
    passwordHash
  });

  try {
    const savedUser = await user.save();
    res.status(201).json(savedUser);
  } catch (err) {
    next(err);
  }
}
```
{: file="backend/src/controllers/registerController.ts"}
{: .nolineno }
{: .blur }

### Creating the Express Application

You are not supposed to write everything at once (maybe except for models, those are the first thing you should think about before you do any coding, and should be the first thing you ever set up in a backend application). Now we have written some controllers for our `User` entity, let's test them out by building a test application. 
#### Main setup

First we have to establish our database connection and configure our environment variables.

We will use MongoDB for our database. Setup your database according to this [short video](https://www.youtube.com/shorts/pIHvoXkwmq4). Then, create a .env file in your backend directory:

```bash
MONGODB_URI={your_mongodb_url}
PORT=3001
```
{: file="backend/.env" }

> **Important**: Never commit your `.env` file to version control! Add it to your `.gitignore` file.
{: .prompt-warning }

Next, create a configuration file to handle environment variables:

{: file="backend/src/config.ts" }
{: .nolineno }

```typescript
import dotenv from 'dotenv';
dotenv.config();

const PORT = process.env.PORT || 3001;
const MONGODB_URI = process.env.MONGODB_URI || '';

export default {
  PORT,
  MONGODB_URI,
  SECRET_KEY
};
```

Next, set up the routers for our endpoints. It makes the function we defined in the controller to be accessible in certain endpoints. For example: 

```typescript
import express from 'express';
import { getAllUsers, getById } from '../controllers/userController';

const userRouter = express.Router();

userRouter.get('/', getAllUsers);
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

> Technically speaking you can do 
> ```typescript
> userRouter.get("/", async(...) => {})
> ```
> directly inside your controller file. This boils down to personal preference for the most part. 
{: .prompt-info}

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

Now our basic backend application should be done. First, start your server:

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

After that you can play around on [jwt.io](https://jwt.io/). Notice it has three parts: headers, payload, and signature. The signature part is done using a private key. However, we don't have a private key yet.

>Task: Create a SECRET_KEY field in your `.env` file and also set it up in the `config` file. It should not just be a random string. Use [this](https://jwt-keys.21no.de/) to generate a secure key.  
{: .prompt-tip}

> You might notice that a typical JWT application involves both public key and private key (assymmetric cryptography). In the scope of this project, however, we will only use a simple shared secret key (symmetric cryptography). 
{: .prompt-info}

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

  if (!user || !(await bcrypt.compare(password, String(user!.passwordHash))))
    return void res.status(401).send({ err: "Invalid credentials "});

  const payload = {
    username: user.username,
    id: user._id
  };

  const token = jwt.sign(payload, config.SECRET_KEY, { expiresIn: 60*60 });

  res.status(200).send({ token, username: username, name: user.name});
}
```
{: file="backend/src/controllers/loginController.ts" }
{: .nolineno }

The login process works by first finding an user with the same username as provided by the request. Then, it hashes the password received from the request and compare it against the one queried from the database. If the username is not valid or the password is incorrect, it sends back a `401 unauthorized`. Otherwise, a JWT is signed along with the payload and returned.

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

This part will cover how the JWT is used. 

##### 1. Token extraction middleware

When the user is logged in and attempts to perform restricted operations, the JWT will be extracted from the request to validate it. This middleware will extracts the token from the `Authorization` header:

```typescript
import { Request, Response, NextFunction } from 'express';
import '@shared/types';

const modifyToken = (req: Request, res: Response, next: NextFunction) => {
  const authorization = req.get("authorization");
  if (authorization != null) console.log("modifyToken " + authorization);

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
      user?: {
        id: string;
        username: string;
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
import "@shared/types";

interface JwtPayload {
  id: string;
  username: string;
  name: string;
  iat: number;
  exp: number;
}

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

You will need to declare a `JwtPayload` type in order to stop TypeScript from throwing errors. Aside from `username` and `id`, the `iat` and `exp` means issued time and expire time of a JWT in Unix epoch, respectively. These two are pretty standard fields inside a JWT. 

To summarize: the first middleware extracts the JWT and attaches it to the request. The second one validates the token, and if the token is valid, it attaches the username and id of the user to the request. 

> You might be wondering why we attaches the username and id to the request after decoding the JWT - would that expose the username and id? Well, the thing is that the JWT payload is not securely encrypted in the first place. JWT use base64 encoding, which is easily reversible, and pretty much everybody can decrypt a JWT once they obtain it. The core part of JWT is to prevent tampering - since only a slight alternation of the content will create a completely different JWT. Read more [here](https://softwareengineering.stackexchange.com/questions/280257/json-web-token-why-is-the-payload-public). 
{: .prompt-info}

##### 3. Adding middleware to protected endpoints 

Finally, we need to configure the middleware in our `app.ts` file. 

> Task: Add the login endpoint and the two middlewares above to our `app.ts` file. The login and register endpoints should still be public, but the users endpoint should be protected by `jwtAuth`. 
{: .prompt-tip}

**Answer (click to unblur):**

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
{: file="backend/app.ts"}
{: .nolineno}
{: .blur}

When the user login/register, there is no JWT, so the `modifyToken` middleware will do nothing. After that, when the user is logged in, they are assigned with a JWT. When they attempts to perform authorized-only operations, requests will be sent to `userRouter` with a JWT. The request will then go through the `modifyToken` middleware, then the `jwtAuth` middleware, then finally arriving at `userRouter` if the JWT is valid. 

### Creating Contact Controller

The final part of our backend is setting up contact controllers. 

> Task: set up `getAllContacts`, `addNewContact` and `deleteById` in `contactController`. Then create a `contactRouter`, and add it to the `app.ts` file and protect with `jwtAuth`. 
{: .prompt-tip}

**Answer (click to unblur):**

```typescript
import Contact from '../models/contact';
import User from "../models/user";
import { Request, Response, NextFunction } from 'express';

export const getAllContacts = async (req: Request, res: Response, next: NextFunction) => {
  const contacts = await Contact.find({}).populate("belongsTo", { username: 1, name: 1 });
  res.json(contacts);
}

export const getById = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const contact = await Contact.findById(req.params.id);
    res.json(contact);
  } catch (err) {
    next(err);
  }
};

export const deleteById = async (req: Request, res: Response, next: NextFunction) => {
  const userId = req.user.id;

  if (!userId) return void res.status(401).send({ error: "Authentication required" });

  const user = await User.findById(userId);
  if (!user) return void res.status(400).send({ error: "User not found" });

  try {
    await Contact.findByIdAndDelete(req.params.id);
    user.contacts = user.contacts.filter(c => c.toString() != req.params.id);
    res.status(204).end();
  } catch (err) {
    next(err);
  }
}

export const addNewContacts = async (req: Request, res: Response, next: NextFunction) => { 
  const { name, number } = req.body;
  const userId = req.user.id;

  if (!userId) return void res.status(401).send({ error: "invalid token" });
  
  if (!name) {
    return void res.status(400).send({ error: "Name is required" });
  }
  if (!number) {
    return void res.status(400).send({ error: "Number is required "});
  }

  const user = await User.findById(userId);
  if (!user) return void res.status(400).send({ error: "missing userId/invalid" });
  
  const contact = new Contact({
    name,
    number,
    belongsTo: userId
  });

  console.log("ok");

  try {
    const newContact = await contact.save();
    user.contacts = user.contacts.concat(newContact._id);

    res.status(201).json(newContact);
    await user.save();
  } catch (err) {
    next(err);
  }
}
```
{: file="backend/src/controllers/contactController.ts" }
{: .nolineno }
{: .blur}

> Note that `req.params.id` is different than `req.users.id`. `req.params.id` is used to access the `/:id` inside our endpoint, not from the request payload.
{: .prompt-info }

After that you should verify your code with Postman. It is always good practice to verify your code before moving on. This is very important later on if you work on projects with multiple people on a CI/CD system - you don't want your app to break apart because your code went wrong. 
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
      : "Username"
    return void res.status(400).json({ error: `${duplicate} has already existed` });

  next(error);
};

export default errorHandler;
```
{: file="backend/middleware/errorHandler.ts"}
{: .nolineno}

Reading from the logs above, we can see the error name is `MongoServerError` and the message includes `E11000 duplicate key error`. We use that to specifically target this error. Next, we check if the duplicated value is an email or username, then returning a message based on that error. 

The next error we will tackle is `CastError`. This is thrown when an user try to access an endpoint with `/:id` but then the id is invalid (only for Mongoose; since this error is thrown if the id is an invalid MongoDb ObjectId). Try it out yourself with Postman and see the error, then add the error handling part. 

**Answer (click to unblur):**

```typescript 
	//
	if (error.name === "CastError") {
    return void res.status(400).send({ error: "Invalid id" });
```
{: file="backend/middleware/errorHandler.ts }
{: .nolineno }
{: .blur }

There are a lot more errors that I have not included. As you test your functionalities against different scenarios, you will eventually find more errors. Add them to `errorHandler` accordingly. 

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

#### Backend proxy

Before we start, if you attempts to send requests from frontend to backend without any configuration, nothing will happen. If you open the console, it would probably all errors and read something about `Cross-Origin Resource Policy`, or CORS for short. To explain shortly, it's a security feature: your frontend is running default on port 5173 (Vite default), and backend on port 3000, so they cannot communicate since they're not on the same origin. 

To mitigate this, you can install `cors` directly on backend, or edit this to your `vites.config.ts`:

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

![[Pasted image 20250708233009.png]]

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

#### Displaying contacts

Then, after the user is logged in, we should display the contacts. 

> Task: Implement displaying the list of contacts after the user is logged in. To do it, you can check if the JWT is not null. 

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

Now, the imports

```tsx
import type { LoginRequest, Contact, JwtPayload } from "@shared/types";
```
{: .nolineno}

which means that we're going to use `types.ts` again. For `Contact`, you can do it yourself - it should look exactly like our contact model (except that it's not Mongoose, of course). For `JwtPayload`:

```tsx
export interface JwtPayload {
  id: string;
  username: string;
  name: string;
  exp?: number;
  iat?: number;
}
```
{: file="@shared/types.ts"}
{: .nolineno}

You might notice this is the same as the `JwtPayload` used earlier in the backend. Yes it is indeed the same, and you should go and replace all of them with this, so that they share the same type. For `LoginRequest`: replace the `Credentials` type above with this for better naming, and import it from `types.ts` instead of defining it locally. Although not specifying `LoginRequest` for the credentials does not result in warning, it is good practice to do so. Imagine having a hundreds of request: `ContactRequest`, `DeleteRequest`, `UpdateRequest`, etc. you will quickly be overwhelmed and lose track of what are which if the types are not concrete.

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

### Register page. Routers

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

First, we create a `useNavigate` hook. This is use to navigate to a different page. In our logic, after the registration success, we will be redirected to the default page `/` (which is currently where our login page is located). We also added another cancel button at the end for users to return to homepage.

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
## Part 4: Contact Management Features

Now let's implement the contacts page and related functionality:

```tsx
import { useState, useEffect } from 'react';
import * as contactService from '../services/contactService';
import { ContactResponse, ContactInput } from '../../shared/types';
import { useNotification } from '../hooks/useNotification';
import ContactForm from './ContactForm';
import ContactList from './ContactList';
import '../styles/ContactsPage.css';

interface ContactsPageProps {
  token: string;
}

const ContactsPage = ({ token }: ContactsPageProps) => {
  const [contacts, setContacts] = useState<ContactResponse[]>([]);
  const [filter, setFilter] = useState('');
  const { showNotification } = useNotification();
  
  useEffect(() => {
    const fetchContacts = async () => {
      try {
        const contactsData = await contactService.getAll(token);
        setContacts(contactsData);
      } catch (error) {
        showNotification('Failed to fetch contacts', 'error');
      }
    };
    
    fetchContacts();
  }, [token, showNotification]);
  
  const addContact = async (contactData: ContactInput) => {
    try {
      const newContact = await contactService.create(token, contactData);
      setContacts(contacts.concat(newContact));
      showNotification(`Added ${newContact.name}`, 'success');
      return true;
    } catch (error) {
      showNotification('Failed to add contact', 'error');
      return false;
    }
  };
  
  const filteredContacts = filter
    ? contacts.filter(contact => 
        contact.name.toLowerCase().includes(filter.toLowerCase())
      )
    : contacts;
  
  return (
    <div className="contacts-container">
      <h2>Contacts</h2>
      
      <ContactForm onAddContact={addContact} />
      
      <div className="filter-container">
        <input
          type="text"
          placeholder="Filter contacts..."
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
          className="filter-input"
        />
      </div>
      
      <ContactList contacts={filteredContacts} />
    </div>
  );
};

export default ContactsPage;
```
{: file="frontend/src/components/ContactsPage.tsx" }
{: .nolineno }

Let's create the contact form component:

```tsx
import { useState } from 'react';
import { ContactInput } from '../../shared/types';
import '../styles/ContactForm.css';

interface ContactFormProps {
  onAddContact: (contact: ContactInput) => Promise<boolean>;
}

const ContactForm = ({ onAddContact }: ContactFormProps) => {
  const [name, setName] = useState('');
  const [phone, setPhone] = useState('');
  const [isFormVisible, setIsFormVisible] = useState(false);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const success = await onAddContact({ name, phone });
    
    if (success) {
      // Reset form
      setName('');
      setPhone('');
      setIsFormVisible(false);
    }
  };
  
  if (!isFormVisible) {
    return (
      <button 
        className="show-form-button"
        onClick={() => setIsFormVisible(true)}
      >
        Add New Contact
      </button>
    );
  }
  
  return (
    <div className="contact-form-container">
      <h3>Add New Contact</h3>
      <form onSubmit={handleSubmit} className="contact-form">
        <div className="form-group">
          <label htmlFor="name">Name</label>
          <input
            type="text"
            id="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
            minLength={3}
          />
        </div>
        <div className="form-group">
          <label htmlFor="phone">Phone</label>
          <input
            type="tel"
            id="phone"
            value={phone}
            onChange={(e) => setPhone(e.target.value)}
            required
          />
        </div>
        <div className="form-buttons">
          <button type="submit" className="submit-button">Add Contact</button>
          <button 
            type="button" 
            className="cancel-button"
            onClick={() => setIsFormVisible(false)}
          >
            Cancel
          </button>
        </div>
      </form>
    </div>
  );
};

export default ContactForm;
```
{: file="frontend/src/components/ContactForm.tsx" }
{: .nolineno }

And finally, the contact list component:

```tsx
import { ContactResponse } from '../../shared/types';
import '../styles/ContactList.css';

interface ContactListProps {
  contacts: ContactResponse[];
}

const ContactList = ({ contacts }: ContactListProps) => {
  if (contacts.length === 0) {
    return <p className="no-contacts">No contacts found</p>;
  }
  
  return (
    <div className="contact-list">
      {contacts.map(contact => (
        <div key={contact.id} className="contact-card">
          <h3>{contact.name}</h3>
          <p>{contact.phone}</p>
        </div>
      ))}
    </div>
  );
};

export default ContactList;
```
{: file="frontend/src/components/ContactList.tsx" }
{: .nolineno }

## Part 5: Final Touches and Improvements

Let's add a navigation bar component to improve user experience:

```tsx
import { Link } from 'react-router-dom';
import '../styles/NavBar.css';

interface NavBarProps {
  token: string | null;
  onLogout: () => void;
}

const NavBar = ({ token, onLogout }: NavBarProps) => {
  return (
    <nav className="navbar">
      <div className="navbar-logo">
        <Link to="/">Contact Manager</Link>
      </div>
      
      <div className="navbar-links">
        {token ? (
          <>
            <Link to="/contacts" className="nav-link">Contacts</Link>
            <button onClick={onLogout} className="logout-button">Logout</button>
          </>
        ) : (
          <>
            <Link to="/login" className="nav-link">Login</Link>
            <Link to="/register" className="nav-link">Register</Link>
          </>
        )}
      </div>
    </nav>
  );
};

export default NavBar;
```
{: file="frontend/src/components/NavBar.tsx" }
{: .nolineno }

And a notification component to display messages to the user:

```tsx
import { useNotification } from '../hooks/useNotification';
import '../styles/Notification.css';

const Notification = () => {
  const { notification, clearNotification } = useNotification();
  
  if (!notification) {
    return null;
  }
  
  return (
    <div className={`notification ${notification.type}`}>
      <p>{notification.message}</p>
      <button onClick={clearNotification} className="close-button">×</button>
    </div>
  );
};

export default Notification;
```
{: file="frontend/src/components/Notification.tsx" }
{: .nolineno }

## Conclusion

Congratulations! You've built a complete full-stack contact management application with TypeScript, React, and Express. This application demonstrates several important concepts:

- **Authentication**: Secure login and registration with JWT tokens
- **Data Management**: Creating and retrieving contacts from a MongoDB database
- **Type Safety**: Using TypeScript for type checking across the stack
- **User Experience**: Notifications, form validation, and proper navigation
- **Code Organization**: Clean separation of concerns with components, hooks, and services

You can extend this application in several ways:

- Add contact editing and deletion functionality
- Implement pagination for large contact lists
- Add profile management for users
- Improve the UI with more sophisticated styling

Happy coding!
