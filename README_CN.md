# Fault automatic detection and recovery mechanism based on log analysis in distributed systems

## 1. 研究范围已经锁定

本文只研究容器化微服务分布式系统中的服务级故障自动检测与恢复。论文题目不作任何改变。

唯一主线为：

> 多服务运行日志 → 故障检测 → 故障服务定位 → 故障类型识别 → 置信度安全门控 → 自动恢复 → 恢复验证。

系统为公开的 OpenTelemetry Astronomy Shop；日志是唯一动态模型输入。服务拓扑只作为静态依赖约束，健康检查和 HTTP 请求状态只用于恢复后的客观核验，不进入模型训练或推理。

正式实验不使用 RCAEval、系统指标、链路追踪、LLM、强化学习或额外数据集。这样避免研究对象、故障类型和恢复动作不一致。

## 2. 最终数据集

运行 `01 Collect final labelled fault-log data` 后，GitHub Actions 将生成最终训练、验证和离线测试数据。

| 项目 | 固定设置 |
|---|---|
| 开源系统 | OpenTelemetry Astronomy Shop |
| 固定源代码版本 | `f9ab699aa7e08074ae3ff1c814484838b87bb158` |
| 目标服务 | `cart`、`checkout`、`recommendation` |
| 故障类别 | `crash`、`stall`、`network` |
| 恢复动作 | 启动/重启、解除暂停/重启、重新接入网络 |
| 每个“服务×故障”组合 | 10 次独立重复 |
| 数据总量 | 3 × 3 × 10 = 90 个真实故障事件 |
| 训练/验证/测试划分 | 5/2/3 次，即 45/18/27 个事件 |

每个事件都有：

```text
incident_id
logs.csv
manifest.json
```

其中 `logs.csv` 是故障前基线、故障后观测窗口内由 Docker Compose 实际导出的容器日志；`manifest.json` 在离线阶段提供真实目标服务、故障类型、故障注入时间和数据划分。模型控制器不会读取该 manifest。

## 3. 模型与对比对象

本文机制使用归一化日志模板、错误/超时/连接异常比例、日志到达间隔、日志沉默程度和相邻服务异常聚合特征。模型输出：

```text
fault_probability
root_service
root_margin
fault_type
type_confidence
selected_action
```

只有满足故障检测阈值、故障类型置信度阈值和根因服务排序间隔阈值时，`proposed` 策略才会执行恢复。否则输出 `alert_only`。

离线对比：

1. B1：错误关键词阈值；
2. B2：TF-IDF + Logistic Regression；
3. B3：去除拓扑约束的日志模型；
4. Proposed：带拓扑约束和安全门控的日志恢复机制。

运行时恢复对比：

1. `alert_only`：只告警；
2. `always_restart`：检测到故障后重启预测服务；
3. `proposed`：本文的日志诊断与安全门控机制。

## 4. 论文正式保留的指标

| 研究环节 | 指标 |
|---|---|
| 故障检测 | F1、平均检测延迟 |
| 故障服务定位 | Top-1 Accuracy |
| 故障类型识别 | Macro-F1 |
| 恢复效果 | Recovery Success Rate、Median MTTR |
| 恢复安全性 | Unsafe Recovery Rate |

`Unsafe Recovery Rate` 定义为：发生自动动作，且动作目标服务错误或恢复验证失败的事件比例。所有恢复成功必须满足：目标服务健康、网页连续 3 次请求成功、并且在 70 秒验证期内保持可用。

## 5. 操作步骤

### 步骤 1：创建 GitHub 仓库

1. 解压本包。
2. 在 GitHub 新建一个公共仓库，例如 `fault-log-recovery`。
3. 将解压后的**全部文件和文件夹**上传到仓库根目录。`.github/workflows` 目录必须保留。
4. 进入仓库的 `Actions` 页面，确认 workflow 可以手动运行。
5. 不修改 `configs/experiment.yaml`、`configs/topology_otel.yaml`、故障类别、服务名、重复次数和随机种子。

### 步骤 2：先运行环境预检

在 GitHub Actions 页面选择：

```text
00 Preflight OpenTelemetry Docker environment
```

点击 `Run workflow`。只有页面显示绿色成功状态，才能继续。预检启动固定版本的 Astronomy Shop，并以 `curl http://localhost:8080/` 验证服务可访问。

### 步骤 3：生成最终真实日志数据

在 Actions 页面运行：

```text
01 Collect final labelled fault-log data
```

该工作流会自动完成 9 个矩阵任务，并在结束时合并为一个 artifact：

```text
final-labelled-incidents
```

下载这个单一 artifact，不要下载每一个 `raw-*` artifact。解压到 Google Drive：

```text
MyDrive/fault_log_recovery/final-labelled-incidents/
```

