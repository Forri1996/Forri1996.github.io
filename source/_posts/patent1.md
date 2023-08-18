---
title: 用gpt写专利
date: 2023-08-18 11:41:18
tags: 提示词学习
---
## 背景
最近一周hw导致日常业务开发工作都陷入停滞了，正好公司有专利撰写的指标，所以就尝试用gpt来写了一下。目前只是一个初稿，能否通过公司审核，结果不得而知…

## 提示词
近期在星球内学习了提示词的撰写方法，相关的经验都在这份文档里了：[如何写提示词](https://github.com/Forri1996/prompts)

按照这份经验，我撰写了一下专利的提示词：
```
## 角色：你是一名专业的技术方案专利撰写专家

## 简述
- 作者：Forri
- 版本：1.0 
- 语言：中文

## 目标
- 分析创新想法的技术实施细节
- 将细节转换为专利技术交底书文案并输出

## 技能
- 掌握专利撰写的方法
- 掌握技术开发领域的方案设计能力

## workflows 工作流程
- 根据用户提供的专利名称，通过问问题的方式逐步获得该创新想法的技术细节。
- 根据技术细节，完善技术交底书的几个模块，包括发明创造的目的，发明创造的技术方案，发明创造的技术优点，发明创造要保护的技术要点


## Initialization 初始化
作为专利撰写专家，我擅长剖析想法背后的技术方案，并将思路转换成技术交底书的文案内容。我将用清晰和精确的语言与您对话。请告诉我你的创新想法，我将竭诚为你服务。
```
我先用bingAI尝试着输出，他按照我的工作流程，问了我一系列的问题。而这些问题都是我在工作流程中已经提到了点，包括发明创造的目的，发明创造的技术方案，发明创造的技术优点，发明创造要保护的技术要点。
![image](https://github.com/Forri1996/blog-talk/assets/128824087/fc91e3d4-ae05-46af-a094-094a2ae830c0)

回答完这些问题后，bingAI给了我这样一个结果：
![image](https://github.com/Forri1996/blog-talk/assets/128824087/21d62318-02aa-43b0-bb28-af0494767864)

从内容上看，基本上就是基于上面他问我的问题，对我的回答进行了一个整理。

这里我认为有一个可以优化的点，就是AI问的问题不应该是我提供的，而是应该由我来引导AI，让他自己思考，写一个技术交底书应该了解哪些内容，然后向我提问。

总体而言，这次输出结果并不是很满意。

后来，考虑到gpt的“母语”是英文，于是我将提示词整体翻译成英文后，甩给chatGPT3.5（没错，就是竞争就是卷），给出了一个不太一样的结果。由于整体篇幅较长，这里直接给出gpt的分享链接（这个功能打满分）：[英文版专利撰写](https://chat.openai.com/share/6641e548-9100-4e15-bd8c-ab82bdcd8af6)

怎么说呢，感觉专业性上提了一个level，而且没有过多地被我的提示词所影响，但是也没有完全按照我提示词的要求来（ask question one by one）

但是，无论他输出的结果如何，其文案质量也已经吊打我这个完全没有专利撰写经验的小白了。

下面给出gpt帮我写的专利英文原文：


## Patent Technical Disclosure Document

**Title: A Method for Detecting Data Consistency in Heterogeneous Systems**

### 1. Purpose of the Invention

The present invention relates to the field of data consistency detection in heterogeneous systems. It proposes a novel method for automatically detecting data consistency between source and target data within heterogeneous systems. The invention aims to provide a non-invasive and universal solution for ensuring data consistency across different types of systems.

### 2. Technical Solution of the Invention

The disclosed method comprises an additional detection service that facilitates automated data consistency detection in heterogeneous systems. The detection service is configured with the following parameters:

1. Acquisition Methods of Source and Target Data:
   - Various acquisition methods, such as database queries or REST interface calls.
   - Acquisition parameters, including database connection information or REST interface addresses.
   - Acquisition results, such as return values or response bodies.

2. Data Consistency Detection Comparison Rules:
   - Comparison objects, specifying fields or attributes to be compared in source and target data.
   - Comparison methods, such as equality, containment, or size comparison.
   - Comparison results, indicating data consistency or inconsistency.

The method involves the following steps:

1. Configuration: Set acquisition methods for source and target data, along with data consistency detection comparison rules.
2. Activation: Initiate the detection service to periodically or trigger-basedly obtain source and target data.
3. Consistency Check: Conduct data consistency checks based on configured comparison rules.
4. Alerting and Repair: Notify relevant personnel via preset alerting methods if inconsistency is detected. Apply predetermined data repair solutions, such as manual or automatic repair.

### 3. Technical Advantages of the Invention

The present invention offers several technical advantages:

1. Non-Invasiveness: The method ensures data detection capability without invasive interference with the original and target systems.
2. Universality: The solution's universality allows it to be applied to diverse heterogeneous systems, regardless of their specific characteristics.

### 4. Protected Technical Points

The invention focuses on the following technical points:

1. A method for automated data consistency detection in heterogeneous systems.
2. Configuration of acquisition methods for source and target data.
3. Establishment of data consistency detection comparison rules.
4. Periodic or trigger-based data consistency checks.
5. Alerting methods for notifying relevant personnel.
6. Data repair solutions for addressing detected inconsistencies.

### 5. Novelty and Uniqueness

The disclosed method stands out due to its distinctive features:

1. Automation: Unlike existing methods, which often involve manual checks, this method offers automated data consistency detection.
2. Heterogeneous Systems: The solution is tailored to handle data consistency across various heterogeneous systems.
3. Universal Applicability: The method's non-invasive nature and universal compatibility make it suitable for any heterogeneous system.

---

中文版：

## 专利技术披露文件

**标题：一种用于检测异构系统中数据一致性的方法**

### 1. 发明的目的

本发明涉及异构系统中的数据一致性检测领域。它提出了一种新颖方法，用于自动检测异构系统中源数据和目标数据之间的数据一致性。本发明旨在为不同类型的系统提供一种非侵入性和通用性的解决方案，以确保数据在各个系统之间的一致性。

### 2. 发明的技术方案

所揭示的方法包括一个附加的检测服务，用于在异构系统中实现自动数据一致性检测。检测服务根据以下参数进行配置：

1. 源数据和目标数据的获取方法：
   - 各种获取方法，如数据库查询或REST接口调用。
   - 获取参数，包括数据库连接信息或REST接口地址。
   - 获取结果，如返回值或响应体。

2. 数据一致性检测比较规则：
   - 比较对象，指定在源数据和目标数据中要进行比较的字段或属性。
   - 比较方法，如相等性、包含性或大小比较。
   - 比较结果，指示数据的一致性或不一致性。

该方法包括以下步骤：

1. 配置：设置源数据和目标数据的获取方法，以及数据一致性检测的比较规则。
2. 激活：启动检测服务，定期或基于触发进行源数据和目标数据的获取。
3. 一致性检查：根据配置的比较规则进行数据一致性检查。
4. 报警和修复：通过预设的报警方法通知相关人员，如果发现不一致性，则应用预定的数据修复方案，如手动或自动修复。

### 3. 发明的技术优势

本发明具有以下几个技术优势：

1. 非侵入性：该方法确保数据检测能力，而不会对原始系统和目标系统造成侵入性干扰。
2. 通用性：该解决方案的通用性使其适用于不同特征的各种异构系统，而不受其特定特性的限制。

### 4. 受保护的技术要点

本发明专注于以下技术要点：

1. 一种在异构系统中实现自动化数据一致性检测的方法。
2. 配置源数据和目标数据的获取方法。
3. 建立数据一致性检测比较规则。
4. 基于周期性或触发的数据一致性检查。
5. 通知相关人员的报警方法。
6. 用于处理检测到的不一致性的数据修复方案。

### 5. 新颖性和独特性

所揭示的方法之所以独特，是因为具备以下显著特点：

1. 自动化：与现有方法不同，该方法提供了自动化的数据一致性检测。
2. 异构系统：该解决方案专门处理各种异构系统中的数据一致性。
3. 通用适用性：该方法的非侵入性和通用兼容性使其适用于任何异构系统。