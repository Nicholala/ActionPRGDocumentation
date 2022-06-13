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

#### 1.4.1 本文档地址

https://github.com/Nicholala/ActionPRGDocumentation

#### 1.4.2 本文档结构

## 2. UE4 GamePlay常用类

在学习ActionRPG之前，我们需要了解UE的一些常用类，这些类也在ActionRPG中起着重要的作用。这样会有助于我们理解ActionPRG中的各个类具体实现的是什么功能。

### 2.1 UObject

在C++中，UObject是所有Object的基类，UObject包含了一些基础且广泛的功能，例如`元数据（UProperty）`、`反射生成`、`GC垃圾回收`、`序列化`、`编辑器可见（UProperty）`等。

### 2.2 Actor 与 Component

#### 2.2.1 Actor

有了UObject之后，Actor便从UObject之上派生出来了。在UObject的基础上，Actor具有了更多的方法包括`Replication`、`Spawn`、`Tick`等。

Actor是UE中最重要的角色之一，组织庞大，最常见的有`StaticMeshActor`, `CameraActor`和 `PlayerStartActor`等。Actor之间还可以互相“嵌套”，拥有相对的“父子”关系。

#### 2.2.2 组件Component

Actor仅具有一些最基本的方法，想让Actor具有各个功能，则需要Component。Component就像Actor的装备，让Actor具有了各种各样的技能。与Actor一样，Component也是继承自UObject。

#### 2.2.3 Actor与Component之间的关系

Actor可以看作Component的容器，引用官方文档的例子：汽车上的车轮、方向盘以及车身和车灯等都可以看作Component，而汽车本身就是 Actor。组件必须绑定在Actor身上，它们无法单独存在。

看到这里，如果你有Unity基础的话，应该会将Actor与Unity中的GameObject联系起来。但其实两者是有一些不同的。Actor并没有像GameObject一样自带Transform。这是因为UE认为Actor并不一定是实例化的事物，也可以是一些概念性的事物，例如游戏规则、游戏模式，它们在游戏中并没有实实在在的三维坐标，但在UE中，这些概念也是Actor。从这里，我们也可以得知，Actor并不只是实例化的对象，也可以是概念性的概念。

现在我们得知了Actor并不天生就是实例化的对象，而决定Actor是否可以实例化的正是Component。除此之外，Actor之间的父子关系其实也是通过Component完成的。

Actor只提供了基本的创建销毁，网络复制，事件触发等一些逻辑性的功能。而其他的功能都是由Component实现的。这也印证了Actor可以看作Component的容器这一说法。

### 2.3 Level 与 World

#### 2.3.1 Level

Level也是继承于UObject。Level相当于Actor的容器,每个Level保存当前所有的Actors。

#### 2.3.2 World

World是一个容器，包含了游戏中的所有关卡。它可以处理关卡流送，还能生成（创建）动态Actor。

#### 2.3.3 Level与World的关系

Level维护了一个Actor容器，World维护了一个Level容器，一方面支持了Level的动态加载，另一方面也允许了团队的实时协作，大家可以同时并行编辑不同的Level。

### 2.4 Pawn 与 Character

#### 2.4.1 Pawn

`Pawn` 是`Actor`的子类，它可以充当游戏中的化身或人物（例如游戏中的角色）。`Pawn`可以由玩家控制，也可以由游戏AI控制并以非玩家角色（NPC）的形式存在于游戏中。在`ActionRPG`中，我们的主角就是由玩家控制的Pawn,而敌人则是由AI控制的`Pawn`。

当`Pawn`被人类玩家或AI玩家控制时，它被视为`已被控制（Possessed）`。相反，当Pawn未被人类玩家或AI玩家控制时，它被视为 `未被控制（Unpossessed）`。

#### 2.4.2 Character

就像正方形也是长方形一样，`Character`可以理解为一种特殊的`Pawn`，即类人式的`Pawn`。默认情况下，它带有一个用于碰撞的胶囊组件和一个角色移动组件。它可以执行类似人类的基本动作，例如跳跃、爬行等。

