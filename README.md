# docs

工作日报 / 周报 / 技术札记。Claude 维护, 我审阅。

## 公司 / 项目层级

- [**FieldQuanta**](./FieldQuanta/) — 公司
  - [`predict_molecule/`](./FieldQuanta/predict_molecule/) — OMol25 分子力场 (~/predict_molecule, fairchem_unit 分支)

每个项目下统一结构: `daily/ weekly/ notes/`。

## Tag 约定

- **路径 tag (主)**: 文件路径本身就是公司 + 项目标签, e.g. `FieldQuanta/predict_molecule/daily/2026-05-19.md`
- **顶部 tag 行 (辅)**: 每个 daily/weekly 顶部一行
  ```
  > Project: FieldQuanta/predict_molecule · Tags: #bug-fix #eval-result
  ```
  GitHub 搜索能命中 `#hashtag` 文本, 便于跨项目检索主题 (例如所有 `#bug-fix`)

## 常用 tag 词表 (predict_molecule)

`#bug-fix` `#training-launch` `#eval-result` `#decision` `#paper-align` `#infra` `#scope` (e.g. `#scope-tier1`)
