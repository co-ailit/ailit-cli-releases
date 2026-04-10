# ailit-cli

`ailit` 是一个面向智慧记AI进销存销售流程的 Go 命令行工具，可在终端中完成本地配置与接口连通性检查、客户/商品/结算账户搜索，以及通过交互式流程或 JSON 草稿创建销售单。CLI 本地名称和配置目录使用 `ailit` 品牌，但后端接口目标仍是 `space.zhihuiji.cn`。

## 功能

- 用 `ailit doctor` 检查本地配置是否完整、接口是否可用
- 在终端里搜索客户、商品和结算账户
- 通过交互式 `sale create` 流程创建销售单
- 从 JSON 草稿预览并提交销售单
- 在搜索和 dry-run 场景下按需输出机器可读的 JSON 结果

## 安装

你可以通过 npm 安装，也可以下载 release 压缩包，或者从源码构建。

前置条件：

- 已安装 Node.js（包含 `npm` / `npx`），用于 npm 安装和独立安装 skills
- 如果要从源码构建，已安装 Go 1.23+
- 已具备可访问 `space.zhihuiji.cn` 的有效 token

### 方式 1：通过 npm 安装

推荐流程：

1. 先通过 npm 安装 CLI
2. 再单独通过 `skills add` 安装 skill

```bash
npm install -g @co-ailit/ailit-cli
npx skills add co-ailit/ailit-cli-releases -y -g
```

这和 `larksuite/cli` 的安装方式保持一致：npm 只负责安装 CLI，skill 作为独立步骤安装。

### 方式 2：下载 release 压缩包

每个 release 压缩包都会包含：

- 对应平台的 `ailit` 二进制
- `skills/ailit-cli`
- `install.sh`
- `install.ps1`

推荐流程：

1. 从 GitHub Releases 下载与你的操作系统和 CPU 架构匹配的压缩包
2. 解压到本地目录
3. 运行 `install.sh` 或 `install.ps1`，或者把解压目录直接交给 agent 处理

示例：

```powershell
./install.ps1
```

```bash
chmod +x ./install.sh
./install.sh
```

安装脚本会：

- 优先把二进制复制到 `INSTALL_BIN_DIR`
- 如果未设置，则复制到平台默认目录，例如 Unix-like 系统下的 `~/.local/bin`，或 Windows 下的 `%USERPROFILE%\AppData\Local\Programs\ailit\bin`
- 优先尝试执行 `npx skills add <本地解压目录> -g -y`，让 `skills` 工具自己决定 Codex、Claude Code 等工具对应的安装目录
- 如果 `skills` 工具不可用或安装失败，再回退到 `INSTALL_SKILLS_DIR`，以及自动探测出的已有 skill 目录
- 默认自动把二进制目录加入用户级 PATH，除非显式设置 `INSTALL_UPDATE_PATH=0`

如果你明确不想调用外部 `skills` 安装器，可以设置 `INSTALL_USE_SKILLS_CLI=0`，强制使用回退复制模式。

这样 release 解压目录本身就是一个可交给 agent 的自包含安装包。

### 方式 3：从源码构建

在仓库中本地构建：

```powershell
go build -o ailit.exe .
./ailit.exe --help
```

如果不想使用 `./ailit.exe` 的方式调用，可以将其添加到 `PATH` 环境变量所在的目录，或者（针对 Linux/macOS）执行 `make install`。

下面所有示例里的 `ailit` 都可以替换成 `./ailit.exe`。

## 快速开始

从零到跑通一条最短链路，可以直接按下面执行：

```powershell
go build -o ailit.exe .
./ailit.exe config set baseUrl https://space.zhihuiji.cn
./ailit.exe config set token "Bearer <token>"
./ailit.exe config set defaultShopId "1"
./ailit.exe doctor
./ailit.exe customer search 123
```

如果已经把编译后的二进制放进 `PATH`，同样的流程可以写成：

```powershell
ailit config set baseUrl https://space.zhihuiji.cn
ailit config set token "Bearer <token>"
ailit config set defaultShopId "1"
ailit doctor
ailit customer search 123
```

## 配置

CLI 会把配置保存到 `AILIT_HOME` 目录下的 `config.json`。如果没有设置 `AILIT_HOME`，则回退到 `~/.ailit`。

