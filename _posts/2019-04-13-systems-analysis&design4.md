﻿---  
layout: post  
title: "系统分析与设计四"  
subtitle: "潘老师的课"  
author: "ChenJF"  
header-img: "img/post-bg-imgs/2-systems-analysis-design.jpg"  
header-mask: 0.4  
catalog: true
tags:  系统分析与设计
---

* TOC
{:toc}

---
## Problems
### 简答题
* 用例的概念
* 用例和场景的关系？什么是主场景或 happy path？
* 用例有哪些形式？
* 对于复杂业务，为什么编制完整用例非常难？
* 什么是用例图？
* 用例图的基本符号与元素？
* 用例图的画法与步骤
* 用例图给利益相关人与开发者的价值有哪些？

### 建模练习题（用例模型）
* 选择2-3个你熟悉的类似业务的在线服务系统（或移动 APP），如定旅馆（携程、去哪儿等）、定电影票、背单词APP等，分别绘制它们用例图。并满足以下要求：
 * 请使用用户的视角，描述用户目标或系统提供的服务
 * 粒度达到子用例级别，并用 include 和 exclude 关联它们
 * 请用色彩标注出你认为创新（区别于竞争对手的）用例或子用例
 * 尽可能识别外部系统和服务
* 然后，回答下列问题：
 * 为什么相似系统的用例图是相似的？
 * 如果是定旅馆业务，请对比 Asg_RH  
 * 用例图，简述如何利用不同时代、不同地区产品的用例图，展现、突出创新业务和技术
 * 如何利用用例图定位创新思路（业务创新、或技术创新、或商业模式创新）在系统中的作用
 * 请使用 SCRUM 方法，选择一个用例图，编制某定旅馆开发的需求（backlog）开发计划表
 * 根据任务4，参考 使用用例点估算软件成本，给出项目用例点的估算

## Answers
### 简答题
#### 用例概念
是软件工程或系统工程中对系统如何反应外界请求的描述，是一种通过用户的使用场景来获取需求的技术。 每个用例提供了一个或多个场景,该场景说明了系统是如何和最终用户或其他系统互动,也就是谁可以用系统做什么,从而获得一个明确的业务目标.
#### 用例和场景的关系？什么是主场景或happy path？
* 关系：用例是场景的集合，场景是一个用例实例。
* happy path：主成功场景也被称为“理想路径”或“典型流程”，它描述了满足涉众关注点的典型成功路径。通常主场景要保持一定连贯性并将所有条件处理都推延至拓展部分。

#### 用例有哪些形式？
* 摘要（Brief）
通常是主要成功场景的简短一段总结。
用于在早期的需求分析中，快速了解主题和范围。可能只需要几分钟来创建。
* 非正式（Casual）
非正式的段落格式。包含多种场景的多个段落。
* 详述（Fully）
所有的步骤和变化都写得很详细，并由支持部分，如先决条件和成功保证。

#### 对于复杂业务，为什么编制完整用例非常难？
复杂的业务涉及到的场景非常多，且场景与场景之间也有各种各样的关联，要编制完整用例不但需要熟悉各种业务场景和流程，还要懂得建模相关的专业知识，如何分离和提炼一个场景的主要元素也是在复杂场景中显得尤为重要和困难。
#### 什么是用例图？
用例图是指由参与者（Actor）、用例（UseCase），边界以及它们之间的关系构成的用于描述系统功能的视图

#### 用例图的基本符号与元素？
基本元素与符号：
 * 人：表示参与者（Actor）。
 * 系统框：框内表示待研究的系统。
 * 椭圆：表示一个用例。
 * 箭头：表示参与者与用例或用例与用例之间的交互关系。包括include 和 exclude。

#### 用例图的画法与步骤
1. 确定用例、系统
2. 识别参与者
3. 识别参与者和用例间的关系
4. 作图（绘制用例）: 
    (1). 使用参与者自身能够理解的名称重命名用例，不要使用与代码有关的名称
    (2). 从主要的事务开始，直到后面较小的交互为止
    (3). 将每个用例放入支持它的系统或主要子系统（忽略只与用户有关的外观或组件）
    (4). 可以在系统边界外绘制用例，表明系统不支持该用例

#### 用例图给利益相关人与开发者的价值有哪些？
1. 每个用例制定了系统提供给客户的有用功能单元，使得客户可以更加清晰地看到系统的用途
2. 对于软件开发者，用例细化了用户的需求，以及软件的使用方式，可以使得软件架构的设计思路更加清晰

### 建模练习
#### 用例图
电影票购买系统：
![电影票购买系统用例图](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SWSAD-in/uc1.png)  
酒店预订系统：
![酒店预订系统用例图](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SWSAD-in/uc2.png)

#### 回答问题
1. 为什么相似系统的用例图是相似的？
相似系统的参与者和用例是相似的，用例之间的关系是相似的，用户的预期功能是相似的。
2. 如果是定旅馆业务，请对比Asg_RH用例图，简述如何利用不同时代、不同地区产品的用例图，展现、突出创新业务和技术
不同时代对定旅馆的要求不同，可以创新的业务是通过推荐算法给用户推荐合适的酒店。
3. 如何利用用例图定位创新思路（业务创新、或技术创新、或商业模式创新）在系统中的作用
应该通过创新点在图中的位置来判断。如果创新位于较高的父级，则作用比较大。如果是子类或者是被包括的关系，则作用相对较小。
4. 请使用 SCRUM 方法，选择一个用例图，编制某定旅馆开发的需求（backlog）开发计划表

|ID	|Name|Imp|Est|How to demo|  
|---|---|------|------|--------|  
|1|find hotel|10|3|find by name or location|  
|2|make reservation|7|4|determine hotel name, room type, time and then pay|  
|3|sign in/out|5|2|register for accounts, login in/out|  
|4|comment hotel|4|2|	comment for a hotel|  

5. 根据任务4，参考 使用用例点估算软件成本，给出项目用例点的估算
简单用例：1 到 3 个事务，权重=5
一般用例：4 到 7 个事务，权重=10
复杂用例：多于 7 个事务，权重=15

|用例|	业务	|计算|	原因|	UC比重|  
|----|-----------|---|-----|--------|
|注册登录|	3|	3	||	简单|
|查找酒店|	3|	3	||	简单|
|预订房间|	10|	7	|支付和数据库，框架|一般|
|评论酒店|	3|	2	||	简单|