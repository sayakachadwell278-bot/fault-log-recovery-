# 最终运行核对表

- [ ] GitHub 仓库已创建为公共仓库，且 `.github/workflows` 已上传。
- [ ] `00 Preflight OpenTelemetry Docker environment` 显示成功。
- [ ] `01 Collect final labelled fault-log data` 显示成功。
- [ ] 下载的 `final-labelled-incidents` 中有 90 个 `manifest.json`。
- [ ] `data_audit_report.json` 显示 `n_eligible=90` 且 `n_errors=0`。
- [ ] `colab/00_train_offline.ipynb` 从头运行无报错。
- [ ] `models/proposed.joblib` 与 `models/proposed.json` 已上传到 GitHub 默认分支。
- [ ] `02 Compare real recovery policies` 显示成功。
- [ ] 下载的 `final-runtime-results` 中有 54 个 `verification.json`。
- [ ] `colab/01_analyze_runtime_results.ipynb` 从头运行无报错。
- [ ] 已保存 `paper_real_results.zip`、`final-labelled-incidents.zip` 和 `final-runtime-results.zip`。
- [ ] 论文只引用 `results/offline/` 和 `results/paper/` 中的真实输出。
