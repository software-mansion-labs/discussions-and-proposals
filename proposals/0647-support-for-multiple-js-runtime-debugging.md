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

The current implementation of metro-inspector-proxy assumes the existence of only one runtime in the application. If an external library creates its own runtime, such as Reanimated, there is no possibility to debug that runtime. By modifying the Metro implementation, any runtimes in the application will be debugable.

## Basic example

The Reanimated library creates its Java Script (Hermes) runtime. With the current implementation of `metro-inspector-proxy`, debugging this runtime via Chrome DevTools isn't possible.

## Motivation

Some libraries, like Reanimated, create their own runtime, and with the current implementation of metro-inspector-proxy, debugging these runtimes isn't possible. This issue can also be encountered in brownfield apps, which might have multiple JavaScript runtimes. This limitation hampers the developer experience, as developers using libraries with custom runtimes can't use a debugger to inspect the code being executed. I would like to see metro-inspector-proxy updated to allow debugging of any JavaScript Hermes runtime within a React Native application.

## Detailed design

In the current implementation of metro-inspector-proxy, information about the existing runtime is stored in a data structure called a page. However, this implementation assumes that there is only one page in the application. The solution to this problem involves using a list of pages to enable the registration of more than one runtime. When a Hermes runtime is created, if enableDebugging is called on it, it tries to send information about the creation of a new page via a WebSocket. Previously, the metro inspector accepted only React Native runtimes - identified by the runtime name provided as a string. By removing this limitation, we have allowed the registration of any runtime.

@kwasow has already prepared a PR that implements this functionality: https://github.com/facebook/metro/pull/864 The PR is waiting for review.

## Adoption strategy

The implementation of this feature doesn't require any additional steps from application or library developers and is fully backward compatible. In Reanimated, we have been using a custom patch for Metro for over a year without encountering any issues, and it hasn't required any extra steps from us.