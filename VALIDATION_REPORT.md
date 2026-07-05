# 交付前验证记录

## 已完成的静态验证

- Python 源码编译检查；
- 两个 GitHub Actions 工作流 YAML 解析检查；
- 合成日志端到端冒烟测试：数据构建、标签隔离、训练、拓扑消融、预测和事件级指标计算；
- 所有训练特征在构造后才附加标签，避免 `is_fault`、`is_root`、真实故障类型和注入时间泄漏进入模型；
- 控制器接口不接收 `manifest.json` 或真实目标服务、真实故障类型字段；
- Actions 聚合步骤使用单一最终 artifact，减少人工下载错漏。

## 未在当前环境执行的部分

当前交付环境没有 Docker，因此无法实际启动 Astronomy Shop、注入容器故障或产生恢复结果。GitHub Actions Docker runner 是本包指定的真实执行环境。其实际 artifact 是唯一可用于论文结果的证据。

## 使用限制

不要把本报告视为真实实验成功记录。真实实验必须完成 `00`、`01`、Colab 训练、`02` 和最终结果汇总后才成立。

## 本地执行结果

- `scripts/smoke_test.py` 输出 `SMOKE_TEST_PASS`；
- `scripts/validate_incidents.py` 在 90 个合成结构化事件上输出 `DATA_AUDIT_PASS`；
- `scripts/build_dataset.py`、`scripts/run_baselines.py`、`scripts/evaluate_models.py` 已完成链路冒烟；
- `scripts/analyze_runtime_results.py` 已对模拟格式的恢复 artifact 成功生成 CSV 和 PNG。

上述合成结构化事件仅用于验证代码路径和字段约束，已在最终压缩包中删除，不能作为论文数据。