解压后应有 **90 个** `manifest.json`，另有 `data_audit_report.csv`。打开报告后必须满足：

```text
n_eligible = 90
n_errors = 0
```

任何其他结果都不能进入训练阶段。

### 步骤 4：在 Google Colab 训练并取得离线结果

1. 打开仓库中的 `colab/00_train_offline.ipynb`。
2. 上传到 Google Colab，或用 Colab 从 GitHub 打开。
3. 将第一段代码中的：

```python
REPO_URL = '<把这里替换为你的GitHub仓库HTTPS地址>'
```

替换为你的仓库 HTTPS 地址。

4. 将 `INCIDENT_DIR` 保持为步骤 3 的 Google Drive 解压位置。
5. 从头运行全部单元格。

该笔记本依次执行：数据审计、事件级窗口构建、三组基线、提出方法、拓扑消融、独立测试和 Bootstrap 95% 置信区间。

完成后下载：

```text
proposed_model_upload.zip
```

将压缩包中的两个文件上传到 GitHub 仓库的 `models/` 目录：

```text
models/proposed.joblib
models/proposed.json
```

上传后必须在 GitHub 网页确认两个文件存在。

### 步骤 5：执行真实恢复策略比较

在 GitHub Actions 页面运行：

```text
02 Compare real recovery policies
```

工作流会对每个“服务×故障”组合，在三个策略下执行两次独立事件：

```text
3 服务 × 3 故障 × 3 策略 × 2 次 = 54 个运行时恢复事件
```

控制器读取的仅是运行时日志、静态拓扑和 `models/proposed.joblib`；真实故障标签在控制器运行后才写入 manifest，用于离线评分和恢复验证。

结束后下载合并 artifact：

```text
final-runtime-results
```

解压到：

```text
MyDrive/fault_log_recovery/final-runtime-results/
```

解压后应有 **54 个** `verification.json`。

### 步骤 6：在 Colab 汇总真实恢复结果

1. 打开 `colab/01_analyze_runtime_results.ipynb`。
2. 同样设置 `REPO_URL`。
3. 将 `RUNTIME_DIR` 指向步骤 5 的解压目录。
4. 从头运行全部单元格。

该笔记本输出：

```text
results/offline/table_offline.csv
results/offline/test_metrics.json
results/offline/test_bootstrap_ci.csv
results/paper/table_recovery.csv
results/paper/table_recovery_bootstrap_ci.csv
results/paper/runtime_incident_records.csv
results/paper/fig_recovery_success_rate.png
```

这些才是论文第 4、5 章可以使用的真实结果。

### 步骤 7：保存最终实验数据包

从 Colab 下载两个文件并妥善保存：

```text
paper_real_results.zip
final-labelled-incidents.zip
final-runtime-results.zip
```

它们共同构成论文的最终实验数据包：原始真实日志、离线预测、动作决策、恢复验证记录、统计表和图。

## 6. 结果写入论文的硬性规则

1. 不能把冒烟测试结果、预期结果或手工估计值写入论文。
2. 不能使用训练集结果替代测试集结果。
3. 不能将 `force_restore.sh` 的清理行为计入自动恢复成功。
4. `alert_only` 只表示安全拒绝，不得被写作“恢复成功”。
5. 论文中要明确：恢复验证指标不属于模型输入。
6. 任何 Actions 任务失败、日志为空、审计不通过或模型文件缺失时，必须修复并重新运行，不能补写结果。

## 7. 本地代码冒烟测试

以下测试只检查 Python 脚本是否能联通，不能生成论文结果：

```bash
pip install -r requirements-colab.txt
python scripts/smoke_test.py
```

应输出：

```text
SMOKE_TEST_PASS
```

## 8. 常见失败处理

| 现象 | 处理方式 |
|---|---|
| `00` 预检失败 | 打开 workflow 日志，确认镜像下载和端口 8080；不要继续运行 `01` |
| `n_eligible` 不等于 90 | 不训练；检查失败矩阵任务并重跑整个 `01` 工作流 |
| Colab 显示 `No usable incident windows` | 检查是否下载了 aggregate artifact，以及 `logs.csv` 与 `manifest.json` 是否在同一事件目录 |
| `models/proposed.joblib` 不存在 | 重新运行 Colab 训练笔记本并上传模型文件 |
| `02` 失败 | 首先检查 `models/` 中模型是否已提交到仓库默认分支 |
| `verification.json` 少于 54 | 不写结果，重跑失败矩阵任务或整个 `02` 工作流 |

## 9. 已知边界

本文只针对服务退出、服务阻塞和服务网络隔离三类低风险、可回滚服务级故障。数据库一致性损坏、分布式事务回滚、恶意入侵、持久化数据破坏和硬件永久故障不属于本研究的自动恢复范围。
