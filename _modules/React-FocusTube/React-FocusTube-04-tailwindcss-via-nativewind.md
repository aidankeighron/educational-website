---
title: "TailwindCSS via NativeWind"
parent_post: React-FocusTube
module_number: 4
layout: module
media_subpath: /assets/tutorials/focustube
---

## TailwindCSS via NativeWind

This portion is optional because I want this tutorial to focus more on APIs and React Native. However, if you are interested in learning more about TailwindCSS, you can try it out on this project! TailwindCSS has shown to be a very powerful library especially when it comes to AI.

Because we installed **NativeWind** during setup, you can use regular Tailwind classes on your Native components.

TailwindCSS is a different way of doing CSS. There are no ```StyleSheet``` objects; instead, you add classes for each style you want.

For example, in regular React Native, you may make a ```StyleSheet``` and add it like this:

```tsx
import { StyleSheet, View, Text } from 'react-native';

const styles = StyleSheet.create({
   centerView: {
     flex: 1,
     justifyContent: 'center',
     alignItems: 'center'
   }
});

export default function MyComponent() {
  return <View style={styles.centerView}> <Text>Hello World</Text> </View>
}
```

In TailwindCSS, each one of these attributes is its own class that we can add straight to the component. For example:

```tsx
import { View, Text } from 'react-native';

export default function MyComponent() {
  return <View className="flex-1 justify-center items-center"> <Text>Hello!</Text> </View>
}
```

For a deeper dive, I highly suggest Fireship's 100-second video on TailwindCSS.

{% include embed/youtube.html id='mr15Xzb1Ook' %}
