# ailit-cli

`ailit` is a Go command-line tool for working with Zhihuiji Space sales workflows from the terminal. It can check local configuration and API connectivity, search customers, products, and settlement accounts, and create sales orders either interactively or from JSON drafts against the `space.zhihuiji.cn` backend.

## Features

- Check whether local config and API access are usable with `ailit doctor`
- Search customers, products, and settlement accounts from the terminal
- Create a sales order through the interactive `sale create` flow
- Preview and submit sales orders from JSON draft files
- Output machine-readable JSON for search and dry-run workflows when needed

## Installation

You can install from npm, download a release bundle, or build from source.

Prerequisites:

- Node.js (`npm` / `npx`) for npm install and skills installation
- Go 1.23+ if you plan to build from source
- Access to a valid `space.zhihuiji.cn` token

### Option 1: Install from npm

Recommended flow:

1. Install the CLI from npm
2. Install the skill separately with `skills add`

```bash
npm install -g @co-ailit/ailit-cli
npx skills add co-ailit/ailit-cli-releases -y -g
```

This matches the same separation used by `larksuite/cli`: npm installs the CLI, and the skill is installed as a separate step.

### Option 2: Download a release bundle

Each release archive includes:

- the platform-specific `ailit` binary
- `skills/ailit-cli`
- `install.sh`
- `install.ps1`

Recommended flow:

1. Download the archive for your OS and CPU from GitHub Releases
2. Extract it to a local directory
3. Run `install.sh` or `install.ps1`, or let your agent install from the extracted folder

Examples:

```powershell
./install.ps1
```

```bash
chmod +x ./install.sh
./install.sh
```

The install scripts:

- copy the binary into `INSTALL_BIN_DIR` if set
- otherwise install to a platform default directory such as `~/.local/bin` on Unix-like systems or `%USERPROFILE%\AppData\Local\ailit\bin` on Windows
- try `npx skills add <local-release-path> -g -y` first, so the `skills` tool decides the right install location for Codex, Claude Code, and other supported agents
- if the `skills` tool is unavailable or fails, fall back to `INSTALL_SKILLS_DIR`, then detected directories such as `$CODEX_HOME/skills`, `~/.codex/skills`, or Claude-compatible locations
- update the user PATH automatically unless `INSTALL_UPDATE_PATH=0`

Set `INSTALL_USE_SKILLS_CLI=0` if you want to skip the external `skills` installer and force the fallback copy mode.

This makes the extracted release folder self-contained for agents that need both the binary and the skill.

### Option 3: Build from source

Build from the repository:

```powershell
go build -o ailit.exe .
./ailit.exe --help
```

You can also use `make install` (Linux/macOS) or move `ailit.exe` to a directory in your `PATH`.

If you do not want to install it globally, use `./ailit` or `./ailit.exe` in the examples below.

## Quick Start

The shortest path to a working local setup is:

```powershell
go build -o ailit.exe .
./ailit.exe config set baseUrl https://space.zhihuiji.cn
./ailit.exe config set token "Bearer <token>"
./ailit.exe config set defaultShopId "1"
./ailit.exe doctor
./ailit.exe customer search 123
```

After you place the built binary on your `PATH`, the same flow becomes:

```powershell
ailit config set baseUrl https://space.zhihuiji.cn
ailit config set token "Bearer <token>"
ailit config set defaultShopId "1"
ailit doctor
ailit customer search 123
```

## Configuration

The CLI stores configuration in `config.json` under the `AILIT_HOME` directory. If `AILIT_HOME` is not set, it falls back to `~/.ailit`.

- Default config file: `~/.ailit/config.json`
- Override home directory: set the `AILIT_HOME` environment variable before running commands
- Required keys for normal usage: `baseUrl`, `token`
- Optional keys: `webBaseUrl`, `defaultShopId`, `defaultOperatorId`

Example `config.json`:

```json
{
  "baseUrl": "https://space.zhihuiji.cn",
  "webBaseUrl": "https://space.zhihuiji.cn/#/",
  "token": "Bearer <token>",
  "defaultShopId": "1",
  "defaultOperatorId": "42"
}
```

Set values with commands:

```powershell
ailit config set baseUrl https://space.zhihuiji.cn
ailit config set webBaseUrl https://space.zhihuiji.cn/#/
ailit config set token "Bearer <token>"
ailit config set defaultShopId "1"
ailit config show
```

`webBaseUrl` is used only for `ailit auth login`. If it is not set, the CLI derives the browser login URL from `baseUrl` by adding `/#/`.

Override the config home for the current PowerShell session:

```powershell
$env:AILIT_HOME = 'D:\path\to\.ailit'
ailit config show
```

## Commands

### Health check

Verify that local configuration is present and the API is reachable:

```powershell
ailit doctor
ailit doctor --format json
```

### Configuration

Manage the local config file:

```powershell
ailit config show
ailit config set baseUrl https://space.zhihuiji.cn
ailit config set webBaseUrl https://space.zhihuiji.cn/#/
ailit config set token "Bearer <token>"
ailit config set defaultShopId "1"
```

