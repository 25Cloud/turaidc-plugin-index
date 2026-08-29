# TuraIDC 插件市场索引

TuraIDC 插件市场的**插件清单仓库**。`plugins.json` 记录全部可安装插件；提交插件 = 往 `plugins.json` 加一条目并发 PR，**合并即审核通过**。

## 插件条目

```json
{
    "slug": "my_sms",
    "domain": "sms",
    "name": "我的短信",
    "description": "一句话描述插件能力",
    "developer": "25Cloud",
    "repo": "25Cloud/my_sms-plugin",
    "tag": "v1.0.0",
    "sha": "a1b2c3d...（可选）",
    "released_at": "2026-08-29T00:00:00+08:00",
    "license": "MIT",
    "homepage": "https://github.com/25Cloud/my_sms-plugin"
}
```

| 字段 | 必填 | 说明 |
| ---- | ---- | ---- |
| `slug` | ✓ | 插件标识（snake_case），全局唯一，与插件目录名一致 |
| `domain` | ✓ | `payment` / `verification` / `captcha` / `mail` / `sms` / `upstream` / `addons` |
| `name` | ✓ | 显示名 |
| `description` | ✓ | 一句话描述（展示在市场列表） |
| `developer` | ✓ | 开发者 / 组织 |
| `repo` | ✓ | 插件仓库 `owner/repo` |
| `tag` | ✓ | Release tag（如 `v1.0.0`），安装器只下载该 tag 对应的代码 |
| `sha` | ✗ | 审核通过的 commit sha；提供后安装器下载该 commit 代码并校验，最强防篡改 |
| `released_at` | ✓ | Release 发布时间（ISO 8601） |
| `license` | ✓ | 插件许可（如 MIT / AGPL-3.0） |
| `homepage` | ✗ | 插件主页 |

## 开发者提交流程

1. 用 [插件模板仓库](https://github.com/25Cloud/turaidc-plugin-template)（**Use this template**）创建插件仓库，`php scripts/rename.php <slug> "<显示名>"` 改名。
2. 开发、本地验证：`php -l`、`php scripts/validate-manifest.php`、`vendor/bin/pint --test`。
3. 发 **Release**：tag = `v{major.minor.patch}`（与 manifest `info.version` 去掉 `v` 一致）。
4. fork 本仓库，在 `plugins.json` 的 `plugins` 数组加一条目，**并发 PR**。
5. 维护者审核 PR 并合并 = 插件上架。TuraIDC 管理端即可搜索安装。

> **防篡改**：安装器只下载条目里 `tag`（或 `sha`）对应的代码。审核通过后代码如有更新，请发新 Release 并另开 PR 更新条目，不要改已审核条目的 tag/sha。

## 审核要点

- 插件遵循[插件包规范](https://github.com/25Cloud/TuraIDC/blob/main/docs/references/integrations/plugins/plugin-package-spec.md)：manifest 合法、入口类存在、命名空间 `TuraIDC\Plugins\{Domain}\{Slug}`。
- `slug` 不与已上架/内置插件冲突。
- `sha` 与 `tag` 指向真实存在且内容一致的提交（tag 与 sha 应指向同一 commit）。
- 插件代码在用户服务器执行，等于完全信任：审核时应检查是否存在恶意行为（外传密钥、后门等）。

## 安装侧

TuraIDC 端命令（详见[插件包规范](https://github.com/25Cloud/TuraIDC/blob/main/docs/references/integrations/plugins/plugin-package-spec.md)）：

```bash
php artisan plugin:market:list               # 同步索引并列出可安装插件
php artisan plugin:market:install my_sms     # 下载 tag 对应代码并安装
php artisan plugin:market:install my_sms --zip=./my_sms.zip   # 手动加载 zip
```

## 许可

索引数据（`plugins.json`）与流程文档采用 [MIT](./LICENSE)。插件本体许可由各条目 `license` 字段声明，与主仓（AGPL-3.0）相互独立，边界见[插件包规范](https://github.com/25Cloud/TuraIDC/blob/main/docs/references/integrations/plugins/plugin-package-spec.md)第 6 节。
