# TuraIDC 插件市场索引

TuraIDC 插件市场的**插件清单仓库**。`plugins.json` 记录全部可安装插件；提交插件 = 往 `plugins.json` 加一条目并发 PR，**合并即审核通过**。

- 索引语义版本：`schema_version: 2`（清单 v2 字段参考 [插件规范](https://github.com/25Cloud/turaidc-plugin-template/blob/main/docs/SPEC.md)）
- 平台解析仍以 `schema: 1` 为准，见下文「版本与兼容」。

## 索引文件结构

```json
{
    "schema": 1,
    "schema_version": 2,
    "updated_at": "2026-10-27T00:00:00+08:00",
    "plugins": []
}
```

| 字段 | 说明 |
| ---- | ---- |
| `schema` | 平台当前解析的索引结构版本，**必须保持 `1`**。改它等于让所有已部署实例拒绝该索引 |
| `schema_version` | 索引语义版本：`1` = 只有基础字段，`2` = 含 v2 字段（本仓当前为 2） |
| `updated_at` | 索引更新时间（ISO 8601，带时区） |
| `plugins` | 插件条目数组，见下 |

### 版本与兼容

- **新增字段不升 `schema_version`**：平台按白名单读取自己认识的字段，未知字段直接忽略，不会报错。
- **删字段、改字段名或改语义才升 `schema_version`**，并且必须先让平台侧支持新结构，再更新索引；顺序反了会把老实例锁死。
- `schema` 与 `schema_version` 并行存在，是刻意的过渡形态：老平台读 `schema`，新平台读 `schema_version`，索引可以同时服务两者。

## 条目字段

```json
{
    "key": "my_sms",
    "name": "我的短信",
    "domain": "sms",
    "slug": "my_sms",
    "repo": "owner/my_sms-plugin",
    "tag": "v1.0.0",
    "sha": "（可选）审核锁定的 commit sha",
    "version": "1.0.0",
    "min_platform_version": "1.0.0",
    "manifest_version": 2,
    "summary": "一句话描述插件能力",
    "description": "一句话描述插件能力（v1 字段）",
    "developer": "owner",
    "released_at": "2026-10-27T00:00:00+08:00",
    "license": "MIT",
    "homepage": "https://github.com/owner/my_sms-plugin",
    "capabilities": ["sms.send"],
    "official": false
}
```

| 字段 | 必填 | 说明 | 示例 |
| ---- | ---- | ---- | ---- |
| `key` | ✓ | 平台内唯一键，与插件清单 `info.key` 一致 | `my_sms` |
| `name` | ✓ | 市场与后台显示名 | `我的短信` |
| `domain` | ✓ | `payment` / `verification` / `captcha` / `mail` / `sms` / `upstream` / `addons` | `sms` |
| `slug` | ✓ | 插件标识，snake_case，等于插件目录名与清单 `info.slug` | `my_sms` |
| `repo` | ✓ | 插件所在仓库 `owner/repo` | `25Cloud/my_sms-plugin` |
| `tag` | 独立仓库必填 | Release tag，安装器只下载该 tag 的代码 | `v1.0.0` |
| `sha` | ✗ | 审核锁定的 commit sha；填写后安装器改用该 commit 并强校验 | `a1b2c3d…` |
| `path` | monorepo 必填 | 插件在仓库内的子目录，逐段只允许 `[A-Za-z0-9._-]`（`..`、绝对路径一律拒绝）。安装器按它**精确落位**，不靠深度猜 | `backend/plugins/addons/ai_assistant` |
| `ref` | monorepo 必填 | 子目录所在分支或 tag；`{ref}` 解析优先级 `sha` → `refs/tags/{tag}` → `refs/heads/{ref}` | `main` |
| `version` | ✓ | 插件版本，**必须等于插件 `config.php` 的 `info.version`** | `1.0.0` |
| `min_platform_version` | ✗ | 允许安装的最低平台版本 | `1.0.0` |
| `manifest_version` | ✗ | 插件清单版本，缺省为 1 | `2` |
| `summary` | ✗ | 一句话简介（v2 字段） | `一句话描述` |
| `description` | ✓ | 一句话描述；**当前平台列表读取的是这个字段**，v2 条目请与 `summary` 保持一致 | `一句话描述` |
| `developer` | ✓ | 开发者或组织 | `25Cloud` |
| `released_at` | ✗ | Release 时间（ISO 8601） | `2026-10-27T00:00:00+08:00` |
| `license` | ✓ | 插件许可，与清单 `info.license` 一致 | `MIT` / `AGPL-3.0-or-later` |
| `homepage` | ✗ | 插件主页 | `https://github.com/owner/my_sms-plugin` |
| `capabilities` | ✗ | 能力声明，点号命名空间，与清单 `info.capabilities` 一致 | `["sms.send"]` |
| `official` | ✗ | 是否官方插件 | `true` |

条目形态只有两种，别混用：

- **独立仓库**：`repo` + `tag`（+ 可选 `sha`）。**推荐形态**：下载量几百 KB，`sha` 可锁定到审核过的 commit。
- **monorepo**：`repo` + `path` + `ref`（可与 `sha` 叠加）。插件随主仓一起发布时用。
  - 安装器会按 `path` 精确落到 `{解压根}/{仓库快照目录}/{path}`，找不到就报错，不会去猜；
  - 下载的是**整仓快照**（几十 MB），超时取 `PLUGIN_MARKET_MONOREPO_TIMEOUT`（默认 120 秒）；
  - 只写 `ref: main` 等于跟随分支漂移，上架条目建议 `path` + `sha` 一起填。

随主仓内置发布的插件仍**优先提供独立发行仓**（下载小、锁定强、不依赖主仓分支状态）：
官方 AI 助手就是这么做的——主仓 `backend/plugins/addons/ai_assistant` 是唯一真源，
发行镜像仓 [`25Cloud/turaidc-plugin-ai-assistant`](https://github.com/25Cloud/turaidc-plugin-ai-assistant)
逐字节同步并按版本打 tag，条目指向镜像仓的 `tag` + `sha`。

## 校验要求

提交前必须自查（维护者会复核）：

1. 条目 `version` 与插件 `config.php` 的 `info.version` 完全一致（去掉 tag 的 `v` 前缀后逐字比较）。
2. `slug` / `key` / `domain` 与清单一致，`slug` 等于插件目录名。
3. 插件仓库本地跑通：
   ```bash
   php -l $(git ls-files '*.php')
   php scripts/validate-manifest.php      # 模板自带，校验清单与类签名
   vendor/bin/pint --test
   ```
4. `capabilities` 是点号标识（`sms.send`），不是自然语言。
5. JSON 合法：`node -e "JSON.parse(require('fs').readFileSync('plugins.json','utf8'))"`。

## 开发者提交流程

1. 用[插件模板仓库](https://github.com/25Cloud/turaidc-plugin-template)（**Use this template**）创建插件仓库，`php scripts/rename.php <slug> "<显示名>"` 改名。
2. 开发并本地验证（见上一节）。
3. 发 **Release**：tag = `v{major.minor.patch}`，与清单 `info.version` 一致。
4. fork 本仓库，在 `plugins.json` 的 `plugins` 数组加一条目（按上面的字段表填全），**并发 PR**。
5. 维护者审核并合并 = 插件上架。TuraIDC 管理端即可搜索安装。

> **防篡改**：安装器只下载条目里 `tag`（或 `sha`）对应的代码。审核通过后代码如有更新，请发新 Release 并另开 PR 更新条目，不要改已审核条目的 tag/sha。

## 审核要点

- 清单合法：入口类存在、`capabilities` 点号命名、`version` 语义化，见[插件规范](https://github.com/25Cloud/turaidc-plugin-template/blob/main/docs/SPEC.md)。
- 只在自有路由前缀下注册：`api/v2/client/{slug}/*`、`api/v2/admin/{slug}/*`、`{slug}/v1/*`；复用平台鉴权中间件，不自建鉴权入口、不注册全局中间件。
- 迁移只新增：不删表、不删列、不改列类型，且兼容 MySQL 5.7 与 8.x。
- `slug` 不与已上架/内置插件冲突。
- `tag` 与 `sha` 指向同一 commit；条目 `version` 与清单一致。
- 插件代码在用户服务器执行，等于完全信任：审核时检查是否存在恶意行为（外传密钥、后门等）。

## 安装侧

TuraIDC 端命令：

```bash
php artisan plugin:market:list               # 同步索引并列出可安装插件
php artisan plugin:market:install my_sms     # 下载 tag 对应代码并安装
php artisan plugin:market:install my_sms --zip=./my_sms.zip   # 手动加载 zip
```

## 许可

索引数据（`plugins.json`）与流程文档采用 [MIT](./LICENSE)。插件本体许可由各条目 `license` 字段声明，与主仓（AGPL-3.0-or-later）相互独立。
