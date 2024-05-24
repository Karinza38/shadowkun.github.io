---
layout: post
title: llmTemplateThinking 
---

# 随笔：思维模型对机器学习 LLM 应用的影响是什么？

> 人类的语言是思维的表达工具，但是，语言实际上是构成思维的砖和瓦。试想一下，脱离了语言，思维基本是不可存在和不可操作的。
> 比如：当我思考思维模型对 LLM 的影响时，我的思维会在思维模型领域搜集大量的概念在 LLM 领域力同步搜集很多概念，然后，开始类比、推理、演绎、归纳等等。
> 但是，在这些类比、推理、演绎、归纳的操作上，语言就像：y = f(x) + b 一样，成为了符号、操作等一系列更为具体的语言，直到这些语言已经具体到直接作用到物理世界或想象世界的某个锚点上为止。
> 因此，LLM 在 Scaling Law 的作用下，才能够表现出令人惊叹的效果。不论我们如何否认，LLM 表现出的“智能”和人类的“智能”，以语言为出发点，在思维模型上是一致的。
> 由此可见，用语言铸就的思维模型一定会存在于 LLM 的权重网络中等待着我们去激活。所以，可以设计一个实验来验证这种想法。

```python
input = """
Structural Learning Theory (Joseph Scandura)
According to structural learning theory, what is learned are rules which consist of a domain, range, and procedure. There may be alternative rule sets for any given class of tasks. Problem solving may be facilitated when higher order rules are used, i.e., rules that generate new rules. Higher order rules account for creative behavior (unanticipated outcomes) as well as the ability to solve complex problems by making it possible to generate (learn) new rules.

Unlike information processing theories which often assume more complex control mechanisms and production rules, structural learning theory postulates a single, goal-switching control mechanism with minimal assumptions about the processor and allows more complex rule structures. Structural learning theory also assumes that “working memory” holds both rules and data (i.e., rules which do not act on other rules); the memory load associated with a task depends upon the rule(s) used for the task at hand.

Structural analysis is a methodology for identifying the rules to be learned for a given topic or class of tasks and breaking them done into their atomic components. The major steps in structural analysis are: (1) select a representative sample of problems, (2) identify a solution rule for each problem, (3) convert each solution rule into a higher order problem whose solutions is that rule, (4) identify a higher order solution rule for solving the new problems, (5) eliminate redundant solution rules from the rule set (i.e., those which can be derived from other rules), and (6) notice that steps 3 and 4 are essentially the same as steps 1 and 2, and continue the process iteratively with each newly-identified set of solution rules. The result of repeatedly identifying higher order rules, and eliminating redundant rules, is a succession of rule sets, each consisting of rules which are simpler individually but collectively more powerful than the ones before.

Structural learning prescribes teaching the simplest solution path for a problem and then teaching more complex paths until the entire rule has been mastered. The theory proposes that we should teach as many higher-order rules as possible as replacements for lower order rules. The theory also suggests a strategy for individualizing instruction by analyzing which rules a student has/has not mastered and teaching only the rules, or portions thereof, that have not been mastered.

Application
Structural learning theory has been applied extensively to mathematics and also provides an interpretation of Piagetian theory (Sandura & Scandura, 1980). The primary focus of the theory is problem solving instruction (Scandura, 1977). Scandura has applied the theoretical framework to the development of authoring tools and software engineering.

Example
Here is an example of structural learning theory in the context of subtraction provided by Scandura (1977):

The first step involves selecting a representative sample of problems such as 9-5, 248-13, or 801-302.
The second step is to identify the rules for solving each of the selected problems. To achieve this step, it is necessary to determine the minimal capabilities of the students (e.g., can recognize the digits 0-9, minus sign, column and rows). Then the detailed operations involved in solving each of the representative problems must be worked out in terms of the minimum capabilities of the students. For example, one subtraction rule students might learn is the “borrowing” procedure that specifies if the top number is less than the bottom number in a column, the top number in the column to the right must be made smaller by 1.
The next step is to identify any higher order rules and eliminate any lower order rules they subsume. In the case of subtraction , we could replace a number of partial rules with a single rule for borrowing that covers all cases.
The last step is to test and refine the resulting rule(s) using new problems and extend the rule set if necessary so that it accounts for all problems in the domain. In the case of subtraction, we would use problems with varying combinations of columns and perhaps different bases.
Principles
Whenever possible, teach higher order rules that can be used to derive lower order rules.
Teach the simplest solution path first and then teach more complex paths or rule sets.
Rules must be composed of the minimum capabilities possessed by the learners.
References
Scandura, J.M. (1970). The role of rules in behavior: Toward an operational definition of what (rule) is learned. Psychological Review, 77, 516-533.
Scandura, J.M. (1973). Structural Learning I: Theory and Research. London: Gordon & Breach.
Scandura, J.M. (1976). Structural Learning II: Issues and Approaches. London: Gordon & Breach.
Scandura, J.M. (1977). Problem Solving: A Structural/Process Approach with Instructional Applications. NY: Academic Press.
Scandura, J.M. & Scandura, A. (1980). Structural Learning and Concrete Operations: An Approach to Piagetian Conservation. NY: Praeger.
Scandura, J.M. (1984). Structural (cognitive task) analysis: A method for analyzing content. Part II: Precision, objectivity, and systematization. Journal of Structural Learning, 8, 1-28.
Scandura, J.M. (2004). Structural Learning Theory: Current Status and New Perspectives.
"""
template = input + """
Question:
How to let student learn and understanding 128 * 32, 412 * 829, 66 * 192?
Answer:
Let us think about it step by step.
"""
llm.invoke(template)
```
我通过 LLM ：Phi-3-mini-128k-instruct.Q4_K_M 得到如下结果：
```bash
Step 1: Understand what multiplication is. Multiplication is a shortcut for repeated addition. For example, instead of adding 5 + 5 + 5 + 5 + 5, we can simply multiply 5 * 5 to get the same result.
Step 2: Break down each problem into smaller parts that are easier to understand and solve. Let's take the first multiplication problem as an example: 128 * 32. We can break it down like this: (100 + 20) * (30 + 2). Now, we can multiply each part separately and then add them together.
Step 3: Solve each smaller part of the problem using multiplication or addition as needed. For example, for our first problem, we would calculate (100 * 30) + (100 * 2) + (20 * 30) + (20 * 2). This simplifies to 3000 + 200 + 600 + 40 = 4,840.
Step 4: Check your answer by using the inverse operation of multiplication, which is division. Divide your final result from step 3 by one of the original numbers in the problem (either 128 or 32). If you get the other number as a result, then your solution is correct!
Step 5: Practice solving similar problems to reinforce your understanding and improve your skills. The more you practice, the better you'll become at multiplication.
By following these steps, students can learn how to multiply numbers like 128 * 32, 412 * 829, and 66 * 192.
```
当然，限于手头的实验环境，我无法追踪此刻的 LLM 有多少规模的权重被激活，这个 A/B Test 还是很容易的，只需要把 Template 中的 Example 等信息去掉即可。如果您感兴趣，可以使用后文中介绍的这些思维模型来提示 LLM 以获得不同的 Result。

