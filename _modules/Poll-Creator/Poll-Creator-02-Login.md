---
title: "The Login Page"
parent_post: Poll-Creator
module_number: 2
layout: module
---

### Building the login page

Now that we have completed all of the setup for our firebase project we are ready to start developing. The first step will be to route the user to a login page whenever the app is initialized

Navigate to the **app** directory. This is where all of the screens of our app will go. Right now there should be two files, an index.tsx and a _layout.tsx. The _layout.tsx is for shared UI across screens, and we will not be modifying it. The index.tsx is automatically the first page rendered and runs code that starts the app.

When the app is first initialized we want the index.tsx file to redirect to the login page, so let's write the code to do this. 

The basic screen to do this is:

```tsx
    import { Redirect } from "expo-router";

    export default function Index() {
        return <Redirect href="/login" />;
    }
```
{: file="index.tsx" }

In order for this to compile you'll need to create another file in the **app** directory called login.tsx. Leave this blank for now, we'll finish the index.tsx page first.

> **Tip:** This is a good time to run the program and make sure it routes to the login page
{: .prompt-info }

 Now that we have this working, we want to consider any code that we want to run upon initialization. One important thing will be to have an admin user created so that we can log in.

 > **Note:** Firebase allows you to create an admin user in the console, and this would be the best practice. However, it will be beneficial for us to show the initialization code
{: .prompt-warning }

First, let's make a place to put our admin credentials. Inside the src directory create another directory called util. This directory is for helper functions or other utilities (hence the name)

For us, it will store a hidden.js file with our admin credentials. **Remember to add this file to your .gitignore!**

Create this file and the variables for the admin email and password. The file should look like this:

```js
    const adminEmail = "...";
    const adminPassword = "...";

    export { adminEmail, adminPassword };
```
{: file="hidden.js"}

Now that we have the admin credentials ready, let's write the code in index.tsx to generate the admin account on the first initialization.

Below are the imports that need to be added at the top of the file:

```js
    import { auth } from "../src/config/firebaseConfig.js";
    import { createUserWithEmailAndPassword } from "firebase/auth";
    import { adminEmail, adminPassword } from "../src/util/hidden.js";
    import { useEffect, useState } from "react";
    import { View, ActivityIndicator } from "react-native";
```
{: file="index.tsx"}

