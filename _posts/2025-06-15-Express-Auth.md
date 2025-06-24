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
- Filter contacts by name

Let's get started!

## Part 1: Backend setup

We'll begin by setting up our TypeScript configuration and creating the basic structure of our backend. 

### Initial Project Setup

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

A *model* or *schema* is basically how our data is stored inside the database. It is a blueprint to tell us how should the data look like (e.g. which fields should the data have, the restrictions to each field, etc). The two main types of database are *SQL* and *NoSQL*. Basically, a *SQL* database require the data to follow the schema as strictly as possible, and invalid data (which does not follow the schema) will not allowed to be persisted. On the other hand, *NoSQL* database are database that are more flexible, allowing users to store data that have wildly different schemas. 

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

This file only consists of GET-ing users. For adding users, we will handle that in a different file, `registerController`. But I'll handle that to you. 

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
{: .nolineno }

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

{: file="backend/src/app.ts" }
{: .nolineno }

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

// Basic error handling for unknown endpoints
app.use((req: Request, res: Response) => {
  res.status(404).send({ error: "unknown endpoint" });
});

export default app;
```

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

First, the user log in with credentials. If the credentials match, JWT is generated. The user can then use the JWT to perform authorized-only operations (e.g. adding a contact to an user's contact list). 

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
 
  if (!config.SECRET_KEY) {
    throw new Error("SECRET_KEY is not defined in config");
  }

  const token = jwt.sign(payload, config.SECRET_KEY, { expiresIn: 60*60 });

  res.status(200).send({ token, username: username, name: user.name});
}
```
{: file="backend/src/controllers/loginController.ts" }
{: .nolineno }

The login process works by first finding an user with the same username as provided by the request. Then, it hashes the password received from the request and compare it against the one queried from the database. If the username is not valid or the password is incorrect, it sends back a `401 unauthorized`. Otherwise, a JWT is signed along with the payload and returned.

#### Creating JWT Middleware

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

Typically, the JWT token will be sent through the `Authorized` header, as we seen above. For now, just use Postman to login first, get the token, and then send the token manually in the `Authorization` header when we want authorized access. We will persist and automatically use the JWT when we develop the frontend. 

##### 1. Token Extraction Middleware

This middleware extracts the token from the `Authorization` header:

```typescript
import { Request, Response, NextFunction } from 'express';
import '../../../shared/types'; // Import our type extensions

const modifyToken = (req: Request, res: Response, next: NextFunction) => {
  const authorization = req.get('authorization');
  
  if (authorization && authorization.toLowerCase().startsWith('bearer ')) {
    req.token = authorization.substring(7);
  }
  
  next();
};

export default modifyToken;
```
{: file="backend/src/middlewares/modifyToken.ts" }
{: .nolineno }

This middleware:

- Checks for the `Authorization` header
- Extracts the token part from `Bearer <token>` format
- Attaches the token to the request object for later use

> **TypeScript Magic**: Notice how we can assign `req.token = authorization.substring(7)` without TypeScript throwing an error, even though the Express `Request` object doesn't normally have a `token` property. This works because we've extended the Express Request interface in our `shared/types.ts` file. TypeScript ensures we only attach fields that we've explicitly declared, preventing accidental property assignments and giving us better type safety and IntelliSense support.
{: .prompt-info}

##### 2. JWT Authentication Middleware

This middleware validates the JWT token and extracts user information:

```typescript
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';
import config from '../config';
import '../../../shared/types'; // Import our type extensions

interface JwtPayload {
  id: string;
  username: string;
  iat?: number;
  exp?: number;
}

export const jwtAuth = (req: Request, res: Response, next: NextFunction) => {
  const token = req.token;
  
  if (!token) {
    return res.status(401).json({ error: 'token missing' });
  }
  
  try {
    const decodedToken = jwt.verify(token, config.SECRET_KEY!) as JwtPayload;
    
    if (!decodedToken.id) {
      return res.status(401).json({ error: 'invalid token' });
    }
    
    req.user = decodedToken;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'invalid token' });
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

> **TypeScript Type Safety**: Here we're adding `req.user = decodedToken` to attach the authenticated user's information to the request object. Again, this works seamlessly because of our type extensions in `shared/types.ts`. TypeScript gives us:
>
> - **Autocomplete**: When we type `req.user.`, we get IntelliSense suggestions for `id` and `username`
> - **Type Checking**: If we try to access `req.user.nonexistentField`, TypeScript will catch this error
> - **Clear Data Types**: We know exactly what data structure we're working with across our entire application
{: .prompt-info}

#### Creating Contact Controller

Now let's create a controller for handling contacts that uses the authenticated user information:

```typescript
import { Request, Response, NextFunction } from 'express';
import Contact from '../models/contact';
import User from '../models/user';

