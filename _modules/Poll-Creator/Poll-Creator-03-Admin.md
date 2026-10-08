---
title: "The Admin Page"
parent_post: Poll-Creator
module_number: 3
layout: module
---

### The admin page

We want the admin to have various capabilities:

- Create a poll
- Invite users to take the poll
- Delete users that should not take the poll
- View the poll results

We're going to work on functionality 1-3 right now and save the poll results for after we make the poll. The first two capabilities are provided in the file below. You'll implement the third capability.

```tsx
    import { View, Text, TextInput, Button, StyleSheet, Alert } from "react-native";
    import { useState } from "react";
    import { createUserWithEmailAndPassword, signInWithEmailAndPassword} from "firebase/auth";
    import { auth, db } from "@/src/config/firebaseConfig";
    import { adminEmail, adminPassword } from "@/src/util/hidden";
    import { doc, getDoc, setDoc, deleteDoc, collection, getDocs } from "firebase/firestore";
    import { router } from "expo-router";

    const maxOptions = 5;

    export default function Admin() {
    const [numOptions, setNumOptions] = useState("");
    const [options, setOptions] = useState<string[]>([]);
    const [inviteEmail, setInviteEmail] = useState("");
    const [question, setQuestion] = useState("");

    /* Generates a password 12 characters long */
    const generatePassword = (length = 12) => {
        const characters =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

        let password = "";

        for (let i = 0; i < length; i++) {
            password += characters.charAt(
                Math.floor(Math.random() * characters.length)
                );
        }

        return password;
    };

    /* updates UI when number of options changes */
    const handleNumOptionsChange = (value: string) => {

        setNumOptions(value);

        const number = Number(value);
        setOptions(Array(number).fill(""));
    };

    /* updates the list of options when admin changes one */
    const handleOptionChange = (index: number, value: string) => {
        const newOptions = [...options];
        newOptions[index] = value;
        setOptions(newOptions);
    };

    const inviteGuest = async () => {
        const userRef = doc(db, "userPasswords", inviteEmail);
        const userSnap = await getDoc(userRef);

        if (userSnap.exists()) {

            const oldPassword = userSnap.data().password;

            Alert.alert(
                "User Already Created - save these credentials",
                `Email: ${inviteEmail}\nPassword: ${oldPassword}`,
                [{ text: "OK" }]
            );

            return;
        }

        const invitePassword = generatePassword();

        try {

            await createUserWithEmailAndPassword(
                auth,
                inviteEmail,
                invitePassword
            );

            await setDoc(doc(db, "userPasswords", inviteEmail), {
                email: inviteEmail,
                password: invitePassword,
            });

            await signInWithEmailAndPassword(
                auth,
                adminEmail,
                adminPassword
            );
            console.log("User created and admin signed back in.");

            Alert.alert(
                "User Created - save these credentials",
                `Email: ${inviteEmail}\nPassword: ${invitePassword}`,
                [{ text: "OK" }]
            );

        } catch (error) {
            console.log(error);
        }
    }

    const setPollContent = async () => {
        if (!question || options.length === 0) {
            Alert.alert("Error", "Please enter a question and at least one option.");
            return;
        } 

        try {
            const resultsSnap = await getDocs(collection(db, "Results"));

            await Promise.all(
            resultsSnap.docs.map((resultDoc) =>
                deleteDoc(resultDoc.ref)
                ));

            await setDoc(doc(db, "polls", "currentPoll"), {
                question: question,
                options: options,
                });

            Alert.alert("Success", "Poll saved!");
            Alert.alert("Warning", "This will overwrite any previous polls submitted.");

            setOptions([]);
            setNumOptions("");
            setQuestion("");

        } catch (error) {
        console.error(error);
        Alert.alert("Error", "Could not save poll.");
        }
    }
    
    return (
        <View style={styles.container}>
        <Text style={styles.title}>Admin Panel</Text>

        <Text style={styles.sectionTitle}>Poll</Text>

        <TextInput
            style={styles.input}
            placeholder="Question"
            value={question}
            onChangeText={setQuestion}
        />

        <TextInput
            style={styles.input}
            placeholder="Number of options"
            keyboardType="numeric"
            value={numOptions}
            onChangeText={handleNumOptionsChange}
        />

        {options.map((option, index) => (
            <TextInput
            key={index}
            style={styles.input}
            placeholder={`Option ${index + 1}`}
            value={option}
            onChangeText={(value) => handleOptionChange(index, value)}
            />
        ))}

        <View style={styles.buttonRow}>
            <Button
            title="Set Poll Content"
            onPress={setPollContent}
            />
            <Button
            title = "Results"
            onPress={() => router.replace("/results")}
            />
        </View>
        

        <Text style={styles.sectionTitle}>Guest Management</Text>

        <TextInput
            style={styles.input}
            placeholder="Guest email"
            keyboardType="email-address"
            value={inviteEmail}
            onChangeText={setInviteEmail}
            autoCapitalize="none"
        />

        <View style={styles.buttonRow}>
            <Button title="Invite Guest" onPress={inviteGuest} />

            <Button title="Remove Guest"/>
        </View>
        </View>
    );
    }

    const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
    },
    title: {
        fontSize: 28,
        fontWeight: "bold",
        textAlign: "center",
        marginBottom: 30,
    },
    sectionTitle: {
        fontSize: 20,
        fontWeight: "bold",
        marginBottom: 10,
        marginTop: 20,
    },
    input: {
        borderWidth: 1,
        borderColor: "#aaa",
        borderRadius: 5,
        padding: 10,
        marginBottom: 15,
    },
    buttonRow: {
        flexDirection: "row",
        justifyContent: "space-around",
        marginTop: 5,
    },
    });
```
{: file="admin.tsx"}