Supported config keys:

- `baseUrl`
- `webBaseUrl`
- `token`
- `defaultShopId`
- `defaultOperatorId`

### Search

Search customers:

```powershell
ailit customer search 123
ailit customer search 123 --format json
```

Search products:

```powershell
ailit product search 面巾纸
ailit product search 面巾纸 --format json
```

Search settlement accounts:

```powershell
ailit account search 支付宝
ailit account search --format json
```

### Non-TTY Selection Continuation

In agent or non-TTY environments, machine-readable search may return `selection_required` instead of forcing a terminal prompt.

Search continuation examples:

```powershell
ailit product search 面巾纸 --format json
ailit product search --result-set <resultSetId> --select <token> --format json

ailit customer search 123 --format json
ailit customer search --result-set <resultSetId> --select <token> --format json

ailit account search 支付宝 --format json
ailit account search --result-set <resultSetId> --select <token> --format json
```

`selection_required` responses include:

- `resultSetId`
- `items[].selectToken`
- `items[].displayName`
- `items[].summary`

Use `resultSetId` together with a returned `selectToken` or stable item ID to continue.

For browser login in non-TTY environments, `ailit auth login` can also return a resumable workflow after browser callback:

```powershell
ailit auth login --non-interactive --format json
ailit auth login --resume <workflowId> --result-set <resultSetId> --select <token> --format json
```

The resumable login response includes:

- `workflowId`
- `step`
- `resultSetId`
- candidate `items`

### Sale creation

Start the interactive sale creation flow:

```powershell
ailit sale create
```

Preview the interactive request without creating the sale:

```powershell
ailit sale create --dry-run
ailit sale create --dry-run --format json
```

Create from a JSON draft:

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-full.json
```

After `ailit auth login`, the current logged-in user's `uid` is synced to `defaultOperatorId` and used as the default `operatorId` when creating sales orders; JSON drafts also auto-fill missing `operatorId` and `billDate`.

## JSON-Driven Sale Creation

JSON mode is useful when you already know the customer, product, and settlement details and want a repeatable draft you can preview before submission.

The CLI expects a business draft with these top-level fields:

- optional `shopId`
- optional `warehouseId`
- `customer`
- `lines`
- `settlement`
- optional `accountId` when you want to choose a specific payee account for `settlement.mode: "FULL"`

Example draft shape:

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

The included sample draft templates live in `skills/space-cli/templates/`:

- `skills/space-cli/templates/sale-create-full.json`
- `skills/space-cli/templates/sale-create-on-account.json`

If `shopId` is empty or omitted, the CLI uses the configured default shop.
If `warehouseId` is empty or omitted, the CLI uses the configured default warehouse when available; otherwise the request falls back to the selected shop.

Preview before creating:

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-full.json --dry-run
ailit sale create --json skills/space-cli/templates/sale-create-on-account.json --dry-run
```

If you need the raw response shape for troubleshooting or automation:

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-on-account.json --dry-run --format json
```

Run the real create command only after the draft looks correct:

```powershell
ailit sale create --json skills/space-cli/templates/sale-create-full.json
```

## Notes and Limitations

- The CLI does not handle login. You must supply a valid token yourself.
- The current backend target is `https://space.zhihuiji.cn`.
- The current command surface is intentionally narrow: configuration, health check, search, and sale creation.
- Full-payment flows depend on a resolved settlement account. You can provide `accountId` in JSON mode to force a specific payee account, or let the CLI resolve a default account when one is available.
- The current full-payment path is practical but not a rich account-selection workflow.
- Interactive prompts require a normal terminal session. Non-interactive environments are better suited to JSON-driven sale creation.
- Commands now resolve `auto -> terminal/headless` at runtime. Headless flows must rely on explicit IDs, JSON drafts, or structured continuation responses instead of prompt-only behavior.

## Troubleshooting

`ailit: command not found`

- Make sure you ran `go build` and use `./ailit.exe`, or
- Add the directory to your `PATH` or use `make install`

`ailit` fails with a `node_modules/ailit-cli/dist/src/index.js` error

- You are invoking an old global npm wrapper from an earlier Node-based build
- Remove the stale shims from `%APPDATA%\npm` such as `ailit`, `ailit.cmd`, and `ailit.ps1`
- Build the Go CLI with `go build -o ailit.exe .` and run `./ailit.exe ...`, or place `ailit.exe` on your `PATH`

`doctor` reports missing configuration or API errors

- Confirm `baseUrl` is set to `https://space.zhihuiji.cn`
- Confirm your token is present in `config show`
- Confirm `AILIT_HOME` points to the config directory you expect

Search commands return no matches

- Try a broader keyword
- Verify the token still has access to the target data
- Use `--format json` if you need the raw response shape for inspection

Sale creation fails from JSON

- Check that required top-level fields are present
- For `FULL` settlement mode, add `accountId` only if you need to force a specific payee account
- Run the same command with `--dry-run` first to inspect the resolved request

You need to inspect the current local config

```powershell
ailit config show
```
