# Product Studio 安装与模型配置

适用于当前正式版 v0.14.12。Product Studio 的发布包是 Node.js 服务，浏览器访问工作台；它不是桌面 EXE/MSI。ZIP 包含已构建的网页、服务端和运行依赖，不含 Node.js、用户数据和模型密钥。

## 准备与校验

1. 安装 Node.js 22.13 或更高版本，运行 `node --version` 确认版本。使用 ZIP 不需要 `npm install`。
2. 从 [最新正式发布](https://github.com/RocLing26/product-studio-releases/releases/latest) 下载 `product-studio-0.14.12-intranet.zip` 和 `product-studio-0.14.12-intranet.zip.sha256`，放在同一目录。
3. 在该目录校验文件，再解压 ZIP：

```bash
# macOS
shasum -a 256 -c product-studio-0.14.12-intranet.zip.sha256

# Linux（二选一，执行这一行即可）
sha256sum -c product-studio-0.14.12-intranet.zip.sha256
```

Windows PowerShell 可运行 `Get-FileHash .\product-studio-0.14.12-intranet.zip -Algorithm SHA256`，将输出的哈希与 `.sha256` 文件第一列比较。校验失败时重新下载。

## 本机启动

解压后进入 `product-studio-0.14.12-intranet` 目录，运行：

```bash
node server/index.mjs
```

浏览器打开 `http://127.0.0.1:4310`。服务默认仅监听本机地址；按 `Ctrl+C` 停止。首次打开为演示模式，可先体验示例流程；真实模型功能按下文配置。

## 数据保存与升级

未设置 `PM_DATA_DIR` 时，数据保存在程序目录下的 `.data/`。长期使用建议让数据目录与程序目录分开，例如在 macOS/Linux 启动前：

```bash
export PM_DATA_DIR="$HOME/product-studio-data"
node server/index.mjs
```

Windows PowerShell 可在启动前设置 `$env:PM_DATA_DIR = 'C:\ProductStudio\data'`。数据目录中包含工作记录和模型配置（包括 API Key），请限制访问权限并备份整个目录。

升级时先停止服务，备份完整数据目录及内网部署使用的 `.env`；将新版 ZIP 解压到新的程序目录，用原来的 `PM_DATA_DIR` 和 `.env` 启动。不要覆盖正在运行的目录，也不要用旧版程序打开新版数据库。工作台可以检查、下载并校验正式更新，但不会自动替换程序。

## 内网 HTTPS 部署

当前是单一管理账号的个人工作台，同一人可以从多台设备访问同一服务；没有团队成员权限和 SSO。内网部署需要已获准运行 Node.js 的服务器、内网域名、TLS 证书，以及同机 HTTPS 反向代理。

在解压后的程序目录运行：

```bash
node scripts/configure-intranet.mjs https://pm.company.example
```

将示例域名替换为实际访问地址。此命令生成 `.env`，其中包含随机 `PM_ACCESS_TOKEN`；不会覆盖现有 `.env`。在文件中设置独立的绝对路径，例如 `PM_DATA_DIR=/var/lib/product-studio`，保持 `PM_BIND_HOST=127.0.0.1`。然后运行：

```bash
node --env-file=.env server/index.mjs
```

由管理员将 HTTPS 域名反向代理到 `http://127.0.0.1:4310`，保留原始 Host 和 Authorization 请求头；ZIP 内有 `deploy/nginx.conf.example`。浏览器访问该 HTTPS 地址，使用账号 `pm` 和 `.env` 中的 `PM_ACCESS_TOKEN` 登录。请由管理员保管 `.env` 和访问口令。

## 模型配置

真实生成需要一个兼容 OpenAI API 的生成模型服务。服务器需要能够连接该服务；模型请求会发送到你配置的地址。未配置时保持演示模式，不调用外部模型。

1. 在工作台打开「工作台配置 → 模型与搜索」，点击「添加 Provider」。
2. 填写「Provider 名称」、`API Base URL`、`API Key`。Base URL 使用服务商给出的 API 根地址，通常以 `/v1` 结尾，不要填写完整的 `/chat/completions` 地址。远程模型地址必须是 HTTPS；本机模型服务可使用 localhost HTTP。
3. 点击「添加模型」，填写准确的「模型 ID」。根据服务商限制设置输出预算等参数，点击「保存 Provider」。
4. 在「当前生成配置」选择刚添加的模型，点击「测试生成模型连接」。成功后再试用灵感探索、PRD 或原型生成。

如需配置知识检索使用的 Embedding，在 Provider 中填写「Embedding 模型 ID」；它可以与生成模型在同一 Provider，也可以使用独立 Provider。然后在「知识冲突检索服务」选择它并点击「测试 Embedding 连接」。Embedding 可选：未配置时使用本地向量召回，但知识冲突的最终判断仍需生成模型。外部产品调研还需单独配置搜索服务。

管理员也可以在 `.env` 中指定生成模型，并用 `node --env-file=.env server/index.mjs` 启动：

```dotenv
PM_MODEL_MODE=compatible
PM_MODEL_BASE_URL=https://model.example.com/v1
PM_MODEL_NAME=your-model-id
PM_MODEL_API_KEY=your-private-api-key
```

`PM_MODEL_*` 环境变量会覆盖界面中的当前生成配置；移除后重启即可改由界面选择。示例值需替换为实际服务信息，不要提交含密钥的 `.env`。如果连接测试失败，检查服务器到模型地址的网络、API Key、模型 ID，以及服务商要求的输出参数和流式支持。

## 健康检查

本机启动后打开 `http://127.0.0.1:4310/api/health`，应返回服务健康状态。内网访问出现 403 时，检查浏览器访问地址、`.env` 中的 `PM_PUBLIC_ORIGIN` 与反向代理保留的 Host 是否一致。
