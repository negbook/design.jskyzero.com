---
layout: post
title: 用于AI的高级随机技术
featured-img: dmc5
mathjax: true
categories: [游戏开发,游戏AI]
pro: true
---

本文会深入讲解游戏AI的若干随机算法，并附带通俗易懂的例子，严谨的模拟试验，和技法实用运用指南。

<!--more-->

# 用于AI的高级随机技术

![](/assets/img/skill/RandomInGameAI/1 (3).JPG)

+ 本文最初为神秘组织编写。希望大家都能深入理解艺术与科学
+ 本文主要讨论游戏AI，或者[人工（キズナ）智障（アイ）](https://www.youtube.com/channel/UC4YaOt1yT-ZeyB0OmxHgolA)
+ 本文最初以为会涉及计算机科学，实际没有，放在这里吓唬人
+ 本文包含概率论，但是不用怕，**<u>作者概率论学得没你好</u>**
+ 本文参考了[Game AI Pro](http://www.gameaipro.com/)的文章，知识是相通的，翻新了文中的试验，添加了与时俱进的例子
+ 作者会从读者的角度揣测可能的疑问，并先手提出解答，如果遗漏可以联系作者
+ 本文使用python语言，但是不要怕，反正正常人都看不懂，天书没有区别
+ 本文例子大量使用神秘组织所推崇的游戏《SEKIRO》，很遗憾例子没能全部使用此游戏


## 随机的意义

> 重复着无数相同的动作，我是为了表演而生，
>
> 而表演……是为了取悦你。——《人偶自白》

![](/assets/img/skill/RandomInGameAI/1 (6).JPG)

总所周知，游戏AI为了良好游戏体验服务。——后边部分的定义基本是句废话，过分感性。不同游戏类型目标体验之间的差距，比人和人的差距还要大。

为此，这里从玩家角度、引入消费时经常使用的“质量”的概念来分析一个游戏的体验

+ 从切片、微观上来说，要讲究质：
  + 体验要达到类型预期及格线，有基本的保真度
  + 体验要结构完整，有始有终
  + 部分好游戏的部分体验会“大大超出预期”
+ 从整体、宏观上来说，要讲究量：
  + 高质量的内容以合适的节奏投放给玩家
  + 投放本身也可以构成一种“涌现”的叙事

于是问题被转移到了高质量内容生产上，除了切实的生产新内容，合理的使用旧内容，比如**<u>利用随机</u>**，也能带来良好的体验。

以动作游戏为例，考虑移动等行为本身的差异性、动作之间不同的衔接、动作释放的概率组成不同的考验等。下面我们给出更加具体的例子：

![](/assets/img/skill/RandomInGameAI/1 (7).JPG)

以怪物设计为例：

+ 从怪物整体上来看
  + 我们可以切实的生产新的故事设定、区域环境、敌人兵种、敌人本身，构建一种有血有肉的敌人
  + 比如：只狼故事初期遇到的苇名兵，回到3年前会遇到山贼、寺庙附近遇到僧兵，游戏后期遇到的使用火药的赤备队等。
  + 我们也可以运用随机的手法，扩充单类敌人的体验，让不同感受的同种敌人交替出现
  + 比如外观上的差异、体型的差异、武器的差异等
+ 从怪物自身上来看
  + 我们需要生产怪物符合怪物性格的整套动作资产
  + 比如休闲状态、攻击状态的移动、转换，特色的攻击动作、“打铁”玩法的格挡动作，其他演出动作等
  + 我们可以用随机的手法，使得单个动作能有更为丰富的体验
  + 比如移动的方向、时间、结束距离，是否衔接移动等等；动作直接的组合，以何等行为逻辑、哪些攻击手法组合呈现给玩家，也大有可为。

![](/assets/img/skill/RandomInGameAI/1 (8).JPG)

提到AI中的随机，反应性和随机性是绕不开的话题。

这两种特性，或者说思考方式，是AI最常见的逻辑组织方式，配合差异性的动作，能够构建出表现丰富的AI体验，下面我们给出一个具体的例子：

![](/assets/img/skill/RandomInGameAI/1 (9).JPG)

> 只狼因为资源格式和数据组织解构大体沿用黑魂，可以用工具进行拆包、反编译得到其具体逻辑配置。
> 展示的数据略去繁琐的处理过程，仅对设计进行讨论。

以只狼中的某只基础小怪为例，它有8种基础行为（动作资源 + 逻辑脚本配置，最终构成基础行为）。

这些行为**<u>以距离为条件划分</u>**：

+ 距离较远：靠近
+ 距离偏远：靠近 + 靠近攻击
+ 距离适中：攻击
+ 距离过近：攻击 + 远离攻击 + 远离

类似的，后面我们还将看到BOSS的AI。


## 随机的类型

> 不重复别人是重要的。
>
> 不重复自己是尤其重要的。——（中国作家）陈祖芬

在了解了随机的目的与基本用法后，本章我们开始讨论具体的随机概率模型。


### 均等分布

![](/assets/img/skill/RandomInGameAI/1 (11).JPG)

均等概率自然是最基础的随机模型。

大部分时候，我们使用的都是离散的均等随机，即便是如图二，也只是把1标准单位拆为了1000份。

![](/assets/img/skill/RandomInGameAI/1 (12).JPG)


以弦一郎的AI为例，在特定距离内，拥有6种行为，按照预期概率进行分布。

值得一说的是：

+ 不同环境的新判别标准
  + 引入了除了距离外的新因素：架势值
  + 就像一般会用的血量、时间等
+ 随时间的差异概率模型（解决长时间的内容疲倦问题）
  + 弦一郎有三个阶段，三个阶段的招式是有差异的
  + 基本上难度是递进的，以至于在第三阶段见到第一阶段的技能会觉得处理得十分得心应手

![](/assets/img/skill/RandomInGameAI/1 (13).JPG)

最后我们对固定概率随机进行一个简单的小结。


### 高斯分布

![](/assets/img/skill/RandomInGameAI/1 (14).JPG)

正态分布本质上是一种对自然的拟合：

+ 图1：
  + 我们进行1000次随机1-6的掷骰子，得到的统计图
+ 图2：
  + 我们每次进行3个1-6的掷骰子，将结果求和，重复1000\*6\*3次，得到的统计图
+ 图3：
  + 我们将3个扩大到10个，次数扩大到1000\*6\*10次，得到的统计图

你会发现，当大量因素都对结果有或大或小影响，最终结果的将会呈现出正态分布。

就像自然界中植物最终的高度，可能受到种子、土壤、水分、空气、光照等等因素的影响。最后综合到一起，就会变成正态分布。

![](/assets/img/skill/RandomInGameAI/1 (15).JPG)

很遗憾，这里本来是想找一些**<u>使用正态分布产生不同体型的怪物</u>**的例子的，但是作者失败了。

在只狼的樱龙BOSS战的前期阶段，需要杀大量的“小龙人”，很可惜，小龙人长得一样高。甚至有种诡异的美感

在黑暗之魂3的幽邃主教群BOSS战中，需要杀大量的主教，很可惜，主教做了几种体型，前后参差，但是相同体型一样高。

不得不再说一次，随机只是一种做法，采用不采用都可以，满足目标体验需求即可。


![](/assets/img/skill/RandomInGameAI/1 (16).JPG)

另外一边，怪物猎人：世界，中的大小金可以说是一个极佳的例子。

本身同一只怪物狩猎时间较长，又需要反复狩猎，于是引入了差异体型和对差异体型追求的大小金玩法。

虽然无从论证到底是不是正态分布（刷大小金的过程中只要不是目标体型就重置了OTZ），但是呈正态分布大致也是没问题的。

![](/assets/img/skill/RandomInGameAI/1 (17).JPG)

自然而然地，想到了环境生物的差异体型。

怪物猎人：世界中会有普通或稀有的环境生物，也有各自的体型，也有一部分玩家会去追求这些。比如图上的摇曳鳗女王，只在夜晚的陆珊瑚地图出现（虽然大概是王溟波任务100%刷出），想要刷出来还是要费些功夫的。

那么只狼中也是有杀锦鲤、收集鳞片的，理论上给鱼做一些随机体型，也是不错的，但是我们发现只狼中把体型这件事做得更为夸张，甚至还有剧情和选择，最后结局十分具有冲击性。

这其实还是定制内容或者填充内容的选择问题，两者如上文所言，各有优劣。


![](/assets/img/skill/RandomInGameAI/1 (18).JPG)

那么对正态分布做一个总结，如上图。

需要一说的是，LOD级别，指的是不会被玩家注意到，去留意其行踪的填充级别（否则会穿帮！）

右边我们以子弹命中为例子来更加具体的对比一下随机分布与正态分布：

+ 随机分布永远都在指定范围内，而正态分布可能超出屏幕（三个标准差）外
+ 随机分布均匀分布，而正态分布从中间到四周由密集到稀疏


![](/assets/img/skill/RandomInGameAI/1 (19).JPG)

这里也把生产过程做一个呈现，留意最终的随机生成方法：

+ 实现了个画三个圆和指定数据点的绘制函数
+ 实现了个极坐标系转平面直角坐标系的绘制函数
+ 实现了以指定方法生成指定个数，并绘制出来的生成函数

接下来我们指定生成方法和个数：

+ 随机分布：
  + 随机角度0-360度
  + 随机距离0-最大值
+ 正态分布：
  + 随机角度0-360度（没错！角度仍然是均等随机！）
  + 正态分布距离，最大值为三个标准差

就可以得到对应的分布图。


### 柏林噪声

![](/assets/img/skill/RandomInGameAI/1 (20).JPG)

到这里开始内容就逐渐“玄幻”起来，理想噪声的标准听起来，大概像是是：变化与方向无关；没有剧变；变化与距离无关（如果不对也很正常，门外汉的理解）。

下方的折线图可以看出，柏林噪声或者柏林随机最重要的特性：连续！前面的随机都是离散的，会突变的。


![](/assets/img/skill/RandomInGameAI/1 (21).JPG)

这里给出两个可能的使用场景，就如之前的鱼一样，这里只是可以这么做，实际需不需要做，需要根据需求判断。

只狼中有需要在白蛇神眼皮底下开溜的流程，恶魔之魂中的龙神boss战也是类似的玩法，这里两者都是默认固定周期摆动，实际上头部的旋转就可以使用柏林噪声的连续随机，当流程时间被拉得更长，做固定的几种动作已经没法满足，或者需要做大量随机待机的表现，就可以使用随机来“填充内容”。


![](/assets/img/skill/RandomInGameAI/1 (22).JPG)

总结一下柏林噪声的随机，最大的特征就是连续而不是离散，可以用在各种连续情况，右边介绍了一些基础的控制方式，这里不进行过多展开，有需要时再查阅即可。

现在柏林噪声更多的用在随机地图生成等场合，同样也需求连贯的随机。


## 虚伪的随机


> 正确的结果，是从大量错误中得出来的；
>
> 没有大量错误作台阶，也就登不上最后正确结果的高座。——钱学森


### 随机值筛选

![](/assets/img/skill/RandomInGameAI/1 (24).JPG)

想要说明白计算机的伪随机、或者自然环境下的真随机，有时并不是我们想要的随机这一点，有些复杂，会比较类似复杂的逻辑下特定情况出现的“人工愚蠢”的现象，这里作者用一个亲身经历的例子来说明这一点。

某天晚上，作者在看所在项目的制作人体验作者配置的AI，好巧不巧，中间的AI机缘巧合连续闪避了多次，于是制作人提出致命问题：为什么它闪避了这么多次？这合理吗？

当然，这是作者配置的问题，有很多方法可以解决，但是第一反应下，假设这种情况的只有跑开和闪避两种行为，概率五五开，大量试验下是一定会出现这种连续触发闪避的。

作者自然不能跟制作人讲，这是一个统计学问题，你也不能跟你的玩家讲，“你这是个概率问题，我的AI逻辑十分精妙，这只是个巧合！”。

所以我们引入了“筛选”的概念，逻辑十分好懂，发现不合适的，就打回去，重新随。


![](/assets/img/skill/RandomInGameAI/1 (25).JPG)

比较复杂的地方在于，针对各种随机数据类型或者随机模型，如何判断是否不合适，上图给出了一些判断条件的例子。

大概的通用规律是：

+ 不可以重复出现、间隔出现
+ 不可以同趋势出现、连续将近
+ 统计学意义上大体随机


![](/assets/img/skill/RandomInGameAI/1 (26).JPG)

当然，规则也不是越复杂越好的，过于复杂可能导致随机本身失去意义。

可以使用一些工具检验你的检查条件是否会破坏随机完整性，已经有比较成熟的方案了，这里不展开叙述。


## 总结与参考


![](/assets/img/skill/RandomInGameAI/1 (1).JPG)


最后对本文的内容做一个总结：如上图。

还是想再次重申：随机是一种方法，它为了我们的目标体验而服务。

若干参考，如下：

+ [文中模拟试验的源码](https://github.com/jskyzero/GameAI.Random/blob/master/GameAI.Random.ipynb)，[网页版](https://jskyzero.github.io/GameAI.Random/)
+ [Advanced Randomness Techniques for Game AI: Gaussian Randomness, Filtered Randomness, and Perlin Noise, Steve Rabin, Fernando Silva, Jay Goldblatt](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter03_Advanced_Randomness_Techniques_for_Game_AI.pdf)
