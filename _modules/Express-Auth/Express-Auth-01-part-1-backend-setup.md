---
title: "Part 1: Backend setup"
parent_post: Express-Auth
module_number: 1
layout: module
media_subpath: /assets/tutorials/express-auth
---

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

