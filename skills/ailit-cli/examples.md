# Space CLI Examples

## Login first

```powershell
ailit auth login
```

## Non-TTY login continuation

```powershell
ailit auth login --non-interactive --format json
ailit auth login --resume <workflowId> --result-set <resultSetId> --select <selectToken> --format json
```

## Health check

```powershell
ailit doctor
ailit doctor --format json
```

## Search customer

```powershell
ailit customer search 张三
ailit customer search 张三 --format json
ailit customer search --result-set <resultSetId> --select <selectToken> --format json
```

## Search product

```powershell
ailit product search 面巾纸
ailit product search 面巾纸 --format json
ailit product search --result-set <resultSetId> --select <selectToken> --format json
```

## Search settlement account

```powershell
ailit account search 支付宝
ailit account search 支付宝 --format json
ailit account search --result-set <resultSetId> --select <selectToken> --format json
```

## Full-payment sale preview

```powershell
ailit sale create --json skills/ailit-cli/templates/sale-create-full.json --dry-run
```

Use this default preview for user confirmation. It should already show shop, warehouse, customer, line items, settlement details, bill date, and total amount.

## On-account sale preview

```powershell
ailit sale create --json skills/ailit-cli/templates/sale-create-on-account.json --dry-run
```

For raw request inspection instead of user-facing preview:

```powershell
ailit sale create --json skills/ailit-cli/templates/sale-create-full.json --dry-run --format json
```

## Real create after confirmation

```powershell
ailit sale create --json skills/ailit-cli/templates/sale-create-full.json
```

## Query sales orders

```powershell
ailit sale list --today
ailit sale list --week
ailit sale list --start 2024-01-01 --end 2024-01-31
ailit sale list --today --format json
ailit sale get <id>
ailit sale get <id> --format json
```

## Invalidate or delete a sale

```powershell
# Always confirm with the user first
ailit sale get <id>
ailit sale invalid <id>
ailit sale invalid <id> --restore   # Restore an invalidated sale
ailit sale delete <id>
```

## Stock queries

```powershell
ailit stock list --format json
ailit stock low                      # 低库存（系统阈值）
ailit stock low --threshold 5        # 自定义阈值
ailit stock out                      # 缺货商品
```

## Sales reports

```powershell
ailit report all                     # 综合报表（推荐，并发效率最高）
ailit report all --format json
ailit report today
ailit report week
ailit report month
ailit report hot-sale
ailit report hot-sale --start 2024-01-01 --end 2024-01-31 --format json
```

## Customer debt

```powershell
ailit customer debt
ailit customer debt --format json
ailit customer get <id> --format json
```

## Purchase records

```powershell
ailit purchase list --today
ailit purchase list --week
ailit purchase list --start 2024-01-01 --end 2024-01-31 --format json
ailit purchase supplier
```

## Suggested agent sequence — sale creation

1. If auth state is unknown, run `ailit doctor`
2. If the CLI is not logged in, use `ailit auth login` for terminal users, or `ailit auth login --non-interactive --format json` in non-TTY environments
3. If login returns `selection_required`, show the numeric `selectToken`, `displayName`, and `fields` when available, falling back to `summary`, then continue with `ailit auth login --resume <workflowId> --result-set <resultSetId> --select <selectToken> --format json`
4. Re-run `ailit doctor`
5. `ailit customer search <keyword>`
6. If search returns `selection_required`, present the structured candidate fields and continue with `--result-set <resultSetId> --select <selectToken> --format json`
7. `ailit product search <keyword>`
8. If search returns `selection_required`, present the structured candidate fields and continue with `--result-set <resultSetId> --select <selectToken> --format json`
9. Extract `productId`, `productSkuId`, `displayName` from `validatedItems[0].meta` — no need to call `ailit product get` again
10. `ailit account search <keyword>` when payment mode is `FULL`
11. If search returns `selection_required`, present the structured candidate fields and continue with `--result-set <resultSetId> --select <selectToken> --format json`
12. `ailit sale create --json <file> --dry-run`
13. Use the dry-run preview as the confirmation view for the user; only switch to `--format json` if raw request inspection is needed
14. Only after explicit confirmation, run `ailit sale create --json <file>`

## Suggested agent sequence — daily business overview

1. `ailit doctor`
2. `ailit report all --format json`     # 今日/本周/本月汇总 + 热销
3. `ailit stock low --format json`      # 低库存预警
4. `ailit customer debt --format json`  # 欠款客户

## When JSON is appropriate

```powershell
ailit doctor --format json
ailit product search 面巾纸 --format json
ailit product search --result-set <resultSetId> --select <selectToken> --format json
ailit auth login --non-interactive --format json
ailit auth login --resume <workflowId> --result-set <resultSetId> --select <selectToken> --format json
ailit sale create --json skills/ailit-cli/templates/sale-create-full.json --dry-run --format json
ailit report all --format json
ailit stock low --format json
ailit customer debt --format json
```
