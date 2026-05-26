---
title: Gameplay Events
permalink: /docs/gas/target_data/

toc: true
toc_sticky: true
sidebar:
  nav: gas_docs
---
The `FGameplayAbilityTargetData` hierarchy is designed to hold targeting information to be consumed at later points during gameplay. For example, we might want to store an array of target actors, or a hit result, etc.

This targeting data is stored as a handle, using the `FGameplayAbilityTargetDataHandle` structure. This allows more flexibility when passing target data around and, critically, allows you to use different types of target data by supporting polymorphism.

## 9.1 Target Data Polymorphism
The Target Data System is designed with extensibility in mind. That means that you can have Target Data structures with many different types of data, fully supported by the system.

`FGameplayAbilityTargetData` can be subclassed, and it's these subclasses which are used to store the data. GAS already provides three subclasses that cover a wide range of use cases:
- `FGameplayAbilityTargetData_ActorArray`
- `FGameplayAbilityTargetData_SingleTargetHit`
- `FGameplayAbilityTargetData_LocationInfo`

You can store an instance of any of these subclasses, or any additional ones you create, with the `FGameplayAbilityTargetDataHandle` structure. Here's an example:
```cpp
auto* TargetData = new FGameplayAbilityTargetData_SingleTargetHit(HitResult);
FGameplayAbilityTargetDataHandle TargetDataHandle = TargetData;
```

## 9.2 Replicating Target Data in Ability Tasks
(PENDING section for replicating target data without ability tasks)
(alternatively, PENDING updating this section to not heavily rely on ability tasks)

One of the most useful features of the Target Data System is the way it is replicated in GAS. Gameplay abilities can request local target data to be sent to the server, and the ability in the server can then react to receiving the target data.

### 9.2.1 Sending Local Target Data to the Server
To send local data to the server for Gameplay Abilities to react, you need to use the following function in the Ability System Component:

```cpp
void ServerSetReplicatedTargetData(FGameplayAbilitySpecHandle AbilityHandle, FPredictionKey AbilityOriginalPredictionKey, const FGameplayAbilityTargetDataHandle& ReplicatedTargetDataHandle, FGameplayTag ApplicationTag, FPredictionKey CurrentPredictionKey);
```

Here's a breakdown of the parameters:
- **AbilityHandle** and **AbilityOriginalPredictionKey**: these are used to identify which ability is wanting to send the replicated target data to the server. You can use the `GetAbilitySpecHandle()` and `GetActivationPredictionKey()` functions from the Ability Task.
- **ReplicatedTargetDataHandle**: this handle refers to the actual target data. Target data may vary, as explained in the Target Data Polymorphism section, but any target data can be stored inside this same handle class.
- **ApplicationTag**: PENDING.
- **CurrentPredictionKey**: you can use the ASC's `ScopedPredictionKey` for the `CurrentPredictionKey` parameter, but you will need to also add an `FScopedPredictionWindow` inside your scope, to ensure that the Prediction Key is correctly set.

**Note**: we will talk about prediction keys later in the course, I'll probably update this section when we do.

Calling this function inside an Ability Task should look something like this:
```cpp
UAbilitySystemComponent* ASC = AbilitySystemComponent.Get();
FScopedPredictionWindow PredictionWindow(ASC);

auto* TargetData = new FGameplayAbilityTargetData_SingleTargetHit(HitResult);
ASC->ServerSetReplicatedTargetData(GetAbilitySpecHandle(), GetActivationPredictionKey(), TargetData, {}, ASC->ScopedPredictionKey);
```

### 9.2.2 Receiving Remote Data on the Server
We've covered how to send data to the server, but how does the server know when data is received and how can it react to it?

The ASC **stores a map of target data delegates** for each GA instance that uses target data. The server can bind to that delegate, and so **it will be notified when target data is received** and invoke a callback.

```cpp
auto& TargetDataSetDelegate = ASC->AbilityTargetDataSetDelegate(GetAbilitySpecHandle(), GetActivationPredictionKey());
TargetDataSetDelegate.AddUObject(this, &ThisClass::OnTargetDataSetOnServer);
```

However, there is not always clarity about whether the server has or hasn't received the target data at the time it binds to the delegate. **If you bind to the delegate, but the data had already been set, the callback will never be invoked!**

That's why the ability system component also has the function `void CallReplicatedTargetDataDelegatesIfSet`. If the target data has already been set, then this function broadcasts the corresponding target data delegate. If it hasn't been set, then nothing happens. The function returns whether the delegate was broadcasted as a result of being called.