export const getAllContacts = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const userId = req.user?.id;
    const contacts = await Contact.find({ belongsTo: userId });
    res.json(contacts);
  } catch (err) {
    next(err);
  }
};

export const addNewContact = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { name, number } = req.body;
    const userId = req.user?.id;

    const user = await User.findById(userId);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    if (!name || !number) {
      return res.status(400).json({ error: 'Name and number are required' });
    }

    const contact = new Contact({
      name,
      number,
      belongsTo: userId
    });

    const savedContact = await contact.save();
    
    // Add contact to user's contacts array
    user.contacts = user.contacts.concat(savedContact._id);
    await user.save();

    res.status(201).json(savedContact);
  } catch (err) {
    next(err);
  }
};
```
{: file="backend/src/controllers/contactController.ts" }
{: .nolineno }

Notice how we use `req.user?.id` to get the authenticated user's ID, which was set by our JWT middleware.

#### Updating Routers

Let's create routers for our new endpoints:

```typescript
import express from 'express';
import { login } from '../controllers/loginController';

const loginRouter = express.Router();

loginRouter.post('/', login);

export default loginRouter;
```
{: file="backend/src/routers/loginRouter.ts" }
{: .nolineno }

```typescript
import express from 'express';
import { getAllContacts, addNewContact } from '../controllers/contactController';

const contactRouter = express.Router();

contactRouter.get('/', getAllContacts);
contactRouter.post('/', addNewContact);

export default contactRouter;
```
{: file="backend/src/routers/contactRouter.ts" }
{: .nolineno }

#### Adding Middleware Support to Express Types

We need to extend the Express Request interface to include our custom properties:

```typescript
// Add this to your shared/types.ts file

declare global {
  namespace Express {
    interface Request {
      token?: string;
      user?: {
        id: string;
        username: string;
      };
    }
  }
}
```
{: file="shared/types.ts" }
{: .nolineno }

#### Updating the Main App

Now let's update our main app to use the new middleware and routes:

```typescript
import express from 'express';
import mongoose from 'mongoose';
import config from './config';
import cors from 'cors';

import loginRouter from './routers/loginRouter';
import registerRouter from './routers/registerRouter';
import userRouter from './routers/userRouter';
import contactRouter from './routers/contactRouter';
import modifyToken from './middlewares/modifyToken';
import { jwtAuth } from './middlewares/jwtAuth';

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

// Extract token from all requests
app.use(modifyToken);

// Public routes (no authentication required)
app.use("/api/login", loginRouter);
app.use("/api/register", registerRouter);

// Protected routes (authentication required)
app.use("/api/users", jwtAuth, userRouter);
app.use("/api/contacts", jwtAuth, contactRouter);

// Basic error handling for unknown endpoints
app.use((req, res) => {
  res.status(404).send({ error: "unknown endpoint" });
});

export default app;
```
{: file="backend/src/app.ts" }
{: .nolineno }

#### Testing Authentication with Postman

Now let's test our authentication system:

##### 1. First, Login to Get a Token

**POST** `http://localhost:3001/api/login`

Body (JSON):
```json
{
  "username": "johndoe",
  "password": "password123"
}
```

Expected Response (200 OK):
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "username": "johndoe",
  "name": "John Doe"
}
```

![[Pasted image 20250623234755.png]]

##### 2. Test Protected Route Without Token

**GET** `http://localhost:3001/api/contacts`

No Authorization header

Expected Response (401 Unauthorized):
```json
{
  "error": "token missing"
}
```

##### 3. Test Protected Route With Token

**GET** `http://localhost:3001/api/contacts`

Headers:
```json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Expected Response (200 OK):
```json
[]
```

##### 4. Create a New Contact

**POST** `http://localhost:3001/api/contacts`

Headers:
```json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

Body (JSON):
```json
{
  "name": "Jane Smith",
  "number": "123-4567890"
}
```

Expected Response (201 Created):
```json
{
  "name": "Jane Smith",
  "number": "123-4567890",
  "belongsTo": "60f7b3b3b3b3b3b3b3b3b3b3",
  "id": "60f7b3b3b3b3b3b3b3b3b3b4"
}
```

##### 5. Get Contacts Again

**GET** `http://localhost:3001/api/contacts`

