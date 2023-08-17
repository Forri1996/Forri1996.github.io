---
title: chatgpt 提示词学习
date: 2023-08-17 10:33:29
tags: 提示词学习
---

结构化的输入。
定义清晰的角色，目标是你希望gpt输出的内容。
核心是wordflows，表示gpt按照这里的工作流程进行模块化输出，并且能够输出答案生成的逻辑。
下面提供一个sql编写的案例

## Role：
你是一位数据库专家，帮助用户解答sql编写的问题

## Goals：
1. 根据输入的表名分析表之间关系
2. 根据用户的需求输出统计分析的sql语句。统计输出如下字段：租户id，物理池id，物理池下实际的region数量，物理池下有安全组的region的数，物理池下每个region的安全组数量

## Constrains：
1. 实现的sql要有注释
2. 实现的sql要可以执行

## workflows：
1. 用文字描述提供的表之间的关系
2. 用文字描述实现数据统计sql的思路
3. 实现代码

下面是表的结构描述
现在有三张表，表的字段结构如下ddl所示。
```
CREATE TABLE `os_region` (
  `region_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '标识',
  `phy_res_pool_id` bigint(20) DEFAULT NULL COMMENT '物理资源池id',
  PRIMARY KEY (`region_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2214 DEFAULT CHARSET=utf8mb4 COMMENT='os region表';

-- cloud_desktop.os_tenant_res_rela definition

CREATE TABLE `os_tenant_res_rela` (
  `rela_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '关联标识',
  `tenant_id` bigint(20) NOT NULL COMMENT '租户标识',
  `region_id` bigint(20) DEFAULT NULL COMMENT '关联region_id',
  `az_id` bigint(20) DEFAULT NULL COMMENT '关联az_id',
  `phy_res_pool_id` bigint(20) DEFAULT NULL COMMENT '物理池ID'
  PRIMARY KEY (`rela_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3226328 DEFAULT CHARSET=utf8mb4 COMMENT='租户资源关联表（tenant_res_rela）';

-- cloud_desktop.os_security_group definition

CREATE TABLE `os_security_group` (
  `security_group_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '安全组标识',
  `tenant_id` bigint(20) DEFAULT NULL COMMENT '租户标识',
  `parent_id` bigint(20) DEFAULT NULL,
  `region_id` bigint(20) DEFAULT NULL,
  `phy_res_pool_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`security_group_id`),

) ENGINE=InnoDB AUTO_INCREMENT=1903552 DEFAULT CHARSET=utf8mb4 COMMENT='安全组';
```

gpt回答：
![image](https://github.com/Forri1996/blog-talk/assets/128824087/2b7b4dab-2330-4308-8a2c-755984694f79)

考虑到输入的token限制，表结构隐藏了一些字段属性，因此输出的sql还需要进行进一步的微调才能输出正确的结果。

通过这个结构化的prompt，gpt可以更有条理的回答我们问题。

下面提供一个完整的提示词模板：
```
# 角色

## profiles 简述
- author： 
- version： 
- language
- description

## goals

## skills

## constrains 限制

## workflows 工作流程

##　output format 输出格式

## examples 示例

## Initialization 初始化
```

下面是多个实际prompt案例：[prompt案例](https://github.com/Forri1996/prompts)
