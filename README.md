# docs

工作日报 / 周报 / 技术札记。

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

## 口吻

daily/weekly/notes 模仿作者本人口吻:

- 中英混杂自然, 技术词用英文 (paper, loss, coef, MSE, force MAE, batch, epoch, ckpt, smoke test, TF32, autograd 等), 不强行翻译
- 短句, 偏口语, 用逗号分句少用句号
- 不用 emoji, 不用 ✓✗⚠ 之类符号
- markdown bold 极克制, 一篇至多 1-2 处用在关键决策/路线名, 不在每个观察上加; 禁 italic
- bullet 不嵌套超过 2 层
- 不用 markdown 表格, 多列数字 bullet 内逗号分隔写
- 直接陈述结论, 不修辞
- 不写"用户 xxx" 之类第三人称指代, daily 是第一人称, 多轮 chat 里的发现/反驳/迭代抽象成最终结论一条 bullet, 不叙述对话过程
- 只写当天实际新做的事 (新 commit / 新实验结果 / 新决策), 不要重复昨天 daily 已写过的内容, 不要灌 ROADMAP 改动 / 待办 list 凑长度; 一次查询的结果 1-2 条 bullet 够
- 涉及时间/ETA/wall-time 必须用 evidence 算 (run start + log mtime + 当前 step 三者齐全), 不准猜 / 不准只凭早期 sample 外推; 没数据直接说没数据
