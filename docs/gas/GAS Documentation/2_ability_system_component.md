---
title: The Ability System Component
permalink: /docs/gas/ability_system_component/

toc: true
toc_sticky: true
sidebar:
	nav: gas_docs
---
GAS uses the **Ability System Component** (ASC) to mark an actor as being _part of_ the Gameplay Ability System framework. Naturally, the most important actor to use an ASC is your player character.

There are two places where the ASC can live for your player character:
1. **On the Pawn class itself**: do this if you want no persistent state for players when they die and respawn.
2. **On the Player State class**: do this if you want to save persistent state for players when they die and respawn.

For enemies, it makes sense to have the ASC on the Pawn class itself, but for player characters you will most likely want to use the player state to store the ASC. See [Init Ability Actor Info](#init-ability-actor-info) for an explanation of the difference.

## 2.1 Replication Mode
When setting up replicated ability system components, there are three replication mode options:
1. **Full**: all gameplay effects are replicated to all clients. This should be used for single player modes, local co-op modes, or any other mode where bandwidth isn't relevant.
2. **Mixed**: gameplay effects are replicated only to the owning connection. This should be used for player-owned actors.
3. **Minimal**: gameplay effects are not replicated at all. Only cues and tags are replicated to all clients. Use for AI-controlled characters or similar server-owned actors.

Choosing the correct replication mode ensures you maintain the functionality you need while saving on performance and bandwidth by not replicating stuff you don't need to replicate.

**Warning**: for an ASC to use Mixed Replication Mode, **the OwnerActor's owner MUST be a player controller**. Otherwise, nothing will replicate.
{: .notice--warning}

Replication mode is set in the constructor, when creating the ASC:
```cpp
AAU_PlayerState::AAU_PlayerState()
{
	AbilitySystemComponent = CreateDefaultSubobject<UAU_AbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Mixed);
}
```

## 2.2 Init Ability Actor Info
An ASC doesn't always affect the actor that owns it. For example, an ASC may be owned by a Player State, but the actual in-game representation of the ASC would be the player character.

For that reason, they have two separate variables:
1. **Owner Actor**: this is the actor that actually created and owns the ability system component.
2. **Avatar Actor**: this is the physical representation of the ASC in the world.

You need to initialize these values as soon as possible during the set up of your GAS-enabled actors. This is achieved by calling `InitAbilityActorInfo`. For our ASC on the Player State, we do this inside the Character class's `PossessedBy` function, setting the Player State as the owner actor, and the Character as the avatar actor.

```cpp
void AAU_PlayerCharacter::PossessedBy(AController* NewController)
{
	Super::PossessedBy(NewController);

    	if (UAU_AbilitySystemComponent* AbilitySystemComponent = GetAuraAbilitySystemComponent(); ensure(AbilitySystemComponent))
	{
		if (AAU_PlayerState* AuraPlayerState = GetPlayerState<AAU_PlayerState>(); ensure(AuraPlayerState))
		{
			AbilitySystemComponent->InitAbilityActorInfo(AuraPlayerState, this);
		}
	}
}
```

Since the component and both variables are replicated, **you only need to call this function in the server**. The course also calls this in the local client, but that's not necessary.

Note that when the ASC is initialized, **both the owner and the avatar actors are automatically set to the component's owner by default**, so there's no need to be explicit about setting both to the owner if that's the case.