### 2.5 Controller

如果把`Pawn`比作国际象棋中的棋子，那`Controller`就是棋手。默认情况下，`Controller`和` Pawn `之间是一对一的关系；也就是说，每个`Controller`在某个时间点只能控制一个 `Pawn`。此外，在游戏期间生成的 `Pawn` 不会被`Controller`自动控制。

`Controller`可以大致分为两类，即`PlayerController`与`AIController`。

#### 2.5.1 PlayerController

`PlayerController`包含了Camera管理、对输入的响应、以及与`UPlayer`的关联等模块。这些模块也是让`PlayerController`与`AIController`区分开的地方。

#### 2.5.2 AIController

AIController内添加了许多AI需要的组件，比如用于寻路的`Navigation`、AI组件、让AI完成任务的Task系统等。

### 2.6 GameMode

Tracy Fullerton 与 Chris Swain 在《游戏设计梦工厂》中提出了形式、戏剧与动态元素这一游戏设计框架。即一款游戏应当包含形式元素、戏剧元素与动态元素三种元素。 而其中的形式元素由游戏的交互模式、游戏目标、游戏规则、游戏过程、游戏的资源、游戏边界以及游戏的结局组成。而在以上诸多游戏机制中，UE使用GameMode来描述他们所认为最基础的游戏规则，即：

- 出现的玩家和观众数量，以及允许的玩家和观众最大数量。
- 玩家进入游戏的方式，可包含选择生成地点和其他生成/重生成行为的规则。
- 游戏是否可以暂停，以及如何处理游戏暂停。
- 关卡之间的过渡，包括游戏是否以动画模式开场。

### 2.7 Player

`Player`相当于输入的发起者，而游戏则是输入的接收者。例如双人游戏就有两个`Player`，用以区分输入指令是由哪个玩家发出的。在本地的`Player`，也为`LocalPlayer`。

### 2.8 游戏实例GameInstance

`GameInstance`是在游戏开始时创建的类，是全局唯一的。每次打开游戏时都会重新创建。也就是说，`GameInstance`这个类可以跨关卡存在，它不会因为切换关卡或者切换游戏模式而被销毁，因此`GameInstance`可以用于保存一些特定类型的持久数据，如用户账户信息。

## 3.  GameAbilitySystem(GAS)

`GAS`是一个复杂的系统，本章仅仅是对`GAS`的一些介绍，想深入了解的同学可以前往以下地址进行学习。

文档：https://github.com/tranek/GASDocumentation

中文文档：https://github.com/BillEliot/GASDocumentation_Chinese

### 3.1 什么是GAS

`Gameplay Ability System(GAS)`是一个高度灵活的框架，可用于构建你可能会在RPG或MOBA游戏中看到的技能和属性类型。你可以构建可供游戏中的角色使用的动作或被动技能，使这些动作导致各种属性累积或损耗的状态效果，实现约束这些动作使用的“冷却”计时器或资源消耗，更改技能等级及每个技能等级的技能效果，激活粒子或音效，等等。此系统可帮助你在任何现代RPG或MOBA游戏中设计、实现及高效关联各种游戏中的技能，既包括跳跃等简单技能，也包括你喜欢的角色的复杂技能集。

以上是官方文档对GAS的介绍，简单来说`GAS`是Epic官方提供的一款UE4插件，用于让程序员和策划设计玩法的系统。

### 3.2 GAS的基本构成

在3.1节中，我们提到了`GAS`是一个复杂的系统，包含了许多模块，让我们来看看GAS具体包含了哪些部分，以及这些部分各自的作用是什么。

#### 3.2.1  Ability System Component(ASC)

