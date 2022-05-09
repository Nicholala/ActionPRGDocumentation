# ActionRPG文档

[TOC]

## 1.  关于ActionRPG

### 1.1  ActionRPG简介

### 1.2  ActionRPG的安装方法

（1）点击Epic启动程序中的 **学习** 标签页，向下滚动到 **游戏** 部分。

（2）点击 **Action RPG** 的图片，查看项目描述。点击黄色的 **免费** 按钮。

（3）短暂加载后，按钮将显示 **创建项目**。点击这个按钮，启动程序会提示你选择项目名称和保存位置。

（4）点击 **创建**，之后项目会下载到你指定的目录中。

### 1.3 ActionRPG的学习途径

### 1.4 关于本文档

本文档地址：https://github.com/Nicholala/ActionPRGDocumentation

## 2. UE4 GamePlay架构

在学习ActionRPG之前，我们需要了解UE4 的GamePlay架构，这样会有助于我们理解ActionPRG中的各个蓝图具体实现的是什么功能，以及为什么要用这种蓝图实现这个功能。

### 2.1 UObject

### 2.2 Actor

### 2.3 Component

## 3.  GAS系统

GAS是一个复杂的系统，本章仅仅是对GAS的一些介绍，想深入了解的同学可以前往https://github.com/tranek/GASDocumentation进行学习。

### 3.1 什么是GAS

Gameplay Ability System(GAS)是一个高度灵活的框架，可用于构建你可能会在RPG或MOBA游戏中看到的技能和属性类型。你可以构建可供游戏中的角色使用的动作或被动技能，使这些动作导致各种属性累积或损耗的状态效果，实现约束这些动作使用的“冷却”计时器或资源消耗，更改技能等级及每个技能等级的技能效果，激活粒子或音效，等等。此系统可帮助你在任何现代RPG或MOBA游戏中设计、实现及高效关联各种游戏中的技能，既包括跳跃等简单技能，也包括你喜欢的角色的复杂技能集。

以上是官方文档对GAS的介绍，简单来说GAS是Epic官方提供的一款UE4插件，是用于让程序员和策划设计玩法的端到端系统。

### 3.2 GAS的基本构成

#### 3.2.1  Ability System Component(ASC)

##### 3.2.1.1 ASC的定义

