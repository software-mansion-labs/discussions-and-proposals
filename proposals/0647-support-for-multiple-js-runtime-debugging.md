---
title: Support for multiple Java Script runtime debugging
author:
- Krzysztof Piaskowy @piaskowyk
- Tomasz Zawadzki @tomekzaw
- Krzysztof Magiera @kmagiera
- Karol Wąsowski @Kwasow
date: 15-05-2023
---

# RFC0647: Support for multiple Java Script runtime debugging

## Summary

Obecna implementacja `metro-inspector-proxy` zakłada istnienie tylko jednego runtime w aplikacji. W przypadku jeśli zewnętrzna biblioteka wtorzy własny runtime, np Reanimated, nie ma mozliwości debugowania takiego runtime. Modyfikujac kod metro mozna wspierac dowolną ilość runtime w aplikacji.

## Basic example

Reaniamted library creates own runtime, and with current implementation of `metro-inspector-proxy` this runtime is unable to debugging via Chrome DevTools.

## Motivation

Niektóre biblitek, np Reanimated tworzą własne Runtime, a przy obecnej implementacji `metro inspector proxy` nie ma możliwości debugowania tych runtimów. Chciałbym, aby `metro inspector proxy` umożliwiało debugowanie dowolnego runtime istniejącego w aplikacji React Native. To polepszy developer experience, poniewaz bedzie mozna debugowac kod jsa wykonywany na dowlonym runtime. Similar issues may be encountered by brownfield apps, which can have multiple JS runtimes

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody familiar with React Native to understand, and for somebody familiar with the implementation to implement. This should get into specifics and corner-cases, and include examples of how the feature is used. Any new terminology should be defined here.

I've updated the implementation of Device.js in the metro-inspector-proxy package for it to be more general. Previously it would only create a “virtual” runtime for the Hermes React Native runtime. Now it uses a map and dynamically creates a reloadable “virtual” runtime for every registered runtime and then handles reloads for all of them.

W obecnej implementacji `metro inspector proxy` informacje o istniejącym runtime są przechowywane w sktruktorzye danych nazywaną page. Implementacja natomiast zaklada istnienie tylko jednej page w aplikacji. Rozwiązaniem tego problemu jest uzycie listy stron i umozliwienie aplikacji zarejestrowanie więcej niz jednego runtime. Hermes runtime podczas tworzenia, jeśli wywoła się na nim enableDebugging probuje wyslac przez webSocketa informacje o strzoweniu nowej strony. natomias metro inspector akceptował tylko runtime react native - identyfikowal go przez nazwę runtime podaną jako string. Usunęliśmy to ograniczenie, i tym samym pozwoliło to nam na zarejestrowanie dowolnego runtime.

@kwasow przygotował już PRa który implementuje tę funkcjonalność: https://github.com/facebook/metro/pull/864

## Adoption strategy

This feature implementation doesn't require any additional steps by aplication or library developers. It is fullcy backward compatible. Uzywamy w reaniamted customowego patcha do metro juz od roku i nie napotkaliśmy się na żadne problemy, ooraz nie wymagało to od nas ządnych doatkowych kroków. 