If the delegate isn't manually called by the `CallReplicatedTargetDataDelegatesIfSet`, the it's good practice to notify the GA that this task is currently waiting for remote data. By default, this makes sure that the tas is cancelled if the remote ability is ended, but the logic can be extended, so it's good to make sure to call this function even if you believe you don't need it.

```cpp
const bool bManuallyCalledDelegate = ASC->CallReplicatedTargetDataDelegatesIfSet(GetAbilitySpecHandle(), GetActivationPredictionKey());

if (!bManuallyCalledDelegate)
{
  SetWaitingOnRemotePlayerData();
}
```

Now, it's important that you consume the data once you've used it. Replicated target data is cached until it is consumed, so you need to let GAS know when a specific replicated target data cache can be cleared. Failure to do this will cause `CallReplicatedTargetDataDelegatesIfSet` to always broadcast the delegate when called (because it assumes that the cached data is new data).

To consume replicated target data, you need to call the following function:

```cpp
void UAbilitySystemComponent::ConsumeClientReplicatedTargetData(FGameplayAbilitySPecHandle AbilityHandle, FPredictionKey AbilityOriginalPredictionKey);
```

### 9.2.3 Putting It All Together
The following code shows an example of an ability task that makes use of Target Data to replicate information about the mouse cursor location. When activated, the ran logic depends on whether we are the server or the local client.

The local client will immediately send the data under the cursor to the server, and then broadcast the data successfully.

The server will wait for the data to arrive, consume the data, and broadcast the data successfully.

```cpp
UAU_AT_TargetDataUnderMouse* UAU_AT_TargetDataUnderMouse::CreateTargetDataUnderMouse(UGameplayAbility* OwningAbility)
{
	UAU_AT_TargetDataUnderMouse* NewTask = NewAbilityTask<UAU_AT_TargetDataUnderMouse>(OwningAbility);
	return NewTask;
}

void UAU_AT_TargetDataUnderMouse::Activate()
{
	Super::Activate();

	const FGameplayAbilityActorInfo* ActorInfo = Ability->GetCurrentActorInfo();
	const AAU_PlayerController* PlayerController = ActorInfo ? Cast<AAU_PlayerController>(ActorInfo->PlayerController) : nullptr;

	if (!PlayerController || !AbilitySystemComponent.Get())
	{
		OnFailure();
		return;
	}

	if (PlayerController->IsLocalController())
	{
		FScopedPredictionWindow PredictionWindow(AbilitySystemComponent.Get());

		const FHitResult& CachedCursorTraceResult = PlayerController->GetCachedCursorTraceResult();
		const FGameplayAbilityTargetDataHandle TargetDataHandle = new FGameplayAbilityTargetData_SingleTargetHit(CachedCursorTraceResult);

		AbilitySystemComponent->ServerSetReplicatedTargetData(GetAbilitySpecHandle(), GetActivationPredictionKey(), TargetDataHandle, {}, AbilitySystemComponent->ScopedPredictionKey);
		OnSuccess(TargetDataHandle);
	}
	else
	{
		FAbilityTargetDataSetDelegate& TargetDataSetDelegate = AbilitySystemComponent->AbilityTargetDataSetDelegate(GetAbilitySpecHandle(), GetActivationPredictionKey());
		TargetDataSetDelegate.AddUObject(this, &ThisClass::OnTargetDataSetOnServer);

		const bool bCalledDelegate = AbilitySystemComponent->CallReplicatedTargetDataDelegatesIfSet(GetAbilitySpecHandle(), GetActivationPredictionKey());
		if (!bCalledDelegate)
		{
			SetWaitingOnRemotePlayerData();
		}
	}
}

void UAU_AT_TargetDataUnderMouse::OnTargetDataSetOnServer(const FGameplayAbilityTargetDataHandle& TargetDataHandle, const FGameplayTag ApplicationTag)
{
	AbilitySystemComponent->ConsumeClientReplicatedTargetData(GetAbilitySpecHandle(), GetActivationPredictionKey());
	OnSuccess(TargetDataHandle);
}

void UAU_AT_TargetDataUnderMouse::OnSuccess(const FGameplayAbilityTargetDataHandle& TargetData)
{
	if (ShouldBroadcastAbilityTaskDelegates())
	{
		OnSuccessDelegate.Broadcast(TargetData);
	}

	EndTask();
}

void UAU_AT_TargetDataUnderMouse::OnFailure()
{
	if (ShouldBroadcastAbilityTaskDelegates())
	{
		OnFailureDelegate.Broadcast({});
	}

	EndTask();
}
```