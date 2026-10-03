# Product Studio · 产品经理 Agent 工作台

**从灵感到上线，让每一次产品决策成为有版本、有依据、可追溯的资产。**

Product Studio 把灵感探索、产品调研、需求文档、交互原型、设计文档、项目规划与产品知识连接在一条工作流中。Agent 会追问关键问题，帮助把模糊想法整理成需求。记录上线时，Agent 结合已确认的需求文档、原型、设计文档和实际交付内容提炼知识、更新知识库；知识冲突会进入待处理队列，供产品负责人核对。

当前版本：**0.18.16**。

## 核心能力

- **灵感探索：**Agent 主动追问用户、场景、目标和边界，结合产品知识与历史产物整理需求。
- **需求到设计：**生成并审阅需求文档、可交互 HTML 原型和设计文档，保留版本与上游依据。
- **知识回写：**记录上线时，从当前产物和实际交付中提炼新知识、更新旧口径，并写入知识库。
- **冲突校验：**提示相互矛盾的产品知识，支持对比原文、忽略或标记已处理。
- **知识图谱：**从资料引用的 Mermaid 单据关系、状态图和术语表生成待审核的关系与别名；审核后，知识搜索可沿关系扩展最多两跳，并返回原文证据和来源版本。
- **规划跟进：**通过需求看板、排期和 Agent 运行记录跟进工作进展。

## 0.18.16 更新

- PRD验收编号与来源追踪更完整，引用或版本有问题时先提示复核，再进入下游流程。
- 原型支持逐项流程检查与选中元素局部修改，设计文档显示具体映射缺项。
- 任务保留失败草稿与实际、估算或未知用量；恢复不会自动重发结果不明的模型请求。
- 交付包新增独立备份与恢复入口，可校验数据库、模型配置和访问口令。操作步骤见[数据保存与升级](INSTALL.md#数据保存与升级)。

## 界面预览

以下截图由 v0.14.12 在全新临时数据目录生成。「客户反馈中心」和冲突规则均为虚构示例，不包含真实项目资料。冲突截图使用测试模型判定；实际使用时需要配置生成模型。

**首页与产品资产流程**

![Product Studio 首页及产品资产流程](screenshots/overview.png)

**灵感探索中的 Agent 追问**

![Agent 追问产品问题并提供回答选项](screenshots/inspiration-followup.png)

**需求文档和交互原型**

![虚构的客户反馈中心需求文档](screenshots/requirement.png)

![虚构的客户反馈中心 HTML 原型](screenshots/prototype.png)

**上线时自动提炼并回写产品知识**

![Agent 根据实际交付更新产品知识的摘要](screenshots/agent-knowledge-writeback.png)

![产品知识库中自动沉淀的上线知识](screenshots/knowledge-updated.png)

**产品知识冲突校验**

![产品知识库展示两条虚构规则的口径冲突](screenshots/knowledge-conflict.png)

![冲突详情并排对比两条虚构规则的原文](screenshots/knowledge-conflict-detail.png)

## 安装与启动

1. 安装 Node.js 22.13 或更高版本。在 [Releases](https://github.com/RocLing26/product-studio-releases/releases/latest) 下载 `product-studio-0.18.16-intranet.zip` 及同名 `.sha256` 文件，并按 [完整安装说明](INSTALL.md) 校验 SHA-256。
2. 解压 ZIP，进入 `product-studio-0.18.16-intranet` 目录，运行 `node server/index.mjs`。包内已包含运行依赖，无需执行 `npm install`。
3. 在本机浏览器打开 `http://127.0.0.1:4310`。默认数据保存在解压目录下的 `.data/`；正式使用建议按 [完整安装说明](INSTALL.md#数据保存与升级)设置独立的 `PM_DATA_DIR`。

这是供浏览器访问的 Node.js 服务包，不是桌面安装程序。多设备内网访问需要配置 HTTPS 地址和访问口令，步骤见 [内网部署](INSTALL.md#内网-https-部署)。

## 模型配置

工作台默认以演示模式运行，不需要模型密钥。要使用真实模型生成、灵感探索和知识冲突判定，打开「工作台配置 → 模型与搜索」：

1. 点击「添加 Provider」，填写服务名称、兼容 OpenAI API 的 `API Base URL` 和 `API Key`。Base URL 通常以 `/v1` 结尾，不要填 `/chat/completions`；远程地址须使用 HTTPS，本机模型可使用 localhost HTTP。
2. 点击「添加模型」，填入服务提供的模型 ID，保存 Provider。
3. 在「当前生成配置」选择该模型，点击「测试生成模型连接」。

Embedding 模型用于知识冲突候选召回，可在 Provider 中填写「Embedding 模型 ID」，再选择「知识冲突检索服务」并测试连接。未配置 Embedding 时使用本地向量召回；**冲突结论仍需已配置的生成模型**。详细步骤和环境变量配置见 [完整安装说明](INSTALL.md#模型配置)。

工作台可检查正式版本并校验下载完整性；下载后仍需由管理员按说明升级。
