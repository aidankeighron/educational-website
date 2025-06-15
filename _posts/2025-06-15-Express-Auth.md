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

Our application consists of three main parts:

1. **Backend**: Node.js with Express and TypeScript, connected to MongoDB
2. **Frontend**: React with TypeScript, using hooks and context
3. **Shared Types**: Common TypeScript interfaces used by both frontend and backend

The application will allow users to:

- Register a new account
- Log in with secure authentication
- View their contacts
- Add new contacts
- Filter contacts by name

Let's get started!

## Part 1: Setting Up the Backend

We'll begin by setting up our TypeScript configuration and creating the basic structure of our backend.

### Initial Project Setup

First, let's create a `tsconfig.json` file for our TypeScript configuration:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
}
```
{: file="backend/tsconfig.json" }
{: .nolineno }

This configuration sets up TypeScript with modern JavaScript features and strict type checking.

### Creating the Express Application

Next, let's create our main Express application in `app.ts`:

```typescript
import express from 'express';
import mongoose from 'mongoose';
import config from './config';

import loginRouter from './routers/loginRouter';
import registerRouter from './routers/registerRouter';
import unknownEndpoint from './middlewares/unknownEndpoint';
import errorHandler from './middlewares/errorHandler';
import contactRouter from './routers/contactRouter';
import userRouter from './routers/userRouter';
import modifyToken from './middlewares/modifyToken';
import { jwtAuth } from './middlewares/jwtAuth';

const app = express();

console.log("connecting to ", config.MONGODB_URI);
mongoose
  .connect(config.MONGODB_URI)
  .then(() => console.log("connected to MongoDB"))
  .catch((error) =>
    console.log("error connecting to MongoDB: ", error.message)
  );

app.use(express.static("dist"));
app.use(express.json());

app.use(modifyToken);

// Public routes (no authentication required)
app.use("/api/login", loginRouter);
app.use("/api/register", registerRouter);

// Apply JWT authentication for protected routes
app.use("/api/users", jwtAuth, userRouter);
app.use("/api/contacts", jwtAuth, contactRouter);

// Error handling middlewares
app.use(unknownEndpoint);
app.use(errorHandler);

export default app;
```
{: file="backend/src/app.ts" }
{: .nolineno }

This sets up our Express application with routes for authentication, user management, and contact management. Note how we separate public routes from protected routes that require JWT authentication.

### Configuration

Let's create a configuration file to handle environment variables:

```typescript
import dotenv from 'dotenv';
dotenv.config();

const PORT = process.env.PORT || 3001;
const MONGODB_URI = process.env.MONGODB_URI || 'mongodb://localhost:27017/contactApp';
const JWT_SECRET = process.env.JWT_SECRET || 'your_secret_key';

export default {
  PORT,
  MONGODB_URI,
  JWT_SECRET
};
```
{: file="backend/src/config.ts" }
{: .nolineno }

### Entry Point

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

### Setting Up Models

Now, let's create our MongoDB models for User and Contact:

```typescript
import mongoose from 'mongoose';
import { User } from '../../../shared/types';

const userSchema = new mongoose.Schema<User>({
  username: {
    type: String,
    required: true,
    minlength: 3,
    unique: true
  },
  email: {
    type: String,
    required: true,
    unique: true
  },
  passwordHash: {
    type: String,
    required: true
  },
});

userSchema.set('toJSON', {
  transform: (_document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
    // Don't reveal the hash
    delete returnedObject.passwordHash;
  }
});

export default mongoose.model<User>('User', userSchema);
```
{: file="backend/src/models/user.ts" }
{: .nolineno }

And for our Contact model:

```typescript
import mongoose from 'mongoose';
import { Contact } from '../../../shared/types';

