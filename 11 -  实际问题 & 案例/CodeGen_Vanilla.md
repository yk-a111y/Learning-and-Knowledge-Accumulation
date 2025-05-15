## 项目介绍
AI 驱动组件代码生成引擎，支持`多技术栈`和`多种AI模型`; 也可以基于该平台定制基于特定技术栈或特定场景的代码组件生成器。
区别于V0的单一技术栈。
![[20241221174859.png]]
![[20250402075536.png]]
## AI工作流程
![[20250406141127.png]]
## 前端模块
![[20250410214440.png]]研发流程![[20250410214518.png]]
## 渲染沙箱
![[Pasted image 20250515223043.png]]
通讯架构
![[image2-2025-04-17-21-45-24.png]]React组件的渲染
![[image1-2025-04-17-21-41-07.png]]
Vue组件的渲染
![[image4-2025-04-18-06-54-15.png]]
## AI驱动的组件生成 -- cursor mdc
```Gerkin
Feature: Codegen 可视化配置

  作为前端开发者，我需要一个直观的可视化的 Codegen 配置页面
  以便快速创建符合团队规范的高质量代码生成器

  Background:
    Given 用户已登录系统
    And 用户有权限访问 Codegen 配置页面

  Scenario: 基础信息配置
    When 用户访问 Codegen 配置页面
    Then 页面应该显示标题和描述的输入框
    And 这些字段应标记为必填项
    When 用户未填写标题或描述
    Then 系统应显示错误提示信息

  Scenario: 技术栈选择
    When 用户访问 Codegen 配置页面
    Then 页面应该显示技术栈下拉菜单
    And 下拉菜单应包含"React"和"Vue"选项
    And 技术栈字段应标记为必填项
    When 用户未选择技术栈
    Then 系统应显示错误提示信息

  Scenario: 渲染器配置
    When 用户访问 Codegen 配置页面
    Then 页面应该显示渲染器URL输入框
    And 渲染器URL字段应标记为必填项
    When 用户未填写渲染器URL
    Then 系统应显示错误提示信息

  Scenario: 引导 Prompts 配置
    When 用户访问 Codegen 配置页面
    Then 页面应该显示引导Prompts配置区域
    When 用户点击添加引导Prompt按钮
    Then 系统应新增一个Prompt输入框
    And 用户应能删除已添加的Prompt

  Scenario: 规则配置区域展示
    When 用户访问 Codegen 配置页面
    Then 页面应平铺展示所有规则类型
    And 用户应能直接编辑各类预设规则

  Scenario: 公共组件库规则配置
    When 用户访问公共组件库规则配置区域
    Then 页面应显示可视化的组件库名称列表编辑界面
    When 用户添加或删除组件库名称
    Then dataSet数组应相应更新

  Scenario: 样式规范规则配置
    When 用户访问样式规范规则配置区域
    Then 页面应显示样式提示词编辑框
    When 用户编辑样式相关提示词
    Then prompt字段应相应更新

  Scenario: 私有组件规则配置
    When 用户访问私有组件规则配置区域
    Then 页面应显示结构化表单
    When 用户点击新增组件库按钮
    Then 系统应新增一个组件库，同时可以配置组件库的名称、组件库包含的组件名、组件描述和API文档
    When 用户删除组件库
    Then 系统应删除该组件库
    When 用户编辑组件库名称、组件名、组件描述和API文档
    Then 对应的组件库名称、组件名、组件描述和API文档应相应更新

  Scenario: 文件结构规则配置
    When 用户访问文件结构规则配置区域
    Then 页面应显示文件结构提示词编辑框
    When 用户编辑文件结构相关提示词
    Then prompt字段应相应更新

  Scenario: 注意事项规则配置
    When 用户访问注意事项规则配置区域
    Then 页面应显示注意事项提示词编辑框
    When 用户编辑注意事项相关提示词
    Then prompt字段应相应更新

  Scenario: 配置验证提示
    When 用户尝试保存不完整的配置
    Then 系统应检查必填字段是否完整
    And 系统应显示明确的错误提示
    And 错误提示应指出哪些字段需要填写
```
