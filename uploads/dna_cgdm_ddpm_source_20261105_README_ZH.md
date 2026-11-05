# DNA-CGDM / DDPM 源码与复现说明

本包发布 DNA 折纸局部位移窗口生成模型 DNA-CGDM 的源码、必要上游实现、许可、配置和测试。它不含模型权重、训练状态、轨迹缓存或原始研究数据。

## 源码与入口

- `dna_cgdm/dna_model.py`：`DNACGDM`，共享结构编码器、监督统计头、上游 U-GNN/DDPM 噪声预测和 DDIM 采样。
- `dna_cgdm/train.py`：训练入口；`--arm D` 为当前联合训练路线，`B` 为 U-GNN 条件特征监督支路，`C` 为纯生成消融。
- `dna_cgdm/sample.py`：从 checkpoint 和设计特征/图输入采样，不读取 MD 轨迹。
- `dna_cgdm/prepare.py`、`bootstrap.py`、`project_data.py`、`datasets.py`：缓存校验、数据准备、划分和训练集归一化。
- `dna_cgdm/efficient_graph.py`：在保持上游逐帧运算的前提下分块执行稀疏图运算，降低内存占用。
- `dna_cgdm/evaluate_generated.py`、`dna_metrics.py`：生成位移窗统计评估。
- `dna_cgdm/export_positions.py`：将局部位移诊断导出到 oxDNA 位置格式；该输出不是完整刚体状态，也未经长时 MD 验证。
- `tests/dna_cgdm/`：模型、稀疏图运算、数据指标和校准测试。
- `vendor/graph-signal-diffusion/`：只包含本适配所需的上游 `src/graph_signal_diffusion/` 源码及其许可/项目信息。固定上游 commit：`018ac033e32105010fe2265e0bdef56db5fa74bb`，上游项目 MIT License。未修改上游仓库。

`MANIFEST.json` 列出每个文件的路径、字节数和 SHA-256；`UPSTREAM.json` 记录上游 URL、提交、许可证及文件范围。

## Harness 对应与当前状态

本包按项目 `AGENTS.md` 的可复现要求整理：目标/范围、固定配置、环境、运行命令、验证证据、数据依赖和已知限制均在此说明中；项目级任务状态及验证记录保存在主项目的 `feature_list.json` 和 `progress.md`。

该实验当前是 85 个严格 QC 设计上的探索性实现。历史划分为 69 train、16 validation、0 test；validation 已用于开发和 checkpoint 选择，不是独立测试集。唯一正式启动的 D 联合训练目标为 120 epochs；发布时状态文件显示 epoch 36，不能据此声称训练或生成质量评估已完成。配置 SHA-256：`81a6cf7d32762f7fa7fbedb176debdefc386ea5d0c0a5af9cef8e1d8b6adc401`。状态证据 `training_start_verification.json` 记录训练启动和 11 项测试通过；`pretraining_resume_verification.json` 记录最大图反向传播与 oxDNA 位置导出往返检查。smoke/probe 只验证执行和显存门，不是模型质量证据。

## 环境

验证环境：Python 3.11.12、PyTorch 2.5.1+cu124、PyTorch Geometric 2.5.2、CUDA 12.4；完整 pip 锁定列表见 `dna_cgdm/environment.lock.txt`。编译扩展 `torch-scatter` / `torch-sparse` 需匹配 torch/CUDA wheel；可参考 `dna_cgdm/requirements.txt` 中的 PyG wheel 提示。使用 NVIDIA A6000、A100 或类似显存卡；最大设计的优化 BF16 反向门峰值约 34.45 GB。GPU 训练需 CUDA 可用。

在项目根目录安装依赖后运行：

```bash
python -m pip install -r dna_cgdm/requirements.txt
# 按所用 torch/CUDA 版本从 PyG 官方 wheel 索引安装 torch-scatter 与 torch-sparse
python -m unittest discover -s tests/dna_cgdm -v
python -m compileall -q dna_cgdm tests/dna_cgdm
```

原验证环境的完整锁定依赖不是跨平台通用安装脚本；CUDA 扩展必须匹配本机 PyTorch 与 CUDA。

## 复现数据和运行

源码依赖项目相对路径中的以下历史数据，数据不在本源码包内：

- `experiments/directional_persistence_models_20261104/data/`：设计输入、映射、模态特征、固定 split 和标签加载兼容模块。
- `experiments/talos97_rmsf_scalar/continuation20m_20261102/labels/`：原始设计参考与 QC 元数据。
- `outputs/talos97_directional_persistence_20261104/labels/`：目标统计标签。
- 原始完整 oxDNA 轨迹：由 `prepare.py` 按 QC 清单读取并生成缓存；不得用 partial 缓存冒充完整缓存。

因此，源码 ZIP 单独解压后可以审阅、编译和运行合成测试，但不能独立重现训练。需要将其叠加到具有上述相对目录及轨迹的项目副本中。标签/输入归档提供部分标签与设计输入，但不含全部原始 MD 轨迹缓存。

在完整项目根目录执行：

```bash
python -m dna_cgdm.prepare --cache outputs/dna_cgdm_20261104/cache --workers 4
python -m dna_cgdm.bootstrap --output outputs/dna_cgdm_20261104/static --cache outputs/dna_cgdm_20261104/cache
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True python -m dna_cgdm.train \
  --static outputs/dna_cgdm_20261104/static --out outputs/my_joint_run \
  --epochs 120 --arm D --device cuda:0
```

每次运行使用一个新建的 `--out` 目录。训练会保存 config、上游 commit、源码哈希、缓存回执、归一化、状态、epoch 记录和 checkpoint。采样示例：

```bash
python -m dna_cgdm.sample --checkpoint outputs/my_joint_run/best.pt \
  --design-input outputs/dna_cgdm_20261104/static/6hb_001.npz \
  --output outputs/my_samples --samples 4 --device cuda:0
```

## 验证证据

打包前通过：

- `python -m unittest discover -s tests/dna_cgdm -v`：11 项通过（模型噪声公式、联合头梯度、无目标采样、U-GNN 监督路径、稀疏时间切片梯度、统计指标等）。
- `python -m compileall -q dna_cgdm tests/dna_cgdm`：源码与测试语法编译通过。
- 真实最大规模 9,996 节点、256 帧的反向更新/优化器门通过；峰值显存约 34.45 GB。
- oxDNA 位置诊断导出往返和取向正交性检查通过。该检查仅验证导出格式属性，不代表生成结构通过 oxDNA 动力学验证。

测试日志与门控结果的汇总副本在 `evidence/`。每个证据文件的字节数和哈希也列在 `MANIFEST.json`。

## 方法边界

模型输出为局部位移窗口 `(samples, window, nodes, 3)`，条件为静态设计图。它不预测核苷酸取向，不保证跨窗口连续性，不是 oxDNA 积分器，也不保证结构可直接接续 MD。局部框架相关向量尚未通过 Rij 边输运，协方差/持续性标签是有限轨迹统计而非已校准材料参数。没有冻结的家族级独立测试集，没有配对加固实验的 DeltaJ 外部验证；B/C 消融尚未完成。未进行训练质量或材料设计性能的结论。
