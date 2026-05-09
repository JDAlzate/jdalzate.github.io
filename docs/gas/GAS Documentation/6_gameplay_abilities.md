---
title: Gameplay Abilities
permalink: /docs/gas/gameplay_abilities/

toc: true
toc_sticky: true
sidebar:
    nav: gas_docs
---

Gameplay Abilities (GAs) are _actions_ or _skills_ that an actor can perform in the game. It defines what an ability _does_ and _under which conditions_ the ability can happen.

Instead of tightly coupling abilities by defining them as functions inside the actor that wants to execute it, GAs provide a self contained object that define how the ability is executed.

Here are some of the benefits of using Gameplay Abilities instead of custom systems:
1. The are **replicated** and **network-predicted** out of the box.
2. They run "asynchronously" since multiple instances of a gameplay ability can be running simultaneously and multiple ability tasks can be active at any given time.
3. Abilities can handle their own internal states, so they can run for any amount of time without any concern from the actor that executes them.
4. They have a built-in concept of **cost** and **cooldown**, plus many other features that tie directly to other GAS systems.

## 6.1 Giving / Removing Gameplay Abilities
To use gameplay abilities, they need to be _given_ to a particular ASC. _Giving_ is not the same as _activating_ — giving an ability just means that the actor it was given to can activate the ability. If an ASC tries to activate an ability that has not been given to it, it won't work.

Given abilities can also be _removed_ once they are given. If an ability is removed from an ASC, it won't be able to activate the ability anymore.

Abilities are given in the form of **Gameplay Ability Specs** (`FGameplayAbilitySpec`) which, like Gameplay Effect Specs, contain the ability class plus other properties that are defined at runtime, like the Level of the ability.

**Abilities must be given and removed on the server**! Ability specs are replicated to the owning client via the `ActivatableAbilities` property in the ASC.

Giving abilities can be done with the following functions:
- `void GiveAbility(...)`: will give the gameplay ability to the ASC. That's it.
- `void GiveAbilityAndActivateOnce(...)`: will give the gameplay ability to the ASC and activate it immediately. **The implication is that the ability will be _removed_ once it finishes execution**.

Removing an ability can be done via:
- `void ClearAbility(...)`: will remove a previously given ability.
- `void ClearAllAbilities(...)`: will remove _all_ previously given abilities.

## 6.2 Gameplay Ability Settings
The Gameplay Ability class has a bunch of different settings that make abilities flexible. Here's a list of the different settings you can tweak for GAs.

### 6.2.1 Gameplay Ability Tags
Similar to the Asset Tags in Gameplay Effects, GAs also make use of Gameplay Tags. 

The different tag containers it uses are pretty well documented in comments and pretty self explanatory, so no need to list them here. However, keep in mind the following info that might not be obvious:

#### 6.2.1.1 Activation Tags
Tags that refer to _activation_ such as _Activation Owned Tags_ and _Activation Required Tags_ mean tags that the activating ASC has.

#### 6.2.1.2 Source Tags
PENDING

#### 6.2.1.3 Target Tags
PENDING

### 6.2.2 Instancing Policy
This setting defines how an ability is *instanced*. By default abilities are **instanced by execution** and that's the most common way for abilities to be instanced. However, depending on the circumstance, you might want to select the other two options. Here's the full list of options:

1. **Not Instanced**: the CDO is used to run this ability, so it can't store state at all.
2. **Instanced per actor**: each actor gets its own instance of this ability. Supports replication and preserves state between executions. Only one active at a time.
3. **Instanced per execution**: every time an ability of this type is activated, a new instance will be created. The instance will be destroyed once the ability ends. Multiple instances can be activated at once, and no state is saved between executions.

The list as described above is listed in order of least performant to most performant. Of course, that also means that it is listed in order of least versatile to most versatile.

**Warning**: In the most recent versions of UE, the _Not Instanced_ instancing policy is marked as deprecated with the suggestion to use _Instanced Per Actor_ instead. There's no clear indication of why this is.
{: .notice--warning}

### 6.2.3 Net Execution Policy
This answers the question "_who executes this ability?" with a few additional considerations. Here's the list of options:

1. **Local Only**: this ability is only run on the local client. The server won't execute this ability at all.
2. **Local Predicted**: this ability is initially activated on the local client, and then activated on the server. Makes use of prediction, meaning that the server can rollback invalid changes.
3. **Server Only**: this ability will only ever run on the server.
4. **Server Initiated**: this ability is initially activated on the server, and then activated on the owning client.

We say "server" here as if the server can't be the local client, but keep in mind that a "local only" ability will in fact run in the server if the server is also the local client, or if the ASC is server-owned.

### 6.2.4 Cost and Cooldown
- **Cost Gameplay Effect Class**: most abilities have the concept of cost. This effect class determines which attribute is used to execute this ability and how much of it is consumed. It is also used as a means to determine if we can actually execute the ability based on the cost.
- **Cooldown Gameplay Effect Class**: this effect determines how long you have to wait before you can activate the ability again.

### 6.2.5 Ability Triggers
These triggers allow us to execute this ability in response to an event. This can either be an actual gameplay event or a response to a tag being added to the activating ASC.

### 6.2.6 Other Settings (Do Not Use)
There are additional settings for Gameplay Abilities that are generally recommended not to be used, either because they are deprecated or not very useful.

- **Replication Policy**: I assume this has to do with abilities that are instanced per actor. However, apparently Epic recommends NOT to use this as it is "useless."
- **Server Respects Remote Ability Cancellation**: this is checked by default. Stephen recommends this should not be used, as server should always have authority.
- **Replicate Input Directly**: this will send RPCs indiscriminately when input is pressed. We probably we want finer control over it. Epic themselves discourage it.

## 6.3 Gameplay Ability State
Gameplay Abilities define the logic for specific actions that can be done by the player or other actor, and these abilities can be fairly complex — most abilities won't be one-shot events.

For that reason, GA instances have a concept of _state_. In the most general terms, abilities are activated and remain active until they are either cancelled or ended.

### 6.3.1 Activating Abilities
When an ability is activated, it will call its virtual function `void ActivateAbility(...)`. This is the entry point of an ability and it's the place where you should add any logic that needs to run as soon as the ability is activated.

For example, a spell ability might want to play an animation and spawn a projectile here.

#### 6.3.1.1 Activating Abilities Manually
Given abilities can be activated via the `void TryActivateAbility(...)` function. This will perform any necessary validations, and if they're met, then the ability is activated.

To activate abilities manually, you need to be mindful about the ability's _Net Execution Policy_. Server-only and server-initiated abilities can only be activated from the server. Local-only and local-predicted abilities can only be activated from the local client.

#### 6.3.1.2 Activating Abilities with Gameplay Events
PENDING

#### 6.3.1.3 Activating Abilities with Input Binding
PENDING

### 6.3.2 Ending Abilities
PENDING

### 6.3.3 Canceling Abilities
PENDING