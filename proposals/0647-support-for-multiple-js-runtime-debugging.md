---
title: Support for multiple Java Script runtime debugging
author:
- Karol Wąsowski @Kwasow
- Krzysztof Piaskowy @piaskowyk
- Tomasz Zawadzki @tomekzaw
- Krzysztof Magiera @kmagiera
date: 15-05-2023
---

# RFC0647: Support for multiple Java Script runtime debugging

## Summary

The current implementation of `metro-inspector-proxy` assumes there is only one runtime in the React Native application. In some cases, an application can have more than just one JavaScript Runtime. Debugging these additional runtimes via Chrome DevTools is currently not possible.

## Motivation

Some libraries, like Reanimated, create their runtime, and with the current implementation of `metro-inspector-proxy`, debugging these runtimes is not possible. Multiple JavaScript runtimes might also be encountered in brownfield apps. This limitation of the Metro server has a negative impact on the developer experience, as developers using libraries with custom runtimes can not use Chrome DevTools to inspect the code being executed. We would like to see `metro-inspector-proxy` updated to allow debugging of any JavaScript Hermes runtime that exists in React Native application.

## Detailed design

The `metro-inspector-proxy` stores information about the existing runtime in a data structure called a `page`. However, current implementation can store only one page per the application which is limiting. To allow the registration of more pages one just save them in a list.

When a Hermes runtime is created, and `enableDebugging` is called on it.
```cpp
facebook::hermes::inspector::chrome::enableDebugging(std::move(adapter), "Runtime Name");
```
This method tries to send information about the new page via a WebSocket to the Metor server. Previously, the `metro-inspector-proxy` accepted only React Native JavaScript runtime - identified by the runtime name provided as a string in page. By removing this limitation, we have allowed the registration of any JavaScript Hermes runtime.

@kwasow has already prepared a PR that implements this functionality: https://github.com/facebook/metro/pull/864 
The PR is waiting for review.

<img width="694" alt="Screenshot 2023-05-19 at 14 57 42" src="https://github.com/facebook/metro/assets/36106620/4919959a-55c5-48e1-ab0b-1f72bb04b3ca">

<img width="1445" alt="Screenshot 2023-05-19 at 15 04 32" src="https://github.com/facebook/metro/assets/36106620/594c4d86-9bbe-4a61-86db-4258662ce302">

## Adoption strategy

The implementation of this feature does not require any additional steps from application or library developers as it is fully backward compatible. In Reanimated, we have been using a custom patch for Metro for almost a year without encountering any issues related to thiose changes.