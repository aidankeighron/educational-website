---
title: "Setup the Project"
parent_post: Poll-Creator
module_number: 1
layout: module
---

### Setup

Create a repository for this project named poll-creator. In addition to having the repository opened in your preferred IDE, you will need to have several tools installed. If you already have any of these tools installed skip ahead to the start of the project.

**What to install:**

- Node.js ([install here](https://nodejs.org/en/download))
- Expo Go (download from your phone's app store)


### Create the Expo App

For this project we will be using Expo, which is the officially recommended framework for working with react native. Expo will allow us to test our code with the Expo Go app you downloaded in the previous step. We will also be using the Expo Router to navigate between the pages of the mobile application.

First we will set up the expo project --- Once in the project folder, run the following command to generate a subfolder for the app:

    npx create-expo-app@latest my-app

> **WARNING:** You will be prompted to choose an SDK version, one of which is recommended for development with **Expo Go** — choose this one.
{: .prompt-danger }

The previous command installed everything needed for developing with react native in Expo Go. Now enter this app by running the terminal command:

    cd my-app

At any point if you want to run the app and see what it looks like on your device, run the following command in the terminal

    npx expo start --tunnel

> **Tip:** Sometimes the tunnel won't load due to an error in the cache. Adding the flag \-\-clear before \-\-tunnel will clear the bundler cache and rebuild it which often fixes the issue
{: .prompt-info }

Scan the QR code generated to run the app with your Expo Go app. When we generated the expo app it automatically generated a basic application, but we don't want this app and are going to discard it. Run the following command in the terminal to reset the project to a blank template (**ensure you are in the right directory**)

    npx run reset-project

> **Tip:** You will have the option to either keep or remove the app-example directory when the project is reset. If you have never developed with react native before it may be helpful to have the directory as a reference
{: .prompt-info }

Now the file structure will have two similar directories, my-app and app. When running the program the command will be run from inside the **my-app** directory. When adding the screens to our application those will go inside **app** which is inside the my-app directory


### Setting up Firebase

For this project we are going to use two tools from firebase: authentication and database (auth and db). The auth will allow us to keep track of login information and support multiple users. We will be using the db for storing the poll and its results.

Go to [Firebase Console](https://console.firebase.google.com/) and sign in with your Google account
> **Tip:** Using a personal Google account is best since it will allow you to keep using Firebase after your MSU email is deleted when you graduate
{: .prompt-info }

Once in the Firebase Console click the button to create a new project and follow the directions for setup. There will be optional choices for AI assistance and Google analytics. This project doesn't require these tools so it's up to you whether you want to enable them or not

### Registering the App with Firebase

Before Firebase can be used in the Expo project, the application needs to be registered with the Firebase project.

In the Firebase Console, open the project you created and go to Project settings (the gear icon). Scroll down to the Your apps section and choose **Web App**

After registering the app, Firebase will display a configuration object that we will use later. Copy the middle section that looks like this:

```js
    // Your web app's Firebase configuration
    const firebaseConfig = {
        apiKey: "...",
        authDomain: "...",
        projectId: "...",
        storageBucket: "...",
        messagingSenderId: "...",
        appId: "..."
    };
```
{: file = "firebaseConfig.js"}

To use the auth and db that we we will need a config file. This file will initialize both tools and export them for use in our program. We are going to put the code we just copied into this file and complete it later

- Make a directory called src inside of **my-app** and then make another directory called config inside of the new src directory
- Create a file called firebaseConfig.js inside of the config directory
    
> **Tip:** While we will only have one config file, for larger projects there can be many. We'll be using the industry standard file structure even though it isn't needed at this scale
{: .prompt-info }

Next, install the Firebase JavaScript SDK. Make sure you are inside the my-app directory before running the command:

    npm install firebase

### Setting up the Authentication & Database

Now that Firebase is installed and your app is registered, we can start to import firebase tools into the project. We will need to add the tools to our project in the console and then set them up in the firebaseConfig.js file that we made
- In the Firebase Console, go to Security &rarr; Authentication. Under Sign-in method, select Email/Password, enable it, and select Save.
- In the Firebase Console, go to Databases & Storage &rarr; Firestore Database and select Create database

The template for the firebase config that we want (complete with both auth and db configuration) is provided below. This will initialize auth and db when we load the app and the exports will allow us to reference the auth or db when we are writing out logic for the screens

```js
    import { initializeApp } from "firebase/app";
    import { getAuth } from "firebase/auth";
    import { getFirestore } from "firebase/firestore";

    //*THIS IS THE CODE YOU COPIED WHEN REGISTERING*
    const firebaseConfig = {
        //...
    };

    const app = initializeApp(firebaseConfig);

    export const auth = getAuth(app);
    export const db = getFirestore(app);
```
{: file="firebaseConfig.js" }

> **Tip:** If at any point you need to access your firebaseConfig object again go to Firebase Console &rarr; Settings &rarr; General. Scroll down and select "Config"
{: .prompt-info }