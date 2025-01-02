---
title: '通过 OneAPI 接入模型'
description: '通过 OneAPI 接入模型'
icon: 'api'
draft: false
toc: true
weight: 745
---

FastGPT 目前采用模型分离的部署方案，FastGPT 中只兼容 OpenAI 的模型规范（OpenAI 不存在的模型采用一个较为通用的规范），并通过 [One API](https://github.com/songquanpeng/one-api) 来实现对不同模型接口的统一。

[One API](https://github.com/songquanpeng/one-api) 是一个 OpenAI 接口管理 & 分发系统，可以通过标准的 OpenAI API 格式访问所有的大模型，开箱即用。


## FastGPT 与 One API 关系

可以把 One API 当做一个网关，FastGPT 与 One API 关系：

![](/imgs/sealos-fastgpt.webp)

## 部署

### Docker 版本

`docker-compose.yml` 文件已加入了 OneAPI 配置，可直接使用。默认暴露在 3001 端口。

### Sealos 版本

* 北京区: [点击部署 OneAPI](https://hzh.sealos.run/?openapp=system-template%3FtemplateName%3Done-api)
* 新加坡区(可用 GPT) [点击部署 OneAPI](https://cloud.sealos.io/?openapp=system-template%3FtemplateName%3Done-api)

![alt text](/imgs/image-59.png)

部署完后，可以打开 OneAPI 访问链接，进行下一步操作。

## OneAPI 基础教程

### 概念

1. 渠道：
   1. OneApi 中一个渠道对应一个 `Api Key`，这个 `Api Key` 可以是GPT、微软、ChatGLM、文心一言的。一个`Api Key`通常可以调用同一个厂商的多个模型。
   2. One API 会根据请求传入的`模型`来决定使用哪一个`渠道`，如果一个模型对应了多个`渠道`，则会随机调用。
2. 令牌：访问 One API 所需的凭证，只需要这`1`个凭证即可访问`One API`上配置的模型。因此`FastGPT`中，只需要配置`One API`的`baseurl`和`令牌`即可。令牌不要设置任何的模型范围权限，否则容易报错。

![alt text](/imgs/image-60.png)

### 大致工作流程

1. 客户端请求 One API
2. 根据请求中的 `model` 参数，匹配对应的渠道（根据渠道里的模型进行匹配，必须完全一致）。如果匹配到多个渠道，则随机选择一个（同优先级）。
3. One API 向真正的地址发出请求。
4. One API 将结果返回给客户端。

### 1. 登录 One API

![step5](/imgs/oneapi-step5.png)

### 2. 创建渠道

在 One API 中添加对应渠道，直接点击 【添加基础模型】，不要遗漏了向量模型（Embedding）

![step6](/imgs/oneapi-step6.png)

### 3. 创建令牌

| | |
| --- | --- |
| ![step7](/imgs/oneapi-step7.png) | ![alt text](/imgs/image-61.png) |

### 4. 修改账号余额

One API 默认 root 用户只有 200刀，可以自行修改编辑。

![alt text](/imgs/image-62.png)

### 5. 修改 FastGPT 的环境变量

有了 One API 令牌后，FastGPT 可以通过修改 `baseurl` 和 `key` 去请求到 One API，再由 One API 去请求不同的模型。修改下面两个环境变量：

```bash
# 务必写上 v1。如果在同一个网络内，可改成内网地址。
OPENAI_BASE_URL=https://xxxx.cloud.sealos.io/v1
# 下面的 key 是由 One API 提供的令牌
CHAT_API_KEY=sk-xxxxxx
```

## 接入其他模型

**以添加文心一言为例:**

### 1. OneAPI 新增模型渠道

类型选择百度文心千帆。

![](/imgs/oneapi-demo1.png)

### 2. 修改 FastGPT 配置文件

可以在 `/projects/app/src/data/config.json` 里找到配置文件（本地开发需要复制成 config.local.json）,按下面内容修改配置文件，最新/更具体的配置说明，可查看[FastGPT 配置文件说明](/docs/development/configuration)。

配置模型关键点在于`model` 需要与 OneAPI 渠道中的模型一致。

```json
{
  "llmModels": [ // 语言模型配置
    {
      "model": "ERNIE-Bot", // 这里的模型需要对应 One API 的模型
      "name": "文心一言", // 对外展示的名称
      "avatar": "/imgs/model/openai.svg", // 模型的logo
      "maxContext": 16000, // 最大上下文
      "maxResponse": 4000, // 最大回复
      "quoteMaxToken": 13000, // 最大引用内容
      "maxTemperature": 1.2, // 最大温度
      "charsPointsPrice": 0, 
      "censor": false,
      "vision": false, // 是否支持图片输入
      "datasetProcess": true, // 是否设置为知识库处理模型
      "usedInClassify": true, // 是否用于问题分类
      "usedInExtractFields": true, // 是否用于字段提取
      "usedInToolCall": true, // 是否用于工具调用
      "usedInQueryExtension": true, // 是否用于问题优化
      "toolChoice": true, // 是否支持工具选择
      "functionCall": false, // 是否支持函数调用
      "customCQPrompt": "", // 自定义文本分类提示词（不支持工具和函数调用的模型
      "customExtractPrompt": "", // 自定义内容提取提示词
      "defaultSystemChatPrompt": "", // 对话默认携带的系统提示词
      "defaultConfig":{}  // 请求API时，挟带一些默认配置（比如 GLM4 的 top_p）
    }
  ],
  "vectorModels": [ // 向量模型配置
    {
      "model": "text-embedding-ada-002",
      "name": "Embedding-2",
      "avatar": "/imgs/model/openai.svg",
      "charsPointsPrice": 0,
      "defaultToken": 700,
      "maxToken": 3000,
      "weight": 100
    },
  ]
}
```

### 3. 重启 FastGPT

**Docker 版本**

```bash
docker-compose down
docker-compose up -d
```

**Sealos 版本**

直接找到 FastGPT 服务，点击重启即可。


## 其他服务商接入参考

这章介绍一些提供商接入 OneAPI 的教程，配置后不要忘记修改 FastGPT 配置文件。

### 阿里通义千问

千问目前已经兼容 GPT 格式，可以直接选择 OpenAI 类型来接入即可。如下图，选择类型为`OpenAI`，代理填写阿里云的代理地址。

目前可以直接使用阿里云的语言模型和 `text-embedding-v3` 向量模型（实测已经归一化，可直接使用）

![alt text](/imgs/image-63.png)

### 硅基流动 —— 开源模型大合集

[硅基流动](https://cloud.siliconflow.cn/i/TR9Ym0c4) 是一个专门提供开源模型调用平台，并拥有自己的加速引擎。模型覆盖面广，非常适合低成本来测试开源模型。接入教程：

1. [点击注册硅基流动账号](https://cloud.siliconflow.cn/i/TR9Ym0c4)
2. 进入控制台，获取 API key: https://cloud.siliconflow.cn/account/ak
3. 新增 OneAPI 渠道，选择`OpenAI`类型，代理填写：`https://api.siliconflow.cn`，密钥是第二步创建的密钥。

![alt text](/imgs/image-64.png)

由于 OneAPI 未内置 硅基流动 的模型名，可以通过自定义模型名称来填入，下面是获取模型名称的教程：

1. 打开[硅基流动模型列表](https://siliconflow.cn/zh-cn/models)
2. 单击模型后，会打开模型详情。
3. 复制模型名到 OneAPI 中。

| | | |
| --- | --- | --- |
| ![alt text](/imgs/image-65.png) | ![alt text](/imgs/image-66.png)| ![alt text](/imgs/image-67.png) |