Headers:
```json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Expected Response (200 OK):
```json
[
  {
    "name": "Jane Smith",
    "number": "123-4567890",
    "belongsTo": "60f7b3b3b3b3b3b3b3b3b3b3",
    "id": "60f7b3b3b3b3b3b3b3b3b3b4"
  }
]
```

#### Common Authentication Errors

1. **401 - token missing**: No Authorization header provided
2. **401 - invalid token**: Token is malformed, expired, or signed with wrong secret
3. **403 - forbidden**: Token is valid but user doesn't have permission (not implemented in our simple app)

> **Important**: Always include the word "Bearer" before your token in the Authorization header. The format should be: `Bearer <your-jwt-token>`
{: .prompt-warning }

Now you have a fully functional authentication system! Users must log in to get a token, and that token is required to access protected resources like contacts. Each user can only see and manage their own contacts.

## Part 2: Building the Frontend Foundation

Now that our backend is set up, let's move on to creating the frontend of our application. We'll use React with TypeScript and Vite for fast development.

### Setting Up the Frontend

Let's start by creating our main React component:

```tsx
import { useState, useEffect } from 'react';
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import LoginPage from './components/LoginPage';
import RegisterPage from './components/RegisterPage';
import ContactsPage from './components/ContactsPage';
import NotificationProvider from './contexts/NotificationContext';
import Notification from './components/Notification';
import NavBar from './components/NavBar';
import { useLogin } from './hooks/useLogin';
import './styles/App.css';

const App = () => {
  const { token, login, logout } = useLogin();
  
  return (
    <Router>
      <NotificationProvider>
        <div className="app-container">
          <NavBar token={token} onLogout={logout} />
          <Notification />
          
          <Routes>
            <Route path="/login" element={
              !token ? <LoginPage onLogin={login} /> : <Navigate to="/contacts" />
            } />
            <Route path="/register" element={
              !token ? <RegisterPage /> : <Navigate to="/contacts" />
            } />
            <Route path="/contacts" element={
              token ? <ContactsPage token={token} /> : <Navigate to="/login" />
            } />
            <Route path="/" element={<Navigate to={token ? "/contacts" : "/login"} />} />
          </Routes>
        </div>
      </NotificationProvider>
    </Router>
  );
};

export default App;
```
{: file="frontend/src/App.tsx" }
{: .nolineno }

### Creating Shared Types

Let's define our shared types that will be used by both frontend and backend:

```typescript
import { Document } from 'mongoose';

// User related types
export interface User extends Document {
  username: string;
  email: string;
  passwordHash: string;
}

export interface UserResponse {
  id: string;
  username: string;
  email: string;
}

export interface LoginCredentials {
  username: string;
  password: string;
}

export interface RegisterCredentials {
  username: string;
  email: string;
  password: string;
}

export interface LoginResponse {
  token: string;
  username: string;
  email: string;
}

// Contact related types
export interface Contact extends Document {
  name: string;
  phone: string;
  user: User['_id'];
}

export interface ContactInput {
  name: string;
  phone: string;
}

export interface ContactResponse {
  id: string;
  name: string;
  phone: string;
}

// JWT related types
export interface JwtPayload {
  id: string;
  username: string;
  iat?: number;
  exp?: number;
}

// For Express request augmentation
declare global {
  namespace Express {
    interface Request {
      token?: string;
      user?: JwtPayload;
    }
  }
}
```
{: file="shared/types.ts" }
{: .nolineno }

### Creating API Services

Let's create services to handle API requests:

```typescript
import axios from 'axios';
import { LoginCredentials, LoginResponse } from '../../shared/types';

const baseUrl = '/api/login';

export const login = async (credentials: LoginCredentials): Promise<LoginResponse> => {
  const response = await axios.post<LoginResponse>(baseUrl, credentials);
  return response.data;
};
```
{: file="frontend/src/services/loginService.ts" }
{: .nolineno }

And a service for contact operations:

```typescript
import axios from 'axios';
import { ContactInput, ContactResponse } from '../../shared/types';

const baseUrl = '/api/contacts';

export const getAll = async (token: string): Promise<ContactResponse[]> => {
  const config = {
    headers: { Authorization: `Bearer ${token}` },
  };
  
  const response = await axios.get<ContactResponse[]>(baseUrl, config);
  return response.data;
};