ASC是负责管理一切GAS相关交互的组件([UAbilitySystemComponent](https://docs.unrealengine.com/4.27/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/))，是GAS的中枢。任何要使用Gameplay Ability,有Attributes，或者接收Gameplay Effects的Actor必须包含一个ASC。

##### 3.2.1.1ASC的作用

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

通过上面的介绍，我们可以知道，ASC主要通过三个方面来管理技能，即`游戏技能GameplayAbilities`、`游戏效果GameplayEffects`、以及`属性GameplayAttributes`。

####  3.2.2游戏标签 Gameplay Tags

`FGameplayTag`是由`GameplayTagManager`注册的形似Parent.Child.Grandchild...的层级Name。我们可以使用这些标签来描述对象的状态，例如为对象添加眩晕Debuff,可以给一个`State.Debuff.Stun`的`GameplayTag`。

`GameplayTags`需要在`DefaultGameplayTag.ini`中提前定义，在`ActionRpg`中的定义如下:

```c++
[/Script/GameplayTags.GameplayTagsSettings]
ImportTagsFromConfig=True
WarnOnInvalidTags=True
FastReplication=False
InvalidTagCharacters="\"\',"
NumBitsForContainerSize=6
NetIndexFirstBitSegment=16
+GameplayTagList=(Tag="Ability.Item",DevComment="")
+GameplayTagList=(Tag="Ability.Melee",DevComment="")
+GameplayTagList=(Tag="Ability.Melee.Close",DevComment="")
+GameplayTagList=(Tag="Ability.Melee.Far",DevComment="")
+GameplayTagList=(Tag="Ability.Ranged",DevComment="")
+GameplayTagList=(Tag="Ability.Skill",DevComment="")
+GameplayTagList=(Tag="Cooldown.Skill",DevComment="")
+GameplayTagList=(Tag="EffectContainer.Default",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Player.Combo.BurstPound",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Player.Combo.ChestKick",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Player.Combo.FrontalAttack",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Player.Combo.GroundPound",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Player.Combo.JumpSlam",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Shared.UseItem",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Shared.UseSkill",DevComment="")
+GameplayTagList=(Tag="Event.Montage.Shared.WeaponHit",DevComment="")
+GameplayTagList=(Tag="Status.DamageImmune",DevComment="")

```

在ActionPRG中，药水Potion的Tag是`Ability.item`，法力的tag则是`Ability.Melee`。

#### 3.2.3属性Attributes

Attributes是由[FGameplayAttributeData](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html)结构体定义的浮点值。

Attribute一般应该只能由GameplayEffect修改, 这样ASC才能预测(Predict)其改变.

在ActionRPG中，这些Attributes定义如下

```c++

```



#### 3.2.4属性集AttibuteSet

AttributeSet用于定义, 保存以及管理对Attribute的修改.

在AttributeSet头文件顶部添加以下宏块会自动为每个Attributes生成getter和setter函数。ActionRPG中的RPGAttributeSet.h文件也是这么做的。

```c++
// Uses macros from AttributeSet.h
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

以生命值Health与最大生命值MaxHealth为例，在ActionRPG中，Attribute是这样定义的

```c++
/** Current Health, when 0 we expect owner to die. Capped by MaxHealth */
	UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing=OnRep_Health)
	FGameplayAttributeData Health;
	ATTRIBUTE_ACCESSORS(URPGAttributeSet, Health)

	/** MaxHealth is its own attribute, since GameplayEffects may modify it */
	UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing=OnRep_MaxHealth)
	FGameplayAttributeData MaxHealth;
	ATTRIBUTE_ACCESSORS(URPGAttributeSet, MaxHealth)
