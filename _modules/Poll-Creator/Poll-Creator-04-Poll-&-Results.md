---
title: "Collecting the Poll Results"
parent_post: Poll-Creator
module_number: 4
layout: module
---

### Poll.tsx

> Do not continue on to this step until you have completed the previous coding task!
{: .prompt-danger }

Now we will make the guest users able to take the poll that we have made. We will provide the UI for the poll and the code to parse our saved poll into the UI. What you will need to do is upload the results into the database so that they can be visualized by the admin in results.tsx later

```tsx
    import { View, Text, Button, StyleSheet, Pressable, Alert } from "react-native";
    import { useEffect, useState } from "react";
    import { doc, getDoc, setDoc } from "firebase/firestore";
    import { auth, db } from "@/src/config/firebaseConfig";
    import { router } from "expo-router";

    export default function Poll() {
    const [question, setQuestion] = useState("");
    const [options, setOptions] = useState<string[]>([]);
    const [selectedOption, setSelectedOption] = useState<number | null>(null);

    const user = auth.currentUser;

    const uploadResult = async () => {
        if (selectedOption === null) {
            Alert.alert("Error", "Please select an option before submitting.");
            return;
        }

        try {

        if (!user) {
            router.replace("/login");
            return;
        }

        await setDoc(doc(db, "Results", user.uid), {
                        choice: selectedOption, 
        });

        router.replace("/completed");
        } catch (error) {
            Alert.alert("Error", "Failed to submit your response. Please try again.");
        }
    }

    useEffect(() => {
        const loadPoll = async () => {
        try {
            const pollRef = doc(db, "polls", "currentPoll");
            const pollSnap = await getDoc(pollRef);

            if (pollSnap.exists()) {
                const pollData = pollSnap.data();

                setQuestion(pollData.question);
                setOptions(pollData.options);
            }
        } catch (error) {
            Alert.alert("Error", "No poll available at this time");
        }
        };

        loadPoll();
    }, []);

    return (
        <View style={styles.container}>
        <Text style={styles.question}>{question}</Text>

        {options.map((option, index) => (
            <Pressable
            key={index}
            style={styles.option}
            onPress={() => setSelectedOption(index)}
            >
            <View style={styles.circle}>
                {selectedOption === index && <View style={styles.selectedCircle} />}
            </View>

            <Text style={styles.optionText}>{option}</Text>
            </Pressable>
        ))}

        <View style={styles.submitButton}>
            <Button
            title="Submit"
            onPress={uploadResult}
            />
        </View>
        </View>
    );
    }

    const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        justifyContent: "center",
    },
    question: {
        fontSize: 24,
        fontWeight: "bold",
        textAlign: "center",
        marginBottom: 30,
    },
    option: {
        flexDirection: "row",
        alignItems: "center",
        borderWidth: 1,
        borderColor: "#aaa",
        borderRadius: 5,
        padding: 15,
        marginBottom: 15,
    },
    circle: {
        width: 24,
        height: 24,
        borderWidth: 2,
        borderRadius: 12,
        marginRight: 12,
        justifyContent: "center",
        alignItems: "center",
    },
    selectedCircle: {
        width: 12,
        height: 12,
        borderRadius: 6,
        backgroundColor: "black",
    },
    optionText: {
        fontSize: 18,
    },
    submitButton: {
        marginTop: 20,
    },
    });
```
{: file="poll.tsx"}

### Challenge Task: Wrapping up the project.

There are two more tasks that you need to complete to make the application fully functional. First, we need **completed.tsx** to display some sort of congratulations message indicating the user has completed the poll. This task can range anywhere from a minimal UI to a fancy design project. It's entirely up to you!

Second, and more important: **we want the admin to be able to see the results!** Come up with a clever way to visualize the data for the admin. Some things to keep in mind:

- You need to pull each user's data from the database so the admin can see all the results
- Nobody wants to look at a bunch of words - try to make a visualization that's appealing. Possible implementations could be a bar graph or a pie chart
- The admin may want to go back to the admin page so you should include a back button

> We know AI can do these tasks! We want to make sure that you've learned how to access the database and make use of useStates/useEffects. If you get stuck make sure to look back at previous code
{: .prompt-warning }