## 教学设计模型和理论历史：
 - 1903年——伊万·巴甫洛夫在对狗的消化系统进行研究时发现了经典条件反射理论。
 - 1910 年-桑代克提出了基于主动学习原则的定律和联结主义理论。
 - 1922 年-马克斯·韦特海默、库尔特·科夫卡和沃尔夫冈·科勒创立了格式塔心理学。
 - 1930 年代-心理学家列夫·维果茨基发表了近侧发展区 (ZPD)和脚手架理论。
 - 1932 年-心理学家弗雷德里克·巴特利特提出图式理论。
 - 1937 年-BF 斯金纳提出操作性条件作用理论。
 - 1937 年——梅和杜布出版了《竞争与合作》，其中提出、讨论和分析了合作与协作学习理论。
 - 1938 年——约翰·杜威在他的《经验与教育》一书中讨论了体验式教育。
 - 1950 年代-信息处理理论出现。
 - 1950 年代- 计算机辅助教学用于教育和培训环境。
 - 1954年-斯金纳提出程序化教学教育模式。
 - 1960 年代-基于建构主义学习理论开发了基于探究的学习模式。
 - 1961 年——杰罗姆·布鲁纳提出了发现学习模式。
 - 20 世纪 60 年代-霍华德·巴罗斯在加拿大麦克马斯特大学的医学教育项目中引入了基于问题的学习 (PBL) 。
 - 1963 年——大卫·奥苏贝尔 (David Ausubel) 发表了他关于包容理论的研究成果。
 - 1962 年——凯勒计划围绕个性化教学模式展开，并在美国各地的教育环境中得到运用。
 - 1971 年-艾伦·派维奥 (Allan Paivio) 提出双重编码理论，一种认知理论。
 - 1971 年-教育家 Vernom S. Gerlach 和 Donald P. Ely 提出了Gerlach 和 Ely 设计模型。
 - 1973 年-约瑟夫·斯坎杜拉 (Joseph Scandura) 出版了《结构学习 I：理论与研究》，概述了结构学习理论。
 - 1974 年-梅林·维特罗克 (Merlin Wittrock) 发表生成学习理论。
 - 1978年-维果茨基的社会文化学习理论影响了西方。
 - 1979 年-查尔斯·雷吉鲁斯 (Charles Reigeluth) 提出精化理论。
 - 1980 年-雷金纳德·雷文斯 (Reginald Revans) 提出行动学习模式。
 - 1983 年-David Merrill 提出了组件展示理论和教学模型。
 - 1983 年——JM Keller 的ARCS 动机模型出版。
 - 1983 年-心理学家霍华德·加德纳提出多元智能理论。
 - 1988 年-斯皮罗 (Spiro)、费尔托维奇 (Feltovich) 和库尔森 (Coulson) 提出他们的认知灵活性理论。
 - 1989 年-布朗、柯林斯、杜吉德和纽曼提出了他们的情境认知理论和认知学徒模式。
 - 1990 年——范德比尔特大学的认知与技术小组开发了锚式教学教育模式。
 - 1990 年代-多媒体和 CD-ROM 被引入教育环境。
 - 1991年-Lave和Wenger在《情境学习：合法的边缘参与》中介绍了实践社区模型和情境学习理论。
 - 1991 年-Hudspeth 和 Knirk 在《绩效改进季刊》上发表了基于案例的学习模型。
 - 1992年——Roger C. Schank发布了一份技术报告，介绍了基于目标的场景模型。
 - 1993 年——第一个计算机支持的有意学习环境(CSILE) 原型在大学环境中使用。
 - 1995 年- Saltzberg 和 Polyson 在万维网上发表了《分布式学习》，概述了分布式学习模型。
 - 1995 年——道奇 (Dodge) 和马奇 (March) 开发了 WebQuest。
 - 1996 年-Joseph R. Codde 教授发表了一份概述合同学习的报告。
 - 2005-乔治·西门子和斯蒂芬·唐斯提出联通主义学习理论。
 - 2006 年-Punya Mishra 和 Matthew J. Koehler 建立了TPACK 框架。
 - 2007 年——M. Lombardi 发表了一份概述真实学习模式的报告。
