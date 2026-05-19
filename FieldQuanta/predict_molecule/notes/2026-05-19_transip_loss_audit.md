# TransIP force loss audit — paper text 跟代码不一致, paper 31.9 实际是 L2NormLoss 训的

> Project: FieldQuanta/predict_molecule · Tags: #paper-align #decision #infra

## 起因

strict baseline `fc_transip_strict_sharedrbf_paper_4M80ep` 跑到 ~42 epoch, F MAE 在 ~56 meV/Å 明显 plateau, 按当前斜率外推 80 epoch ~52 meV/Å, 远超 paper 31.9。E 反而追上甚至有望反超 paper 3.99。

观察 plateau 形状是 MSE 的典型 signature — 梯度 ∝ 残差, 残差小则梯度小, 训练自衰减锁死。怀疑 paper 表面写的 MSE force loss 跟代码实际跑的不一致。

## ~/TransIP 多分支 audit (2026-05-19)

跑了 7 个分支的 yaml + lightning 源码 grep, 找 force loss 实际实现:

| 分支 | 文件类型 | Energy loss | Force loss | 是否匹配 paper text "MSE" |
|---|---|---|---|---|
| `main` | `tasks/omol*.yaml` | `PerAtomMAELoss` | `L2NormLoss` | ❌ |
| `dsornd` | `tasks/omol*.yaml` | `PerAtomMAELoss` | `L2NormLoss` | ❌ |
| `arun_main` | `lightning/src/omol_train.py:training_step` | `F.l1_loss(e_pred/nat, e_t_n/nat)` per-atom MAE | `sqrt(Σ(F-F*)²)` per-atom L2 (一阶), mean over atoms per graph, mean over graphs | ❌ |
| `rbf` | 同上 | 同上 | 同上 | ❌ |
| `rbf_2` | 同上 | 同上 | 同上 | ❌ |
| `rbf_2_dist_rbf_llnl` | 同上 | 同上 | 同上 | ❌ |
| `rbf_edge_index` | 同上 | 同上 | 同上 | ❌ |

**全部 7 个分支, 一致用 L2 norm 一阶, 没一个用 MSE (平方)。**

## 关键代码 (`arun_main:lightning/src/omol_train.py:training_step`, line ~284-340)

```python
# === energy loss ===
# per graph MSE
# e_loss   = F.mse_loss(e_pred, e_t_n)          # ← 注释掉, 试过 MSE 后弃用
# loss = self.energy_coef * e_loss

# per atom MAE
nat = nat.to(e_pred.dtype)
e_loss = F.l1_loss(e_pred / nat, e_t_n / nat)    # ← 实际用的
loss = self.energy_coef * e_loss

# === force loss ===
if "forces" in out and f_t is not None:
    f_pred = out["forces"]

    # global mean L2
    # f_loss = torch.mean(torch.sqrt(torch.sum((f_pred - f_t / self.scale) ** 2, dim=-1)))
    # loss = loss + self.force_coef * f_loss     # ← 注释掉, 试过 global mean 后弃用

    # per-structure L2 (实际用的)
    per_atom_l2 = torch.sqrt(torch.sum((f_pred - f_t / self.scale)**2, dim=-1))
    B = int(data.batch.max().item()) + 1
    per_graph_sum = per_atom_l2.new_zeros(B).index_add_(0, data.batch, per_atom_l2)
    counts = torch.bincount(data.batch, minlength=B).clamp_min(1)
    per_graph = per_graph_sum / counts            # 先 mean over atoms per graph
    f_loss = per_graph.mean()                     # 再 mean over graphs
    loss = loss + self.force_coef * f_loss
```

注释痕迹明显: 既试过 energy MSE 也试过 global force L2, 最终都不用, 落在 `per-atom L2 → mean over atoms → mean over graphs` 这条路。

## 3 项硬证据 (一致指向 paper text MSE 错)

1. **代码注释痕迹** — 上述 `# e_loss = F.mse_loss(...)` 显示作者试过 MSE 后**主动弃用**, 不是从来没考虑
2. **跨分支一致性** — 7 个分支同一公式, 没一个 MSE。即使 paper 真用 MSE, 至少应该有 1 个分支留下 MSE 代码
3. **我们的训练曲线 fingerprint** — MSE plateau ~56 meV/Å, 按梯度 ∝ 残差几乎不可能自己降到 31.9; L2 norm 梯度幅度恒为 1, 才能继续推 F 到 paper 量级

## 最终诊断

**TransIP paper text "L_F = (1/(3|m|))·|F−F*|² (MSE)" 是 typo / 笔误**, 跟 paper 表 `equiv_coef: 5000 #5000` 但 paper text 写 5 同款 stale typo 性质。

**paper 报告的 F=31.9 meV/Å 是 L2NormLoss 训出来的**, 不是 MSE。

## Action

立刻切到 `tasks=omol_e_force` (TransIP code-aligned L2NormLoss), resume from 当前 step_110000 ckpt, 跑剩余 ~38 epoch 测试 L2 norm 是否真能突破 MSE plateau 推到 paper 量级。

启动 yaml 见 `daily/2026-05-19.md` 末尾段。新 run 用独立 `timestamp_id` (含 `l2norm_resume` 后缀) 避免覆盖 MSE run 的 ckpt — MSE run 当作"已完成对照组", 它的 65 个 eval block + plateau 证据已经记在 `2026-05-19_strict_baseline_eval_curve.md`。

## Followup

- 把这个 audit 写邮件给 TransIP 作者, 问 paper text 是否 typo (低优, 可选)
- ROADMAP 死路表 / Tier 0 描述 update, 改成 "paper 实测公式 L2NormLoss, paper text 文本错"
- `CLAUDE.md` `TransIP strict-alignment knobs` 段同步纠正 — 把 "Our default is paper-aligned via MSELossPerAtomSummed" 改成 "Paper text MSE 是 typo, 实际代码 L2NormLoss, 我们也切到 code-aligned"