ASC是负责管理一切GAS相关交互的组件([UAbilitySystemComponent](https://docs.unrealengine.com/4.27/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/))，是GAS的中枢。任何要使用Gameplay Ability,有Attributes，或者接收Gameplay Effects的Actor必须包含一个ASC。

##### 3.2.1.2 ASC的作用

在了解ASC的作用之前，我们先来看看在ASC源码中，虚幻官方对ASC的描述

```c++
/** 
 *	UAbilitySystemComponent	
 *
 *	A component to easily interface with the 3 aspects of the AbilitySystem:
 *	
 *	GameplayAbilities:
 *		-Provides a way to give/assign abilities that can be used (by a player or AI for example)
 *		-Provides management of instanced abilities (something must hold onto them)
 *		-Provides replication functionality
 *			-Ability state must always be replicated on the UGameplayAbility itself, but UAbilitySystemComponent provides RPC replication
 *			for the actual activation of abilities
 *			
 *	GameplayEffects:
 *		-Provides an FActiveGameplayEffectsContainer for holding active GameplayEffects
 *		-Provides methods for applying GameplayEffects to a target or to self
 *		-Provides wrappers for querying information in FActiveGameplayEffectsContainers (duration, magnitude, etc)
 *		-Provides methods for clearing/remove GameplayEffects
 *		
 *	GameplayAttributes
 *		-Provides methods for allocating and initializing attribute sets
 *		-Provides methods for getting AttributeSets
 *  
 */
```

通过上面的介绍，我们可以知道，ASC主要通过三个方面来管理技能，即游戏技能GameplayAbilities、游戏效果GameplayEffects、以及属性GameplayAttributes。

#### 3.2.2 Gameplay Abilities(GA)

##### 3.2.2.1 Gameplay Abilities的定义

Gameplay Abilities(GA)是游戏中Actor的行为或者是技能。比如冲刺、射箭、格挡等。

可以通过C++或者蓝图来实现Abilities。

##### 3.2.2.2 Gameplay Abilities的作用

GA包含了技能的具体逻辑，同时提供了常用的一些属性，如技能的冷却时间，技能等级等。除此之外，也可以绑定输入等。GA也支持同时激活多个Gameplay Abilities如冲刺攻击，或者在攻击的同时恢复生命等，都可以通过该系统实现。

#### 3.2.3 Gameplay Effects

##### 3.2.3.1 Gameplay Effects的定义

Gameplay Effects是GA改变他人的Attributes 和 GameplayTags的途径。

##### 3.2.3.2 Gameplay Effects的分类

`GameplayEffects`有三种持续类型：立即（`Instant`），持续（ `Duration`），和无限（`Infinite`）。

####  3.2.4 Tags

Tags

#### 3.2.5Attributes和AttibuteSet

#### 3.2.6Cues



## 4.  ActionRpg功能实现介绍

本章介绍了ActionRPG项目中各项功能的实现方法，当然，ActionRPG是一个庞大复杂且在不断更新的项目，同时本人对UE4的理解水平还有待提升，因此本章介绍的功能实现可能与ActionRPG真正实现方法有一定出入，

### 4.1 角色与摄像机的基本移动

#### 4.1.1  ARPGCharacterBase

要让角色动起来，首先得有个角色。而ActionRPG中游戏的基类，就是ARPGCharacterBase。ARPGCharacterBase是一个C++类，我们可以看一部分ARPGCharacterBase.h文件中的代码来大致了解这个类的作用。

```c++
/** Base class for Character, Designed to be blueprinted */
UCLASS()
class ACTIONRPG_API ARPGCharacterBase : public ACharacter, public IAbilitySystemInterface, public IGenericTeamAgentInterface
{
	GENERATED_BODY()
public:
	// Constructor and overrides
	ARPGCharacterBase();
    
	/** Returns current health, will be 0 if dead */
	UFUNCTION(BlueprintCallable)
	virtual float GetHealth() const;
    
	/** Returns maximum health, health will never be greater than this */
	UFUNCTION(BlueprintCallable)
	virtual float GetMaxHealth() const;
    
	/** Returns current mana */
	UFUNCTION(BlueprintCallable)
	virtual float GetMana() const;
    
	/** Returns maximum mana, mana will never be greater than this */
	UFUNCTION(BlueprintCallable)
	virtual float GetMaxMana() const;
    
	/** Returns current movement speed */
	UFUNCTION(BlueprintCallable)
	virtual float GetMoveSpeed() const;
    
	/** Returns the character level that is passed to the ability system */
	UFUNCTION(BlueprintCallable)
	virtual int32 GetCharacterLevel() const;
    
	/** Modifies the character level, this may change abilities. Returns true on success */
	UFUNCTION(BlueprintCallable)
	virtual bool SetCharacterLevel(int32 NewLevel);
}
```

我们可以看到这个类中，包含了一些具有普遍性的方法。例如获取血量 GetHealth()，或者获取移动速度GetMoveSpeed()等。ARPGCharacterBase中还包含了许多方法，不过目前我们只需要知道ActionPRG项目中的角色的基类是ARPGCharacterBase就行了。

#### 4.1.2 BP_Character

BP_Character是一个蓝图类，是游戏中玩家与NPC的基类，这个类继承自ARPGCharacterBase。让我们来看看这个类中有哪些Component。

![BP_Character组件](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/BP_Character01.png)

包含了一个**胶囊体组件**，其中包含一个**箭头组件**以及一个**网格体组件**；还包含了一个**角色移动组件**与一个**ASC组件**。其中胶囊体组件是角色的碰撞体，也可以理解为物理体积；箭头组件代表了了角色移动的方向，方便开发时观察；在角色移动组件中，我们可以进行一些相关设置，例如将角色方向旋转至与移动方向一致等。至于ASC组件已经在第三节中介绍过，这里不再赘叙。

#### 4.1.3 BP_PlayerCharacter

现在，我们已经了解了**BP_Character**的是如何创建的，并且知道了它包含了哪些组件。接下来就让我们一起来看看**BP_PlayerCharacter**的一个具体应用**BP_PlayerCharacter**。

![image-20220509163624254](../Images/image-20220509163624254.png)

在BP_Character的基础上，BP_PlayerCharacter添加了一个弹簧臂（SpringArm）并在上面绑定了一个摄像机。

接下来，我们选中网格体组件，并按照下图进行设置。为网格体选择动画以及骨骼。

![image-20220509164113418](../Images/image-20220509164113418.png)

做完这些操作后，我们的角色就算创建好了。

#### 4.1.4 角色动画ABP_Player

#### 4.1.4 BP_PlayerController



#### 4.1.5 GameMode 

#### 4.1.5 角色移动的实现

在上述工作完成之后，我们就可以来具体实现角色的移动和摄像机移动了。

### 4.2 武器相关功能

### 4.3 攻击

## 参考资料