- 默认配置文件位置：`~/.ailit/config.json`
- 自定义配置目录：先设置环境变量 `AILIT_HOME`，再执行命令
- 常规必填项：`baseUrl`、`token`
- 可选项：`webBaseUrl`、`defaultShopId`、`defaultOperatorId`

示例 `config.json`：

```json
{
  "baseUrl": "https://space.zhihuiji.cn",
  "webBaseUrl": "https://space.zhihuiji.cn/#/",
  "token": "Bearer <token>",
  "defaultShopId": "1",
  "defaultOperatorId": "42"
}
```

通过命令设置配置：

```powershell
ailit config set baseUrl https://space.zhihuiji.cn
ailit config set webBaseUrl https://space.zhihuiji.cn/#/
ailit config set token "Bearer <token>"
ailit config set defaultShopId "1"
ailit config show
```

`webBaseUrl` 只用于 `ailit auth login` 打开浏览器。如果没有设置，CLI 会基于 `baseUrl` 自动补成带 `/#/` 的前端登录地址。

在当前 PowerShell 会话中临时覆盖配置目录：

```powershell
$env:AILIT_HOME = 'D:\path\to\.ailit'
ailit config show
```

## 命令

### 健康检查

确认本地配置存在且接口可访问：

```powershell
ailit doctor
ailit doctor --format json
```

### 配置管理

查看和维护本地配置文件：

```powershell
ailit config show
ailit config set baseUrl https://space.zhihuiji.cn
ailit config set webBaseUrl https://space.zhihuiji.cn/#/
ailit config set token "Bearer <token>"
ailit config set defaultShopId "1"
```

当前支持的配置键：

- `baseUrl`
- `webBaseUrl`
- `token`
- `defaultShopId`
- `defaultOperatorId`

### 搜索

搜索客户：

```powershell
ailit customer search 123
ailit customer search 123 --format json
```

搜索商品：

```powershell
ailit product search 面巾纸
ailit product search 面巾纸 --format json
```

搜索结算账户：

```powershell
ailit account search 支付宝
ailit account search --format json
```

### 非 TTY 续做选择

在 agent 或无 TTY 环境里，机器可读搜索结果可能会返回 `selection_required`，而不是强制进入终端交互。

搜索续做示例：

```powershell
ailit product search 面巾纸 --format json
ailit product search --result-set <resultSetId> --select <token> --format json

ailit customer search 123 --format json
ailit customer search --result-set <resultSetId> --select <token> --format json

ailit account search 支付宝 --format json
ailit account search --result-set <resultSetId> --select <token> --format json
```

`selection_required` 响应会包含：

- `resultSetId`
- `items[].selectToken`
- `items[].displayName`
- `items[].fields`
- `items[].summary`

其中：

- `items[].fields` 是优先展示的结构化字段列表，每项包含 `label` 和 `value`
- `items[].summary` 是兼容旧消费方的单行摘要文本

当前内置搜索候选字段约定：

- 商品：商品名称 + 规格、条形码、零售价、批发价
- 客户：客户名称 + 联系电话、客户欠款
- 结算账户：账户名称 + 余额

后续使用返回的 `resultSetId` 搭配 `selectToken` 或稳定 ID 继续执行。

在无 TTY 浏览器登录场景下，`ailit auth login` 也可能在浏览器回调后返回可恢复 workflow：

```powershell
ailit auth login --non-interactive --format json
ailit auth login --resume <workflowId> --result-set <resultSetId> --select <token> --format json
```

这类登录续做响应会包含：

- `workflowId`
- `step`
- `resultSetId`
- 候选 `items`

### 销售单创建

启动交互式销售单创建流程：

```powershell
ailit sale create
```

只预览交互式流程生成的请求，不真正创建销售单：

```powershell
ailit sale create --dry-run
ailit sale create --dry-run --format json
```

