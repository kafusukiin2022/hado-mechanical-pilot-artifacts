# TALOS97 模型诊断交接包（总上传大小小于100 MB）

下载：[单个ZIP压缩包](./talos97_agent_handoff_100mb_20261030.zip)；[SHA256及核验记录](./talos97_agent_handoff_100mb_20261030_PACKAGE.json)。

ZIP大小：83,874,752 bytes；SHA256：`15283984c0ecd4e6cca684ceaee688b53edc234a5bc826dedbe2f2839000de93`。

包含14设计真实NPZ（8train/6validation）、三个seed原模型与Phase11权重、实际模型和训练代码、诊断报告和JSON/CSV结果、CPU独立核验入口。无测试数据。

精简删除：全部诊断激活缓存、probe/对照临时权重和NPZ、Phase1大型中间NPZ、生成的3D HTML/VTP。保留分析与可视化脚本，可重新生成；本包不支持完整68/14重训。

解压后先让Agent阅读 `README_AGENT_ZH.md` 与 `NEXT_AGENT_PROMPT_ZH.md`。创建Python环境并安装 `requirements-minimal.txt`，运行：

```bash
python portable_verify.py --output portable_verification.json
```

CPU核验42组输出已通过，最大坐标差1.06e-5nm；完整文件SHA256清单在ZIP内。目标是独立诊断跨设计域质心平移，连接点编码不是已证实的解决方案。
