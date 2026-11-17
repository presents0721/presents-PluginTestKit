# Alife 插件测试框架

面向 Alife 5.x 的插件自检框架。核心是一条**真编译闸门**：用宿主同一套 Roslyn 引用集把插件源码重新编译一遍，把「本地全绿、客户端全红」挡在提交之前。

## 为什么需要它

Alife 用 Roslyn **逐文件直编** `.cs`，**不开 `ImplicitUsings`**。本地如果开了隐式 using，缺 `using System` 的代码在本地测试全绿，到客户端会炸成一片 `CS0246`，整包编译失败，模块系统发现不了这个插件 —— 功能列表里根本没有它，界面上也没有任何提示。

传统静态检查（清单、目录、工具名、配置规范）全都建立在「已经编译成功」这个前提上，而这个前提从没人验证过。这个框架先验证前提。

## 检查项

| 分类 | 内容 |
| --- | --- |
| **编译** | Roslyn 真编译；两遍策略区分「缺 using」与「缺第三方包」；已有 dll 且不旧于源码的插件跳过 |
| **契约** | 编译产物时效、双份 DLL 一致性、配置类规范、事件退订、工具注册、UI 基类调用、Content 模式 Closing |
| **XmlFunction** | 工具名重复/非法、参数类型 AI 能否生成、Description 缺失；注入参数自动排除 |
| **静态** | manifest 字段、目录卫生、类型全名冲突、依赖环 |
| **死配置** | 配置项在整个插件里只出现在定义处 —— 面板能改，改了没反应 |
| **UI 冒烟** | 用 agent-browser 真打开每个本地面板，抓 JavaScript 运行时错误（默认关闭） |
| **回归对比** | 两次报告按测试 ID 对齐，直接列出新失败 / 已修复 |

## 使用

**面板**：模块详情页点「打开测试面板」。

+ 全量自检：约 8 秒
+ 单插件快测：填插件 id，约 0.3～1.5 秒，**不落盘**
+ 对比最近两次：看有没有引入新失败
+ 面板冒烟：真浏览器逐个打开，约 20 秒

**AI 工具**：

| 工具 | 作用 |
| --- | --- |
| `testpanel` | 打开报告面板 |
| `testui` | 运行面板冒烟 |

**HTTP 接口**：`/api/run`、`/api/run?plugin=<id>`、`/api/compare`、`/api/uismoke`、`/api/report`、`/api/reports`

## 配置

`EnableCompileChecks` 默认开 —— 这是最关键的一道闸门。`EnableUiSmokeChecks` 默认关，需要外部 CLI `agent-browser`，建议用 `testui` 按需触发。

## 设计取舍

编译检查采用**两遍策略**：

1. 第一遍排除插件自己的旧 dll —— 否则旧类型会把「源码缺 using」掩盖掉
2. 若失败，第二遍加回自己的 dll —— 若能过，说明只是缺 `OpenCvSharp` / `NAudio` / `SherpaOnnx` 这类不在引用集里的第三方包，降级为提示而不是失败

`_razor.g.cs`（Razor 生成）与自绘 UI（大量 `OpenElement` 或 `Configuration.*`）不会被误报「未调 `base.BuildRenderTree`」。

## 许可

MIT
