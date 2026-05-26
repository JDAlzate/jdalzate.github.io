---
title: Ability Tasks
permalink: /docs/gas/ability_tasks/

toc: true
toc_sticky: true
sidebar:
  nav: gas_docs
---
Ability Tasks (ATs) are pretty well described in the `AbilityTask.h` header file. It says:

"Ability Tasks are small, self contained operations that can be performed while executing an ability. They are latent/asynchronous is nature. They will generally follow the pattern of 'start something and wait until it is finished or interrupted'."

You can have as many active ability tasks as you like. They will be running "asynchronously" (not necessarily in a multi-threaded way, but rather they run across multiple frames) until they end, firing relevant delegates as they progress.

Ability tasks are shown in Blueprints as async nodes, generally with more than one execution pin. These pins are the representation of the delegates defined in C++.

## 7.1 Common Ability Tasks
There are a lot of ability tasks that you can use out of the box when using GAS. These have already been coded by Epic and cover the most common requirements.

Here's an list of the ones that you will probably use the most.

### 7.1.1 AbilityTask_PlayMontageAndWait
This task is similar to the PlayMontage async node that is widely used in Unreal, but as an ability task it is engrained into the GAS framework. Most importantly, **it is replicated!**

It starts playing a montage on the avatar actor's skeletal mesh component (an example of the importance of defining owner actor vs. avatar actor). It has a few execution pins (delegates in C++) to react to events related to the anim montage. Here's a breakdown:

- **On Completed**: fired when the montage is completely done (i.e., not being an influence on the Skeletal Mesh at all).
- **On Blended In**: fired when the montage has finished its blend-in process. If the blend-in time is set to two seconds, then this delegate will fire two seconds after execution.
- **On Blend Out**: fired when the montage has started its blend-out process.
- **On Interrupted**: PENDING
- **On Cancelled**: PENDING

For example, the first ability in the course is the Fire Bolt ability. We want Aura to play a montage when casting the fire bolt, so we use this node to play the montage, and only end the ability when the montage is completed.

![Example](/docs/gas/Resources/play-montage-and-wait-fire-bolt.png)

### 7.1.2 AbilityTask_WaitGameplayEvent

## 7.2 Creating a Custom Ability Task