通过 JSON 草稿创建：

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-full.json
```

执行 `ailit auth login` 后，当前登录用户的 `uid` 会同步写入 `defaultOperatorId`，销售单创建时会默认作为 `operatorId` 使用；JSON 草稿如果未填写 `operatorId` 和 `billDate`，也会自动补齐。

## 基于 JSON 的销售单创建

当你已经明确客户、商品和结算信息，并希望把草稿重复使用、先预览再提交时，JSON 模式会更合适。

CLI 读取的是业务草稿结构，顶层通常包括：

- 可选的 `shopId`
- 可选的 `warehouseId`
- `customer`
- `lines`
- `settlement`
- 可选的 `accountId`，仅当你想在 `settlement.mode: "FULL"` 时强制指定某个收款账户才需要填写

示例草稿结构：

```json
{
  "shopId": "",
  "warehouseId": "",
  "customer": {
    "id": "1375392384811008",
    "name": "1231"
  },
  "lines": [
    {
      "productId": "1409956763009024",
      "productSkuId": "1409956763009026",
      "productName": "小招喵牌面巾纸",
      "quantity": 2,
      "unitPrice": 15
    }
  ],
  "settlement": {
    "mode": "FULL",
    "remark": "CLI create"
  }
}
```

仓库内提供的示例模板位于 `skills/space-cli/templates/`：

- `skills/space-cli/templates/sale-create-full.json`
- `skills/space-cli/templates/sale-create-on-account.json`

如果 `shopId` 为空或省略，CLI 会使用当前配置里的默认门店。
如果 `warehouseId` 为空或省略，CLI 会优先使用当前配置里的默认仓库；如果没有默认仓库，则请求层会回退到当前门店。

其中 FULL 付款模板里可以看到带 `accountId` 的写法，但完整 JSON 草稿并不强制要求该字段；只有在你想固定某个收款账户时才需要提供。

先做预览，再确认是否真实创建：

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-full.json --dry-run
ailit sale create --json skills/space-cli/templates/sale-create-on-account.json --dry-run
```

如果需要排查原始返回结构或做自动化处理，可再加：

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-on-account.json --dry-run --format json
```

确认草稿无误后，再执行真实创建：

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-full.json
```

## 说明与限制

- CLI 不负责登录，你需要自行准备有效 token。
- 当前后端目标地址是 `https://space.zhihuiji.cn`。
- 当前命令面故意保持收敛，只覆盖配置、健康检查、搜索和销售单创建。
- 全款流程依赖可解析的结算账户。JSON 模式下可通过 `accountId` 强制指定收款账户；如果不填，则由 CLI 在可用时尝试解析默认账户。
- 目前的全款流程强调实用可用，不是完整的富账户选择工作流。
- 交互式流程需要正常终端环境；如果运行环境不适合交互，优先使用 JSON 草稿模式。
- CLI 现在会在运行时统一解析 `auto -> terminal/headless` 交互模式；headless 路径必须依赖显式 ID、JSON 草稿或结构化 continuation 响应，不能只依赖 prompt 交互。

## 故障排查

`ailit: command not found`

- 确认是否已通过 `go build` 编译，并使用 `./ailit.exe` 执行，或
- 将目录加入到你的 `PATH` 环境变量中

`ailit` 提示 `node_modules/ailit-cli/dist/src/index.js` 找不到

- 这说明当前命中的是历史遗留的 npm 全局包装脚本，不是现在的 Go 可执行文件
- 删除 `%APPDATA%\npm` 下遗留的 `ailit`、`ailit.cmd`、`ailit.ps1` 等脚本
- 重新执行 `go build -o ailit.exe .`，然后使用 `./ailit.exe ...`，或者把 `ailit.exe` 放进 `PATH`

`doctor` 提示缺少配置或接口报错

- 确认 `baseUrl` 已设置为 `https://space.zhihuiji.cn`
- 确认 `config show` 中已存在 token
- 确认 `AILIT_HOME` 指向的是你预期的配置目录

搜索命令没有返回结果

- 尝试使用更宽泛的关键词
- 确认当前 token 仍有目标数据的访问权限
- 如果需要排查原始返回结构，可加 `--format json`

通过 JSON 创建销售单失败

- 检查顶层必需字段是否齐全
- 对于 `FULL` 结算模式，只有在你要强制指定收款账户时才填写 `accountId`
- 先用同一条命令加 `--dry-run` 检查最终请求内容

想确认当前本地配置

```powershell
ailit config show
```
