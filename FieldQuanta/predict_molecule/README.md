# predict_molecule

OMol25 大规模分子力场模型。FieldQuanta 公司项目之一。

- 仓库: `~/predict_molecule` (本机, crystal 分支, 暂未推 GitHub)
- 当前里程碑: Matbench-Discovery 晶体稳定性 F1 (非等变 transformer + 数据/增广)
- 目标: F1 ≥ 0.9 (北极星 eSEN-30M-OAM 0.925; 同级参考 eSEN-30M-MPtrj 0.819)

## 子目录

- `daily/` — 每日日报
- `weekly/` — 周报 (ISO week, 每周二写)
- `notes/` — 长篇技术调研 / 决策札记

## 约定

- 加班当天的日报,文件名在日期后加 `(+)`,如 `2026-06-06(+).md`。

### 周报

- 每周二写,覆盖上周三~本周二,文件名 `YYYY-Www.md` (按当周二的 ISO week 编号)。
- 只写 2-3 点最关键进展,不堆细节。
- 直接用专业术语 (F1 / MPtrj / OMat24 / OAM 等),读者都懂,不做外行化解释。
- 关键数字用表格呈现。
- 对标模型写全称精确版本 (如 `eSEN-30M-MPtrj` 0.819 / `eSEN-30M-OAM` 0.925),不简写成 "eSEN"。
