# 从这里开始

本包是论文 **Fault automatic detection and recovery mechanism based on log analysis in distributed systems** 的最终收束实验方案。

正式数据只有一套：OpenTelemetry Astronomy Shop 在 GitHub Actions Docker runner 中真实运行并采集的服务日志。包内没有预制“论文结果数据”，也没有任何虚构指标。

只做四件事：

1. 上传本包到一个 GitHub 公共仓库；
2. 在 GitHub Actions 点击运行 `00`、`01`；
3. 在 Colab 运行 `colab/00_train_offline.ipynb` 并上传导出的模型；
4. 在 GitHub Actions 点击运行 `02`，再在 Colab 运行 `colab/01_analyze_runtime_results.ipynb`。

完整细节见 `README_CN.md`。