export const create = async (token: string, newContact: ContactInput): Promise<ContactResponse> => {
  const config = {
    headers: { Authorization: `Bearer ${token}` },
  };
  
  const response = await axios.post<ContactResponse>(baseUrl, newContact, config);
  return response.data;
};
```
{: file="frontend/src/services/contactService.ts" }
{: .nolineno }

## Part 3: Adding Authentication

Now, let's implement the login functionality with a custom hook and components.

### Creating the Login Hook

```typescript
import { useState, useEffect } from 'react';
import { LoginCredentials } from '../../shared/types';
import * as loginService from '../services/loginService';

export const useLogin = () => {
  const [token, setToken] = useState<string | null>(null);
  const [username, setUsername] = useState<string | null>(null);
  
  useEffect(() => {
    const savedToken = localStorage.getItem('userToken');
    const savedUsername = localStorage.getItem('username');
    
    if (savedToken && savedUsername) {
      setToken(savedToken);
      setUsername(savedUsername);
    }
  }, []);
  
  const login = async (credentials: LoginCredentials) => {
    try {
      const response = await loginService.login(credentials);
      
      localStorage.setItem('userToken', response.token);
      localStorage.setItem('username', response.username);
      
      setToken(response.token);
      setUsername(response.username);
      
      return { success: true };
    } catch (error) {
      if (axios.isAxiosError(error) && error.response) {
        return { 
          success: false, 
          error: error.response.data.error || 'Login failed' 
        };
      }
      return { success: false, error: 'Login failed' };
    }
  };
  
  const logout = () => {
    localStorage.removeItem('userToken');
    localStorage.removeItem('username');
    setToken(null);
    setUsername(null);
  };
  
  return { token, username, login, logout };
};
```
{: file="frontend/src/hooks/useLogin.ts" }
{: .nolineno }

### Creating the Login Component

```tsx
import { useState } from 'react';
import { LoginCredentials } from '../../shared/types';
import { useNotification } from '../hooks/useNotification';
import { Link } from 'react-router-dom';
import '../styles/LoginPage.css';

interface LoginPageProps {
  onLogin: (credentials: LoginCredentials) => Promise<{ success: boolean; error?: string }>;
}

const LoginPage = ({ onLogin }: LoginPageProps) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const { showNotification } = useNotification();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const result = await onLogin({ username, password });
    
    if (result.success) {
      showNotification('Login successful!', 'success');
    } else {
      showNotification(result.error || 'Login failed', 'error');
    }
  };
  
  return (
    <div className="login-container">
      <h2>Login</h2>
      <form onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="username">Username</label>
          <input
            type="text"
            id="username"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            required
          />
        </div>
        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            type="password"
            id="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />
        </div>
        <button type="submit" className="login-button">Login</button>
      </form>
      <div className="register-link">
        <p>Don't have an account? <Link to="/register">Register here</Link></p>
      </div>
    </div>
  );
};

export default LoginPage;
```
{: file="frontend/src/components/LoginPage.tsx" }
{: .nolineno }

### Creating a Notification System

Let's implement a notification context and hook for user feedback:

```tsx
import { createContext, useState, ReactNode } from 'react';

type NotificationType = 'success' | 'error' | 'info';

interface Notification {
  message: string;
  type: NotificationType;
}

interface NotificationContextType {
  notification: Notification | null;
  showNotification: (message: string, type: NotificationType) => void;
  clearNotification: () => void;
}

export const NotificationContext = createContext<NotificationContextType>({
  notification: null,
  showNotification: () => {},
  clearNotification: () => {},
});

interface NotificationProviderProps {
  children: ReactNode;
}

const NotificationProvider = ({ children }: NotificationProviderProps) => {
  const [notification, setNotification] = useState<Notification | null>(null);
  
  const showNotification = (message: string, type: NotificationType) => {
    setNotification({ message, type });
    
    // Auto clear after 5 seconds
    setTimeout(() => {
      setNotification(null);
    }, 5000);
  };
  
  const clearNotification = () => {
    setNotification(null);
  };
  
  return (
    <NotificationContext.Provider value=\{\{ notification, showNotification, clearNotification \}\}>
      {children}
    </NotificationContext.Provider>
  );
};

export default NotificationProvider;
```
{: file="frontend/src/contexts/NotificationContext.tsx" }
{: .nolineno }

And the corresponding hook:

```typescript
import { useContext } from 'react';
import { NotificationContext } from '../contexts/NotificationContext';

export const useNotification = () => {
  return useContext(NotificationContext);
};
```
{: file="frontend/src/hooks/useNotification.ts" }
{: .nolineno }

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
