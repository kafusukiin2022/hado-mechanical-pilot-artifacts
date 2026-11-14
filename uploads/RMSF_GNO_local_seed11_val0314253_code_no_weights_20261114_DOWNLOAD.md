# RMSF 0.314253 nm 模型代码（无权重）

下载3个分卷，放同一目录，按顺序合并并解压：

```bash
cat RMSF_GNO_local_seed11_val0314253_code_no_weights_20261114.zip.part00{1,2,3} > RMSF_GNO_local_seed11_val0314253_code_no_weights_20261114.zip
unzip RMSF_GNO_local_seed11_val0314253_code_no_weights_20261114.zip -d RMSF_code
```

SHA256见同名DOWNLOAD.json。包含403个原工程文件：完整模型、训练、特征代码及配置、输入、标签、缓存、训练记录、预测和评估证据。按用户要求不含权重，不能直接加载checkpoint推理；可按README协议重训。历史seed11/local最佳epoch55，71训练设计排除015；17验证设计后处理RMSE 0.3142528666 nm，原18设计选模RMSE 1.6944933675 nm。ZIP CRC、原manifest哈希、Python语法、入口依赖导入、原标签复算验证通过。