Here's a breakdown of what each import statement does:
- The auth import allows us to access the authentication that we set up in the config file
- Likewise, the next import is a function to create a user within the auth
- Of course we want the admin credentials we just made
- These are react-specific and extremely important (they'll get their own section)
- Some extra UI components the screen

### useState and useEffect

First, let's look at the entire index.tsx file that we will have at the end. If you have experience with react make sure to skim the code and know what it does. If you don't, pay careful attention to this section.

> **Tip:** Few comments will be provided in code given to you in this tutorial. A great way to understand the code step by step is to go through any code provided and add your own comments to increase understanding and help you remember what you learned
{: .prompt-info }

```tsx
    export default function Index() {
        const [isReady, setIsReady] = useState(false);

        async function createAdmin() {
            try {
                await createUserWithEmailAndPassword(auth, adminEmail, adminPassword);
                console.log("Admin account created successfully.");
            } catch (error) {
                if ((error as { code: string }).code === "auth/email-already-in-use") {
                console.log("Admin account already exists. Proceeding to login...");
                } else {
                console.error("Failed to create admin account", error);
                }
            }   finally {
                setIsReady(true);
            }
        }

        useEffect(() => {
            createAdmin();
        }, []); 

        if (!isReady) {
            return (
            <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
                <ActivityIndicator size="large" />
            </View>
            );
        }

        return <Redirect href="/login" />;
        }
```
{: file="index.tsx"}

First, we'll discuss the first line:

```tsx
    const [isReady, setIsReady] = useState(false);
```
{: file="index.tsx"}

A useState is declared like a variable with two items within a set of brackets. The first is the variable name, in this case isReady. The second is a function that will be used to update the value of our variable, in this case setIsReady. Even though setIsReady is a function **you do not need to write what it does**, it automatically takes one parameter and updates the variable to take on that value.

> **Tip:** Another value setIsReady can take is a function of the initial state. For example, setIsReady could flip the value of isReady from false to true and vice versa
{: .prompt-info }

You must always set this equal to a useState with the default value in parentheses as shown above. The useState function will return an array that we decompose into isReady and setIsReady

### Why a useState?

The useState will help us re-render our screen if something changes. Whenever setIsReady is called, React will "notice" this and re-render the component associated with the useState (in this case, the screen). Thus, whenever the async function completed and we set isReady to true the Index() function will run. Since isReady will have changed, the code to change screens will run

### What's with the useEffect?

Why shouldn't we just use createAdmin() as a regular function in this case. Well, according to the useState logic we just wrote the component re-renders every time isReady changes. If isReady were to change often for some reason, we would be running createAdmin() every single time, which is wasteful. Thus, we write it as a useEffect.

The syntax for a useEffect is as follows:

```tsx
    useEffect(/*function to be executed*/, /*dependency array*/);
```
{: file="example.tsx"}

A dependency array is a list of useStates (or other triggers) that when changed should trigger the useEffect to run. We could put isReady within the dependency array if we wanted the useEffect to run every time that value was changed. An empty dependency array indicates that the useEffect should run once on the initial render.

### useStates and useEffects are the bread and butter of React

Take a moment to read back over the program and consider how the useState and useEffect allow the UI to "react" to changes. This is key to understanding React and will be necessary to complete parts of the tutorial. 

**Suggested Practice**: take a few minutes to write a simple UI that has a useState (maybe a count) that affects some other aspect of the UI (say a count tracker). Add the count useState to the dependency array of a useEffect that performs some visual change on the screen. This will help you get the feel for how these components work.
(*This will not be a part of the final tutorial but is good for someone just learning useStates/useEffects*)

### The Login Page

Now let's start to fill out the code for our login.tsx. We want to have an admin panel from which the admin can invite guests as well as a guest screen where the poll can be taken. We won't fill these yet, but let's create **admin.tsx** and **poll.tsx**

Let's start with the code to log in the admin with the credentials that we made in the previous section. 

```tsx
import { View, Text, TextInput, Button, StyleSheet } from "react-native";
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";
import { useState } from "react";
import { adminEmail } from "../src/util/hidden.js";
import { router } from "expo-router";


const auth = getAuth();

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  const handleLogin = () => {
  setError("");

  signInWithEmailAndPassword(auth, email, password)
    .then(async (userCredential) => {
      const user = userCredential.user;

      if (user.email !== adminEmail) {
        return;
      }

      router.replace("/admin");
      return;
    })
    .catch(() => {
      setError("Invalid email or password.");
    });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>

      <TextInput 
        style={styles.input} 
        placeholder="Email" 
        value={email}
        onChangeText={setEmail}
      />
      <TextInput 
        style={styles.input} 
        placeholder="Password" 
        secureTextEntry
        value={password}
        onChangeText={setPassword}
      />

      <Button title="Log In" onPress={handleLogin} />
      {error && <Text style={{ color: "red" }}>{error}</Text>}

    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    padding: 24,
  },
  title: {
    fontSize: 32,
    fontWeight: "bold",
    marginBottom: 24,
  },
  input: {
    borderWidth: 1,
    padding: 12,
    marginBottom: 16,
  },
});
```
{: file="login.tsx"}

Notice how we use useStates in this case. We don't need to render the email and password visually, so why "track" them with a useState?

Well, when the user hits login we want the version of email and password that are in those corresponding fields. A useState is a great way to make sure that we have access to the most recent email/password the user typed in.

You'll also notice most of this file is UI and styling. This creates a simple UI that is merely functional, but students interested in design are encouraged to make each screen more user-friendly and visually appealing.

**Make sure to test this!** Create a basic UI for the admin screen and ensure that the credentials and routing work. If you've forgotten the credentials you chose they are in hidden.js