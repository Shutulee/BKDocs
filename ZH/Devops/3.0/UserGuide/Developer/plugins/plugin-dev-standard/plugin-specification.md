# 插件开发规范

## 插件定义规范

* 插件描述类基本信息通过线上流程定义
  * 插件工作台新增插件时，可指定标识、名称、开发语言
    * 标识、开发语言初始化之后不能再更改
  * 上架/升级插件时，可更新插件基本信息，包括名称、分类、描述等信息
* 插件执行、输入、输出相关定义，通过插件代码库中名为 task.json 的文件定义
  * task.json 存储在插件代码库根路径下
  * 详见 [插件配置规范](plugin-config.md)

## 插件开发规范

* 插件封装为命令行可执行命令：
  * 调起命令由task.json的 `execution.target` 字段指定
    * execution.target 格式为字符串
    * 调起命令中可以使用 ${var\_name} 的方式获取变量
  * 若调起执行前需安装依赖，安装命令由 task.json 的 `execution.demands` 字段指定
    * execution.demands 为列表，可配置多个安装命令
    * 安装命令可以使用 ${var\_name} 的方式获取变量
* 插件输入字段值，从指定的输入文件中获取
  * 输入信息文件名，由环境变量 `bk_data_input` 指定
  * 输入信息文件存放路径，由环境变量 `bk_data_dir` 指定
  * 输入信息文件内容为json，示例如下：

    ```text
    {
        "inputVar_1": "value1",
        "inputVar_2": "value2"
    }
    ```

  * 使用 SDK 进行开发时，无需关注输入文件位置和命名，使用 SDK 提供的方法获取输入即可
* 插件输出信息通过写文件的方式通知 bkci agent
  * 输出信息文件名，由环境变量 `bk_data_output` 指定
  * 输出信息文件存放路径，由环境变量 `bk_data_dir` 指定
  * 输出信息文件格式，详见[插件输出规范](plugin-output.md)
  * 使用 SDK 进行开发时，无需关注输出文件位置和名称，使用 SDK 提供的方法设置输出即可
* 插件执行结果将由两个策略判定：
  * 若输出信息文件中指定了 `status` 字段，则以该字段值为准
  * 若未指定输出信息文件，则以插件执行命令返回值为准
    * 返回值为 0，标识成功
    * 返回值非 0，标识失败
* 插件日志输出到控制台
  * bkci agent 将收集控制台日志（标准输出流、标准错误流）展示到流水线前台
* 插件级别的敏感信息管理
  * 如账号密码等敏感数据，可以在插件设置→私有设置界面管理
    * 平台将加密存储
    * 插件中使用时，通过 SDK 提供的方法获取即可
* 插件错误码和错误分类
  * 插件开发者需对导致插件执行失败的各种场景进行细分，使用错误码（errorCode）进行标识，并在插件日志、使用指引中给出详细的描述和解决方法，方便用户快速定位和解决问题
  * 插件开发者需对导致插件执行失败的错误进行归类，指定错误类型 errorType，用于度量统计

## 插件代码库管理规范

插件代码建议统一管理，便于分享、交接和系统级别的管理。

* 企业内部按照企业代码管理方式，统一到一个 group 下管理
* 开源的通用插件，可以联系 bkci 客服，提交到蓝鲸开源组件管理 [TencentBlueKing](https://github.com/TencentBlueKing) 下

## 插件帮助文档（详细描述）规范

* 目的
  * 方便用户了解插件工作原理和适用范围
  * 方便用户遇到问题时自助获取解决方案
* 提供方式
  * 发布插件时，填写在插件详细描述字段
  * 将展示在研发商店查看插件详情界面，或执行日志查看帮助。
* 使用文档**使用 markdown 语言**撰写，建议包含如下内容：

### 1、插件介绍 

* 简要说明插件功能

### 2、插件适用场景

* 详细描述插件适用场景

### 3、插件使用限制和受限解决方案

* \[可选\]调用频率限制
* \[可选\]并发限制
* \[可选\]网络限制，说明网络要求，或者需申请的网络策略
* \[可选\]若插件使用有权限控制（IP、用户等），需详细描述申请方式

### 4、插件常见的失败原因和解决方案

* 列举可能的失败原因、错误码，针对场景给出解决方案，方便用户快速解决问题

## 插件新增/发布流程

### 插件状态流转图

![png](../../../assets/store_plugin_status.png)

### 流程描述

* 开发者在插件工作台新增插件
  * 初始化的目的是拿到系统中唯一的插件标识
* 开发者本地开发调试插件
* 开发者调试 OK 后，在插件工作台新增版本，启动发布流程
* 提交版本后：
  * 进入测试阶段
    * 开发者可以在新增插件时选择的调试项目下，创建测试流水线，判断插件是否符合预期
    * 测试过程中，若发现问题，可修复代码 bug 后重新打成包，再重新传包后继续测试
  * 测试 OK 后，开发者点【继续】确认，则成功发布版本到研发商店