```



#### 3.2.5  游戏技能 Gameplay Abilities(GA)

Gameplay Abilities(GA)是游戏中Actor的行为或者是技能。比如冲刺、射箭、格挡等。

可以通过C++或者蓝图来实现Abilities。

##### 3.2.2.1 Gameplay Abilities的作用

GA包含了技能的具体逻辑，同时提供了常用的一些属性，如技能的冷却时间，技能等级等。除此之外，也可以绑定输入等。GA也支持同时激活多个Gameplay Abilities如冲刺攻击，或者在攻击的同时恢复生命等，都可以通过该系统实现。

#### 3.2.3 游戏效果 Gameplay Effects

[GameplayEffect(GE)](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html)是Ability修改其自身和其他[Attribute](https://github.com/BillEliot/GASDocumentation_Chinese#concepts-a)和[GameplayTag](https://github.com/BillEliot/GASDocumentation_Chinese#concepts-gt)的容器。

##### 3.2.3.1 Gameplay Effects的分类

`GameplayEffects`有三种持续类型：立即（`Instant`），持续（ `Duration`），和无限（`Infinite`）。





#### 

#### 3.2.6Cues

### 3.3 GAS在ActionRPG中的结构



## 4.  ActionRpg 功能实现介绍

本章介绍了ActionRPG项目中各项功能的实现方法，当然，ActionRPG是一个庞大复杂且在不断更新的项目，同时本人对UE4的理解水平还有待提升，因此本章介绍的功能实现可能与ActionRPG真正实现方法有一定出入，

### 4.1 角色的基本移动与摄像机旋转

#### 4.1.1  ARPGCharacterBase

要让角色动起来，首先得有个角色。而ActionRPG中角色的基类，就是ARPGCharacterBase。ARPGCharacterBase是一个C++类，我们可以看一部分ARPGCharacterBase.h文件中的代码来大致了解这个类的作用。

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

**BP_Character**是一个蓝图类，是游戏中玩家与NPC的基类，这个类继承自ARPGCharacterBase。让我们来看看这个类中有哪些Component。

![BP_Character组件](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/BP_Character01.png)

包含了一个**胶囊体组件**，其中包含一个**箭头组件**以及一个**网格体组件**；还包含了一个**角色移动组件**与一个**ASC组件**。其中胶囊体组件是角色的碰撞体，也可以理解为物理体积；箭头组件代表了了角色移动的方向，方便开发时观察；在角色移动组件中，我们可以进行一些相关设置，例如将角色方向旋转至与移动方向一致等。至于ASC组件已经在第三节中介绍过，这里不再赘叙。

#### 4.1.3 BP_PlayerCharacter

现在，我们已经了解了**BP_Character**的是如何创建的，并且知道了它包含了哪些组件。接下来就让我们一起来看看**BP_PlayerCharacter**的一个具体应用**BP_PlayerCharacter**。同样的，我们先来看看这个蓝图具有哪些组件：

![image-BP_PlayerCharacter01](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/BP_PlayerCharacter01.png)

可以看出，在BP_Character的基础上，BP_PlayerCharacter添加了一个弹簧臂（SpringArm）并在上面绑定了一个摄像机。

接下来，我们选中网格体组件，并按照下图进行设置。为网格体选择动画以及骨骼。

![image-BP_PlayerCharacter02](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/BP_PlayerCharacter02.png)

做完这些操作后，我们的角色就算创建好了。

#### 4.1.4 BP_PlayerController

角色创建好后，我们在UE4项目设置的输入中添加以下输入事件：

![InputSettings](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/InputSettings.png)

接下来，我们就可以在 BP_PlayerController的事件图标中使用这些事件，BP_PlayerController中控制角色移动的蓝图如下：

![CharacterMove](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/CharacterMove.png)

我们可以通过添加移动输入（Add Movement Input）方法来实现角色的移动。其中，目标是玩家Pawn,即我们的BP_PlayerCharacter。WorldDirection则是摄像机的方向，我们通过获取摄像机旋转的向前与向右向量来获得。ScaleValue则是前进的值，即为我们刚才设置的MoveForward与MoveRight。这样，角色移动的蓝图就完成了。控制摄像机旋转的蓝图如下：

![CameraRoll](https://github.com/Nicholala/ActionPRGDocumentation/blob/master/Images/CameraRoll.png)

设置好后角色的移动就完成了。

ActionRPG中实际的实现方式会比上述方法更复杂一点，因为其中还包括了一些其他功能，例如判断是否处于自动战斗模式等以及通过触屏方式旋转摄像机等，这些功能会在后续介绍。

### 4.2 添加武器

#### 4.2.1

#### 4.2.2

#### 4.2.3

### 4.3

### 4.3 药水

### 4.4 翻滚

### 4.5 

## 参考资料