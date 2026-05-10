---
title: What is the Gameplay Ability System
permalink: /docs/gas/what_is_gas/
toc: true
toc_sticky: true
sidebar:
  nav: gas_docs
---
One of the definitions that Epic has provided for the Gameplay Ability System is the following:

"The Gameplay Ability System (GAS) is a highly flexible framework for building the types of abilities and attributes that you might find in an RPG or MOBA title. It can help you design, implement, and efficiently network in-game abilities from as simple as jumping to as complex as your favorite character's ability in any modern title."

At a glance, the Gameplay Ability System has these main components:
1. **Attributes**: Floating-point values that represent a character's stats. They can be read and modified by other GAS systems. E.g., health, max health, mana, stamina, etc.
2. **Gameplay Abilities**: Actions that a character can perform. They define the logic and conditions for something _you do_. E.g., jump, cast fireball, cast shield, etc.
3. **Gameplay Effects**: Data-driven containers that modify attributes or state, either temporarily or permanently. They represent something that _affects you_ or other actors.
4. **Gameplay Cues**: Visual and audio responses tied to gameplay events. They are the tangible sensory representations of what is happening in the game. E.g., an explosion VFX, a fire sound effect, etc.

The Gameplay Ability System also makes extensive use of **Gameplay Tags**, which are not exclusive to GAS, but rather a general-purpose UE system.

As you might imagine, GAS is very complex and goes well beyond these main systems, but everything else exists to enable or extend them.

## 1.1 How do these components work together in practice?
A player character might have a Cast Fireball _ability_. The player can activate this ability, but only if their mana _attribute_ has a high enough value, which spawns a projectile. Spawning the projectile consumes mana. When the projectile hits an enemy, it will apply a _gameplay effect_ to it, reducing its health attribute. At the same time, the effect will trigger a gameplay cue so that a fire sound and a fire effect are triggered at the point of impact.