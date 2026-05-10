---
title: Attribute Sets
permalink: /docs/gas/attribute_sets/

toc: true
toc_sticky: true
sidebar:
  nav: gas_docs
---
Attribute Sets (AS) store attributes for a given Ability System Component. The way to associate attribute sets with an ASC is simply to create them both as default subobjects on the owner class of the ASC.

```cpp
    // Since these are both being created in the constructor, the ASC will have knowledge about the AS.
    AbilitySystemComponent = CreateDefaultSubobject<UAU_AbilitySystemComponent>(TEXT("AbilitySystemComponent"));
    AttributeSet = CreateDefaultSubobject<UAU_AttributeSet>("AttributeSet");
```

You can have multiple Attribute Sets for a single ASC, but they would need to be different classes with no inheritance (the ASC locates attribute sets by class). You can also just use one monolithic Attribute Set for everything. In [Stephen's GAS Course](https://www.udemy.com/course/unreal-engine-5-gas-top-down-rpg/) we use the monolithic approach, but know that professional projects use either approach.

## 4.1 Creating Attributes
Attribute Sets require quite a bit of boilerplate code to create a single attribute. 

Here's the necessary steps inside you attribute set class:
1. Declare the attribute using `FGameplayAttributeData` in the attribute set class declaration.
2. Mark it as `ReplicatedUsing = ...`
3. Inside the `OnRep` function, use the `GAMEPLAYATTRIBUTE_REPNOTIFY` macro.
4. Inside the `GetLifetimeReplicatedProps` function, make sure to mark the attribute as `REPNOTIFY_Always`.
5. Use the `ATTRIBUTE_ACCESSORS_BASIC` macro to generate getters and setters for the attribute.

Here's how a strength attribute would look for your game:
```cpp

// .h
UCLASS()
class AURA_API UAU_AttributeSet : public UAttributeSet
{
    GENERATED_BODY()

private:
    UPROPERTY(Transient, BlueprintReadOnly, ReplicatedUsing=OnRep_Strength, Category = "Primary Attributes", meta = (AllowPrivateAccess = "true"))
    FGameplayAttributeData Strength;

    UFUNCTION()
    void OnRep_Strength(const FGameplayAttributeData& PreviousStrength);

public:
    ATTRIBUTE_ACCESSORS_BASIC(UAU_AttributeSet, Strength);
}

// .cpp
void UAU_AttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    DOREPLIFETIME_CONDITION_NOTIFY(UAU_AttributeSet, Strength, COND_None, REPNOTIFY_Always)
}

void UAU_AttributeSet::OnRep_Strength(const FGameplayAttributeData& PreviousStrength)
{
    GAMEPLAYATTRIBUTE_REPNOTIFY(UAU_AttributeSet, Strength, PreviousStrength)
}
```

## 4.2 Modifying Attributes
You can modify attributes directly in code. You can do this by using the mutators created by the `ATTRIBUTE_ACCESSORS_BASIC` macro shown in the previous section.

Following the same example from the previous section, you can simply call `SetStrength` to modify the strength attribute directly. 

You should avoid doing this, however. The preferred way to modify attributes is to do so through Gameplay Effects. Effects have many advantages such as prediction, and have a lot more functionality that is skipped by simply setting the value directly.

See the Gameplay Effects section for more information on how they modify attributes.

## 4.3 How Attributes are Stored
Attributes are stored in the attribute set as `FGameplayAttributeData`, which stores two floats:
1. **Base Value**: the permanent value of the attribute.
2. **Current Value**: the value resulting from the base value + any temporary modification.

A common misconception is that the base value can represent a "max value" and the current value is just a fraction of that base value. That is **NOT** a correct way to picture it, because the base value can and should change.

For example, you should not assume that the base value of health is 100 and when the actor takes 10 points of damage, the current value will be 90. That is a permanent change that should be represented by the base value. If you need a "max value", you should have a separate attribute representing it.

## 4.4 Initializing Attributes
There are several different methods to initialize attributes. Some are more flexible than others. Here's the list of different ways, in the order they appear in the course.

### 4.4.1 Direct Initialization
Thanks to the `ATTRIBUTE_ACCESSORS_BASIC` macro, we have getters and setters for each attribute. One of the functions defined for the attribute is the initter. For example, for the `Health` attribute, we will have an `InitHealth()` function.

We can use the initter to initialize the health attribute in the attribute set constructor, for example. This is the most straightforward way, but also the least flexible. We use this for testing purposes in the course, but eventually replace every call to this with another method.

### 4.4.2 Initialization from Data Table
Inside the gameplay ability system, there is an option to specify data tables to initialize attributes to default values by setting the `Default Starting Data` value.

This is not a good way to initialize attributes though, since at the time of writing this article (UE 5.7) the data table's row struct data is marked as being a "work in progress", containing variables that are not usable yet.

It also seems like this method doesn't allow initializing different values based on different levels.

### 4.4.3 Initialization from Gameplay Effect
The most flexible way to initialize attributes in an attribute set is by applying a gameplay effect that overrides the base value of the attribute you want to initialize.

This is a pretty useful way to do this, because you can also specify the Curve Table for levels, in case that's relevant for your game.

For more information of exactly how Gameplay Effects modify attributes, please see the Gameplay Effects page.