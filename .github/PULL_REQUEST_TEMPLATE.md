## 插件提交流程

- [ ] 我已发布插件 Release：`{tag}`（本 PR 填写的 tag）
- [ ] 插件仓库遵循 [插件包规范](https://github.com/25Cloud/TuraIDC/blob/main/docs/references/integrations/plugins/plugin-package-spec.md)
- [ ] `plugins.json` 条目中 `slug` / `domain` / `repo` / `tag` 与实际情况一致
- [ ] `sha` 已填写为本次审核对应的 commit sha（发布后若改动代码，请更新 sha）
- [ ] 我已阅读并同意[审核与安全条款](https://github.com/25Cloud/turaidc-plugin-index/blob/main/README.md)

### 发布检查清单

- `php -l` 通过
- `php scripts/validate-manifest.php` 通过（模板自带）
- manifest `info.version` 与 tag 一致（去掉 `v` 前缀）

> 说明：安装器只下载本条目 `tag`（或 `sha`）对应的代码。审核通过后如需更新插件，请发新 Release 并另开 PR。
