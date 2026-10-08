---
title: "Setup the Project"
parent_post: React-FocusTube
module_number: 1
layout: module
media_subpath: /assets/tutorials/focustube
---

## React vs React Native Patterns

Before we begin coding, it is important to understand the difference between React (for the web) and React Native (for mobile). 

- **No HTML or DOM:** React Native compiles down to actual iOS and Android UI elements, not a web browser. This means you **cannot** use HTML tags like `<div>`, `<h1>`, or `<p>`. 
- **Component Translation:** Instead of HTML, React Native provides its own primitive components that you must import. You will swap web tags for Native tags like this:
  - `<div>` -> `<View>`
  - `<h1>`, `<p>` -> `<Text>`
  - `<input>` -> `<TextInput>`
  - `<button>` -> `<Pressable>` or `<TouchableOpacity>`

## Setup the project

You will be making your own project instead of forking a repository. Making a React Native app with Expo is incredibly easy!

Open a terminal. In the terminal, go to the folder you want your project to be in, then run this command:

```console
npx create-expo-app@latest
```

You will be prompted to name your project. I named mine **my-app**.

Next, you will be prompted to select an Expo SDK version (here's what mine looks like):

Then, you MUST select **For learning with Expo Go (SDK 54)**.

```console
Select an Expo SDK version: » - Use arrow-keys. Return to submit.
    Latest (SDK 57)
    For learning with Expo Go (SDK 54)
    Other SDK version…
```

The setup automatically configures **Expo Router**, which gives us Next.js style file-based routing out of the box!

<span style="text-decoration: underline;" title="type 'cd <file path>' into your terminal to change your working directory">cd</span> into that folder:

```console
cd my-app
```

### Installing NativeWind

Since we want to use standard TailwindCSS classes (like `className="flex flex-col"`) instead of writing complex `StyleSheet` objects, we will install **NativeWind**.

Run this command in your project folder to install NativeWind and its dependencies:

```console
npm install nativewind@4.2.3 tailwindcss@3.4.19 react-native-css-interop@0.2.3
npx expo install react-native-reanimated babel-preset-expo
```

Initialize Tailwind:

```console
npx tailwindcss init
```

This creates a `tailwind.config.js` file. Open it and update the `content` array to include your app's files, and add the nativewind preset:

```javascript
module.exports = {
  content: ["./app/**/*.{js,jsx,ts,tsx}", "./components/**/*.{js,jsx,ts,tsx}"],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Next, NativeWind requires a global CSS file. Create a file named `global.css` in your project root and add this inside:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Now, create a `metro.config.js` file in your root folder and add this code:
```javascript
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

const config = getDefaultConfig(__dirname);
module.exports = withNativeWind(config, { input: "./global.css" });
```

Finally, newer versions of Expo don't always create a Babel config by default. Create a new file named `babel.config.js` in the root of your project folder (if it doesn't already exist) and update it to this:

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      ["babel-preset-expo", { jsxImportSource: "nativewind" }],
      "nativewind/babel"
    ]
  };
};
```
> **QUESTION:** Take a moment to explore what Babel is and how it transforms JavaScript. Understanding Babel will help you grasp the role of the `babel.config.js` file in this tutorial. 
{: .prompt-tip }

To make sure your app loads this styling globally, go into `app/_layout.tsx` and add this import to the very top of the file:
```javascript
import "../global.css";
```

### Start the App

Now, run this command to start your development server:

```console
npm run start
```

This will give you a QR code. You can download the **Expo Go** app on your phone to scan it and view your app live on your physical device, or press `i` to open it in an iOS Simulator if you have a Mac!

> **NOTE:** When you change your code and save, the Expo Go app will automatically refresh the screen for you. You can also enter 'r' into the console. 
{: .prompt-info }

Now you are ready to start the tutorial! Take a look at your file tree. It should look something like this:

```
my-app/
├── app/
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── explore.tsx
│   │   └── index.tsx
│   ├── _layout.tsx
│   └── modal.tsx
├── assets/
│   └── images/
├── components/
│   └── ...
├── constants/
│   └── ...
├── hooks/
│   └── ...
├── scripts/
│   └── ...
├── .gitignore
├── app.json
├── babel.config.js
├── eslint.config.js
├── expo-env.d.ts
├── global.css
├── metro.config.js
├── nativewind-env.d.ts
├── package-lock.json
├── package.json
├── tailwind.config.js
└── tsconfig.json
```

Feel free to check out **app/(tabs)/index.tsx** in your app folder, this is the **home screen** in your Expo app, which is a good transition into the first topic.

> **QUESTION:** There are some new files in our project. What are `package.json` and `app.json` used for in an Expo project?
{: .prompt-tip }
