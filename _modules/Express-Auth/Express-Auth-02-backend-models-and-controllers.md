---
title: "Backend models and controllers"
parent_post: Express-Auth
module_number: 2
layout: module
media_subpath: /assets/tutorials/express-auth
---

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
