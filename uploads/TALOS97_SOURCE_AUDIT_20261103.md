# TALOS97 原始设计文件与结构问题说明

日期：2026-11-03

## 交付内容

本目录随附的 `DNA_6HB_100_designs.zip` 是目前找到的 TALOS/6HB 原始设计包；它包含 100 个 TALOS 设计（`6hb_001`–`6hb_100`）及 caDNAno JSON、CanDo `.cndo`、序列、oxDNA `design.top`/`initial.dat`、转换审计和复现脚本。TALOS97 训练集实际使用其中 97 个设计；训练集没有 `6hb_064`、`6hb_079`、`6hb_089`，因此这三个设计在本包中作为额外候选保留，没有删除原始内容。

我逐设计把训练数据清单中的文件 SHA-256 与该包比对，结果如下：

| 文件 | 97 个设计中逐字节一致 |
|---|---:|
| `design.top` | 97/97 |
| `initial.dat` | 97/97 |
| `designed_pairs.txt` | 97/97 |
| `topology_to_cadnano.json` | 包内没有该训练缓存映射文件 |
| `target_replica1_mean.conf` | 包内没有训练标签文件 |

因此，这个包可以作为 **97 个设计的原始结构/设计文件来源**；它不是完整的 TALOS97 训练数据发布包。训练标签、NPZ 缓存和 `topology_to_cadnano.json` 仍以 TALOS97 endpoint 数据包为准。源包 SHA-256 和逐设计比对结果记录在 `TALOS97_SOURCE_MATCH_MANIFEST.json`。

源包 SHA-256：

```text
6fea0a528f1326f6e6019197e87c7b37bb7b8833f29e0292a0c3a11f2c441c81
```

## 已发现的结构与映射问题

### 1. 全部 97 个设计都有同一类长 backbone 边候选

只读审计了每个设计的 `initial.dat`、`design.top`、`topology_to_cadnano.json`、`designed_pairs.txt` 和输入缓存。每个设计都出现 22–38 条“同 virtual helix、相邻 axial index”的拓扑 backbone 边，而这些边的最短几何长度超过 2 nm；97 个设计合计 2,964 条。最大长度范围约 3.56–5.82 nm，中位数约 4.60 nm。

这更像是一个系统性的 virtual-helix/axial 映射、交叉连接或拓扑转换问题，不能据此直接判定 97 个设计全部损坏，也不能只删除少数设计。应回到原始 caDNAno/CanDo、拓扑连接和转换日志逐边核对。跨 virtual helix 的长边可能是合法 crossover，不能使用“超过 1 nm”这一条规则一概删除。

### 2. 未配对区段可以构造，但不能证明恢复了原坐标

97 个设计共有 2,964 段连续的 7 nt 未配对区段。利用两端锚点和相邻 backbone 步长，可以为每段构造等步长圆弧桥接；这证明局部几何上存在可行坐标，但不证明它就是 TALOS 当时使用的坐标。

在 `6hb_001` 的一个区段中，两端锚点间距约 1.6941 nm；两套满足相同端点和步长的镜像构造，中间节点最大相差约 2.469 nm。缺少原始碱基姿态、扭转、局部曲率、排斥/接触和转换器日志时，解并不唯一。当前留存的 `initial.dat` 已是转换产物，不能反推转换前的真实未配对坐标。详细重建边界见仓库内 `experiments/talos97_rmsf_scalar/input_source_audit_20261103/RECONSTRUCTION_ZH.md`。

### 3. `6hb_009`/`6hb_015` 的问题主要在续跑轨迹/标签，不是初始结构筛查的特异异常

原始初始几何筛查中，009 的非局部 `<0.2 nm` 候选数为 6，015 为 21，全体中位数为 21；它们没有因为初始输入几何而成为明显离群点。现有标签审计显示，009 的 continuation 轨迹物理失稳：四个节点 RMSF 约 799–1256 nm，坐标超出 200 nm 盒达到数千 nm，minimum-image backbone 邻接距离约 97–98 nm，首帧总能量约 `1.147×10^9`。这四个点贡献约 99.9709% 的该设计平方误差。015 也有同类严重异常。

同一续跑批次中，047、024、051 也从第一帧起有异常高能量；047 的 1999/2000 帧出现超过 10 nm 的拓扑邻接距离，024 和 051 分别有 11 和 32 帧。它们是否达到 009 的污染程度尚未由标签值单独证明，不能按预测误差直接删样本。

## 当前结论与使用边界

1. 现在可以确认：训练集的 97 个结构文件（至少 `design.top`、`initial.dat`、`designed_pairs.txt`）在这个 TALOS 设计包中有逐字节对应来源。
2. 现在不能确认：包内的 caDNAno/CanDo 文件就是训练流程当时保存的唯一原始版本，也不能用它替代训练标签、映射缓存或模拟日志。
3. 系统性长边候选应优先追踪转换/映射规则；不能把它当作 009 的单点异常。
4. 009/015 的极端 RMSF 首先应按物理轨迹 QC 处理（重跑、标记无效或由数据负责人决定），不能裁剪标签后继续训练。
5. 本次只做文件核对和报告打包，没有修改原始设计、标签、缓存或模型，没有训练、模拟、GPU、远程主机或 `sbatch` 任务。

## 可复核记录

- 原始设计包：`DNA_6HB_100_designs.zip`
- 逐设计哈希比对：`TALOS97_SOURCE_MATCH_MANIFEST.json`
- 初始输入审计：`docs/harness/talos97_source_structure_audit_2026-11-03.md`
- 未配对坐标重建边界：`experiments/talos97_rmsf_scalar/input_source_audit_20261103/RECONSTRUCTION_ZH.md`
- 标签/轨迹物理失稳审计：`experiments/talos97_rmsf_scalar/rmse_floor_diagnosis_20261103/REPORT_ZH.md`
