---
title: Gameplay Tags
permalink: /docs/gas/gameplay_tags/

toc: true
toc_sticky: true
sidebar:
    nav: gas_docs
---
Gameplay Tags are not exclusive to GAS, but they are essential to this framework and used extensively.

Gameplay tags are basically **FNames on steroids**. They are just plain old names, but they have a whole system built on top of them that make them very flexible as identifiers or as state representation.

For example, if a character is stunned, we could give it a `State.Debuff.Stun` tag for the duration of the stun. That way, we can easily query if an enemy is stunned and react accordingly.

## 3.1 Gameplay Tag Queries
Gameplay Tags are hierarchical names in the form of `Parent.Child.Grandchild`. This is one of their biggest advantages over Strings, Names, and Enums.

What this means is that we can easily classify objects not just by a particular state or tag, but by a particular category of states or tags.

For example, when stunning an enemy we could give it the `State.Debuff.Stun` tag, so that we can easily check if it is stunned by checking if the enemy contains the exact `State.Debuff.Stun`, but we could just as easily check if the enemy has _any kind of_ debuff by checking if the enemy contains any tag that matches `State.Debuff`.

The two main functions to query tags are:
```cpp
// "A.1".MatchesTag("A") will return True, "A".MatchesTag("A.1") will return False.
bool MatchesTag(const FGameplayTag& TagToCheck) const;

// "A.1".MatchesTagExact("A") will return False.
bool MatchesTagExact(const FGameplayTag& TagToCheck) const;
```

## 3.2 Gameplay Tag Containers
An `FGameplayTagContainer` can be perceived as a wrapper for an array of gameplay tags, but it has quite a lot more built into it, so it should always be the preferred way to represent a collection of gameplay tags.

Gameplay Tag Containers have similar query functions to the ones shown in the previous section, but to check if the container has a tag:
```cpp
// {"A.1"}.HasTag("A") will return True, {"A"}.HasTag("A.1") will return False
bool HasTag(const FGameplayTag& TagToCheck) const

// {"A.1"}.HasTagExact("A") will return False
bool HasTagExact(const FGameplayTag& TagToCheck) const
```

## 3.3 Creating Gameplay Tags
There are quite a few ways to create gameplay tags. Here are all the ways that we explore in [Stephen's GAS Course](https://www.udemy.com/course/unreal-engine-5-gas-top-down-rpg/).

### 3.3.1 Creating Gameplay Tags through the editor
The most straightforward way to add new gameplay tags is directly inside the editor by going to **Project Settings > Gameplay Tags**. Here, you'll be able to create new gameplay tags and save them to a .ini file.

Any place where you can select a gameplay tag from a drop-down will also have the option to create a new gameplay tag.

### 3.3.2 Creating Gameplay Tags through Data Tables
This is also a very convenient way to add gameplay tags, which consists in creating data tables that inherit from `GameplayTagTableRow`. These data tables can then be added to the gameplay tag settings in **Project Settings**.

### 3.3.3 Creating Gameplay Tags in C++
Another way to create gameplay tags is natively in C++ by using the following function call:
```cpp
UGameplayTagsManager::Get().AddNativeGameplayTag("ExampleTag", TEXT("Example doc comment"));
```

A good place to add gameplay tags is inside the Asset Manager, overriding the `StartInitialLoading` function. This makes sure that we have access to the tags inside the editor. If you used a lazy initializer instead, you'd only be able to use the tags at runtime inside C++.