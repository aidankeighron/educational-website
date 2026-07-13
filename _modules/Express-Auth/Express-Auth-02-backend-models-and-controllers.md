---
title: "Backend models and controllers"
parent_post: Express-Auth
module_number: 2
layout: module
media_subpath: /assets/tutorials/express-auth
---

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

