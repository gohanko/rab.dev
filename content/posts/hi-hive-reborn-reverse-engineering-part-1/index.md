---
title: "Hi-Hive Reborn: Reverse Engineering the Application (Part 1)"
date: 2024-09-09
tags: ['react-native', 'android', 'javascript', 'reverse-engineering']
---

During COVID-19, my university debuted an attendance tracking called Hi-Hive Community. A lecturer would present a QR code, and the students will scan it to take attendance.

After using the application for a few years, I noticed that it is too bloated with unnecessary features, it's not very responsive, and important items are buried in unending levels of menus.

Thus, I have the idea of rebuilding the application using a Jetpack Compose, and more modern UI frameworks.

To do so, I first needed to figure how the API works.

# Preliminary Analysis
Through regular usage, I suspected that it uses some kind of web/mobile hybrid framework. 

Some symptoms I've noticed are:
1. **Slow and slightly off user interface** - There will be slight delays when tapping on elements, animation is slow, and low frame rates at times.
2. **Lack of uniformity in the components** - A lot of the components seems custom made. There are a mix of native elements with custom ones.
3. **Existence of web-view like pages** - Some of the functions are loaded through webview.

All in all it _feels_ like using a website. My gut feelings tell me that it's a web/mobile hybrid framework. Most likely using React Native.

# Dissecting the Application
The only method to really confirm this is by cracking it open and have a look at the internals. Here, we will be using version `2.3.1` of the applicaiton downloaded from [ApkPure](https://apkpure.com/hi-hive-community/com.slc.hihive.community).

## Unpacking the `.apk`
I unpacked the .apk using `jadx` and immediately there are telltale signs of React Native.

For example, this image below shows configuration for a react native push notification library.

![Image of configuration for a React Native library](./images/reactnative_proof.png "Image of configuration for a React Native library")

Which upon further investigation, it appears that this library that was used: https://github.com/zo0r/react-native-push-notification, which is a React Native library.

## Checking for presence of `index.android.bundle`

Furthermore, there is also presence of `index.android.bundle` as seen below.

![Image of index.android.bundle in file structure](./images/reactnative_bundle_proof.png "Image of index.android.bundle in file structure")

The `index.android.bundle` contains compiled and bundled JavaScript code the application.

## Disassembling Hermes Bytecode
I tried to open the bundle using a text editor but it showed an error. 

![Image of cannot open bundle](./images/cannot_open_bundle.png "Image of cannot open bundle")

After a google search, I found out that Hermes VM is the default compilation target for more recent versions of React Native. Which is confirmed by the `file` command.

![Image of file tool showing result](./images/file_command_result.png "Image of file tool showing result")

Fortunately, [P1 Security](https://www.p1sec.com/blog/releasing-hermes-dec-an-open-source-disassembler-and-decompiler-for-the-react-native-hermes-bytecode) released a tool to disassemble, and decompile Hermes bytecodes.

Using the tool, [hermes-dec](https://github.com/P1sec/hermes-dec/), I disassembled the Hermes Bytecode into somewhat readable bytecode.

![Image of not so readable pseudocode](./images/not_so_readable_pseudocode.png "Image of not so readable pseudocode")

However this is still super unreadable, due to it being converted from `javascript` -> `obfuscated javascript` -> `hermes bytecode` -> `decompiled WASM` -> `disassembled JS`. Which meant that it's a "readable" version of the WASM code.

Ideally, we want it to be `javascript` -> `obfuscated javascript`, which should mean it's more readable.

## Back to the beginning and cracking an older version.
The `hermes-dec` GitHub README file mentioned that React Native only started targetting Hermes VM by default after React Native 0.70.

I thought, _what if the targetting of Hermes VM isn't actually on purpose by the developer?_ So I grabbed version `1.0.2` of the application and cracked it open.

As I thought, the `index.android.bundle` file is only obfuscated Javascript, not Hermes Bytecode!

![Image of mode readable bundle](./images/more_readable_bundle.png "Image of mode readable bundle")

This is a more readable version of the `index.android.bundle` file compared to the one of version `2.3.1` but not by much. But it's still better than nothing.

What this means is that we now have a base to reverse engineer the API used by the application.

# Reversing the API

# Putting together a library

# Conclusion