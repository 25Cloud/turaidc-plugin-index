## 插件提交检查清单

- [ ] 我已发布插件 Release：`{tag}`（本 PR 填写的 tag）
- [ ] 插件仓库通过[插件规范](https://github.com/25Cloud/turaidc-plugin-template/blob/main/docs/SPEC.md)校验
- [ ] `plugins.json` 条目中 `slug` / `domain` / `repo` / `tag` 与实际情况一致
- [ ] `sha` 已填写为本次审核对应的 commit sha（发布后若改动代码，请更新 sha）
- [ ] 我已阅读并同意[审核与安全条款](https://github.com/25Cloud/turaidc-plugin-index/blob/main/README.md)

### 清单字段完整

- [ ] `config.php` 的 `info` 必填字段齐全：`domain` / `slug` / `key` / `name` / `version` / `entry` / `capabilities`
- [ ] `capabilities` 是点号命名空间（如 `sms.send`），不是自然语言
- [ ] 声明了 `manifest_version: 2` 的条目，`version` / `min_platform_version` 均为语义化版本
- [ ] 条目里的 `key` / `domain` / `slug` / `version` / `capabilities` 与清单完全一致

### 版本号一致

- [ ] 条目 `version` 等于插件 `config.php` 的 `info.version`（去掉 tag 的 `v` 前缀后逐字相同）
- [ ] tag 与 `version` 指向同一个发布内容

### 路由与鉴权

- [ ] 只在自己 slug 前缀下注册路由：`api/v2/client/{slug}/*`、`api/v2/admin/{slug}/*`、`{slug}/v1/*`
- [ ] 没有自建鉴权入口，鉴权复用平台中间件（`ensure.admin` / `ensure.client` / `auth:sanctum` / `permission:*`）
- [ ] 没有注册全局中间件、全局配置或调度任务（调度走清单 `extra.scheduled_tasks` 声明）

### 迁移

- [ ] 迁移只新增：不删表、不删列、不改列类型
- [ ] 自建表使用 `plugin_{domain}_{slug}_` 前缀
- [ ] SQL 同时兼容 MySQL 5.7 与 8.x（未使用窗口函数、CTE 等 8.0 专属语法）

### 本地验证结果

```text
php -l                  →
php scripts/validate-manifest.php →
vendor/bin/pint --test  →
```

> 说明：安装器只下载本条目 `tag`（或 `sha`）对应的代码。审核通过后如需更新插件，请发新 Release 并另开 PR。