const contactSchema = new mongoose.Schema<Contact>({
  name: {
    type: String,
    required: true,
    minlength: 3
  },
  phone: {
    type: String,
    required: true
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
});

contactSchema.set('toJSON', {
  transform: (_document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
  }
});

export default mongoose.model<Contact>('Contact', contactSchema);
```
{: file="backend/src/models/contact.ts" }
{: .nolineno }

### Authentication Middleware

Let's implement JWT authentication middleware:

```typescript
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';
import config from '../config';
import { JwtPayload } from '../../../shared/types';

export const jwtAuth = (req: Request, res: Response, next: NextFunction) => {
  const token = req.token;
  
  if (!token) {
    return res.status(401).json({ error: 'token missing' });
  }
  
  try {
    const decodedToken = jwt.verify(token, config.JWT_SECRET) as JwtPayload;
    
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

And a middleware to extract the token from the request:

```typescript
import { Request, Response, NextFunction } from 'express';

const modifyToken = (req: Request, _res: Response, next: NextFunction) => {
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

### Router Implementation

Let's implement our login router:

```typescript
import express from 'express';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import User from '../models/user';
import config from '../config';
import { LoginCredentials } from '../../../shared/types';

const loginRouter = express.Router();

loginRouter.post('/', async (req, res) => {
  const { username, password } = req.body as LoginCredentials;

  const user = await User.findOne({ username });
  
  const passwordCorrect = user === null
    ? false
    : await bcrypt.compare(password, user.passwordHash);

  if (!(user && passwordCorrect)) {
    return res.status(401).json({
      error: 'invalid username or password'
    });
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  };

  const token = jwt.sign(
    userForToken, 
    config.JWT_SECRET,
    { expiresIn: 60*60 }
  );

  res
    .status(200)
    .send({ token, username: user.username, email: user.email });
});

export default loginRouter;
```
{: file="backend/src/routers/loginRouter.ts" }
{: .nolineno }

And our registration router:

```typescript
import express from 'express';
import bcrypt from 'bcrypt';
import User from '../models/user';
import { RegisterCredentials } from '../../../shared/types';

const registerRouter = express.Router();

registerRouter.post('/', async (req, res) => {
  const { username, email, password } = req.body as RegisterCredentials;

  if (!username || !email || !password) {
    return res.status(400).json({
      error: 'username, email, and password are required'
    });
  }

  if (username.length < 3) {
    return res.status(400).json({
      error: 'username must be at least 3 characters long'
    });
  }

  if (password.length < 5) {
    return res.status(400).json({
      error: 'password must be at least 5 characters long'
    });
  }

  const existingUser = await User.findOne({ username });
  if (existingUser) {
    return res.status(400).json({
      error: 'username must be unique'
    });
  }

  const existingEmail = await User.findOne({ email });
  if (existingEmail) {
    return res.status(400).json({
      error: 'email must be unique'
    });
  }

  const saltRounds = 10;
  const passwordHash = await bcrypt.hash(password, saltRounds);

  const user = new User({
    username,
    email,
    passwordHash,
  });

  const savedUser = await user.save();

  res.status(201).json(savedUser);
});

export default registerRouter;
```
{: file="backend/src/routers/registerRouter.ts" }
{: .nolineno }

Let's also implement our contact router:

```typescript
import express from 'express';
import Contact from '../models/contact';
import User from '../models/user';
import { ContactInput } from '../../../shared/types';

const contactRouter = express.Router();

// Get all contacts for the authenticated user
contactRouter.get('/', async (req, res) => {
  const userId = req.user?.id;
  
  const contacts = await Contact.find({ user: userId });
  res.json(contacts);
});

// Create a new contact
contactRouter.post('/', async (req, res) => {
  const body = req.body as ContactInput;
  const userId = req.user?.id;
  
  const user = await User.findById(userId);
  
  if (!user) {
    return res.status(404).json({ error: 'user not found' });
  }
  
  if (!body.name || !body.phone) {
    return res.status(400).json({ error: 'name and phone are required' });
  }
  
  const contact = new Contact({
    name: body.name,
    phone: body.phone,
    user: user._id
  });
  
  const savedContact = await contact.save();
  res.status(201).json(savedContact);
});

export default contactRouter;
```
{: file="backend/src/routers/contactRouter.ts" }
{: .nolineno }

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

The project structure we've created is scalable and can be used as a foundation for more complex applications.

**Answer (click to unblur):**
Building a full-stack application with proper authentication and data management can be accomplished in a relatively short time by following good architectural patterns and leveraging TypeScript for type safety across the entire stack.
{: .prompt-info }

By following the commit history of this project, you can see how it evolved from a simple backend to a complete application with authentication, data management, and a user-friendly interface. This incremental approach is a great way to build complex applications without getting overwhelmed.

Happy coding!