**NOT all the imports are being used**. However, they will all be a part of the final product, so keep them for now and they will serve as hints for what you need to add.

> You need to add **results.tsx** to your project! This will be where we create a visualization of the results later on in the project
{: .prompt-warning }

### Walking through the code

Let's take a look at the key functional components of admin.tsx:

First, there are more helper functions in this file than we've had before. There are 3 small helper functions that are self-explanatory.

One of those three helper functions could use the following code to make sure that the admin can't choose an unreasonable amount of options for the poll:

```tsx
    if (parseInt(value) > maxOptions){
      value = maxOptions.toString();
    }
```
{: file="admin.tsx"}

You should be able to find where it goes and understand what it's doing if you look through the helper functions.

The last two helper functions are the most important for the screen! These functions are:

- inviteGuest
- setPollContent

We'll go through each of these in detail next

### inviteGuest

The code to invite a guest is similar to the code that we used to create our admin user. We ultimately want to use createUserWithEmailAndPassword() from firebase auth to add another user.

One important thing to note is that we need to check and see if that email has already been invited. We access the user data from the database (if it exists) and then based on whether it exists we make a decision whether to create a new user or not. 

> When you use createUserWithEmailAndPassword() it automatically signs in the new user. We want the admin to stay signed in. Pay attention to how this is done
{: .prompt-warning }

After the new user is made we want to show the admin their credentials so that they can copy them and distribute them to another user. We won't be doing this, but a good practice to expand upon this would be to give the admin an option to send emails with credentials included or another way to distribute login information.

### setPollContent

When we make a new poll we want to do two things:

(1) Reset any results from a previous poll that was made

(2) Create a new poll in the database so that it can be accessed when a guest user logs in

The most important thing to pay attention to is how to add and remove things from the database. You'll be needing to do this on your own in the coming sections.

**Once you feel confident adding and removing from the database you are ready to write some code!**

### Removing Users

An admin may want to remove a user from the poll. Perhaps they typed in the email wrong or they are changing the poll and who it pertains to. We already have the button to delete a user in the admin screen, but we need to get it working!

First, the following code is going to go in **login.tsx** before we log in the admin (you should be able to find where this is).

```tsx
    if (user.email !== adminEmail) {

        const removeRef = doc(db, "toRemove", email);
        const removeSnap = await getDoc(removeRef);

        if (removeSnap.exists()) {
            await deleteDoc(doc(db, "Results", user.uid));
            await deleteDoc(doc(db, "userPasswords", email));
            await deleteUser(user);
            await deleteDoc(removeRef);         

            setError("This account has been removed.");
            return;
        }

        const resultsRef = doc(db, "Results", user.uid);
        const resultsSnap = await getDoc(resultsRef);

        if (resultsSnap.exists()) {
            router.replace("/completed");
            return;
        }

        router.replace("/poll");
        return;
    }
```
{: file="login.tsx"}

You need to include the following imports: **doc, getDoc, deleteDoc** from firebase/firestore, **db** from our config file, and **deleteUser** from firebase/auth

This code deletes a user and all related fields if they have been added to a "toRemove" doc by the admin.

**Your job is to add the email to the "toRemove" document when the admin uses the deleteUser button!**
- I would recommend also giving the admin user an alert that lets them know the user has successfully been marked for removal.

> Don't forget to add an onPress statement! And pay attention to the name of the document!
{: .prompt-warning }

Before this works you will also need to create a completed.tsx and poll.tsx files. We will be using these later. The completed screen is redirected to if the user has already completed the poll, and poll.tsx is routed to if the user still needs to take the poll.