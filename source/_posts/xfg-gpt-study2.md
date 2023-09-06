---
title: xfg-gpt-study2
date: 2023-09-05 17:17:58
tags: 星球学习
---

项目工程分为5大模块
- chatgpt-api：demo模块，主要用于调试devops模块部署相关的内容。
- chatgpt-data：后端模块，主要提供前端接口，微信回调接口等。
- chatgpt-sdk-java：通用组件工具模块。
- chatgpt-web： 前端模块。
- dev-ops： 部署模块，主要维护如docker-composer文件等。

我们重点关注chatgpt-data，chatgpt-sdk-java和dev-ops模块。

整体项目的核心流程，通过微信扫码登录，认证通过后访问到聊天界面，通过响应式接口响应用户query，实现聊天交互的能力。

基于此，我们可以大致梳理出项目所需依赖的内容：
- 对接微信公众号，生成登录验证码
- 验证登录验证码，生成jwt登录凭证
- 对接chatgpt接口