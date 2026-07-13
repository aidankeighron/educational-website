---
title: "Creating the Express Application"
parent_post: Express-Auth
module_number: 3
layout: module
media_subpath: /assets/tutorials/express-auth
---

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
    "dev": "nodemon --watch 'src/**/*.ts' --exec 'ts-node -r tsconfig-paths/register' src/index.ts",
  }
```
{: file="backend/package.json"}
{: .nolineno}

The important part here is the `-r tsconfig-paths/register` part. This will enable path mapping support (like `@shared/types`) and without this your `@shared/*` imports won't work. You can look up the rest if you don' understand.

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

