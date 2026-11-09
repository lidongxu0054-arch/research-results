# WSSF 多模态遥感影像匹配加速

「面向多模态遥感影像匹配的 WSSF 自适应计算与加速方法研究」的实验结果。在 204 对影像上对基线及 A / B / C 三个叠加版本做消融。论文全文、四版源码、影像数据与运行日志保存在私有仓库 `WSSF-Matching-Acceleration`；本目录只公开结果说明与 CSV。

## 实验口径

- **样本**：204 对 = 原 84 对 + MapData 前 120 对。MapData 的 L/R 灰度相关约 0.082，属典型跨模态，SURF 在该数据上失效，不能用它判断影像方向。
- **版本关系**（逐层叠加，不是并列方案）：基线 WSSF2023 → A（SURF 粗估尺度，非对称截断金字塔层数）→ B（+ Sobel 结构 ROI，pct=99 / pad=8 / tile=128）→ C（+ GPU 实现层加速，算法逻辑一行未改）。
- **环境**：MATLAB R2023b，parpool 14 workers，RTX 4050 Laptop。
- **字段**：`time_s` 单对耗时；`NCM` 正确匹配点数；`RCM` 正确匹配率；`RMSE` FSC/RANSAC 重投影误差；`SR` 匹配成功率；`gtErr` 估计 H 与真值在 17×17 网格上的中位位移（见 `csv/mapdata_gt_check.csv`）。A / B / C 版另有 `s_est`、`n_est`、`roi_frac` 等中间量。

## 核心结果（204 对）

| 版本 | 耗时 s/对 | 加速比 | 正确匹配点数 NCM | 重投影误差 RMSE | 真值误差 gtErr(px) |
|---|---|---|---|---|---|
| 基线版 baseline | 15.82 | — | 276.46 | 1.9618 | 1.80 |
| A 版 | 9.67 | 1.64× | 377.63 | 1.9475 | 1.28 |
| B 版 | 7.55 | 2.10× | 179.55 | 1.9060 | 1.54 |
| C 版 | 5.72 | 2.77× | 178.99 | 1.8887 | 1.62 |

SR 全程 100%。关闭 GPU 后复核为 8.71 s/对，与 B 版统计等价，说明 C 版的收益来自实现层而非算法改动。MapData 120 对单独统计为 15.65 → 9.35 → 6.90 → 5.14 s/对（3.04×），逐对 0/120 变慢。

**口径提醒**：表中 `RMSE` 是重投影误差，不是真值误差；`gtErr` 才是相对真值的偏差，两者不能混用。`FSC.m` 的 RANSAC 随机种子未固定，同一份代码重跑会有逐对波动，严格复现需先固定种子。新增数据须沿用同一口径才能追加比对。

## 结果文件

| 文件 | 行数 | 内容 |
|---|---|---|
| `csv/baseline.csv` | 204 | 基线主结果 |
| `csv/ablation_A.csv`、`ablation_B.csv`、`ablation_C.csv` | 各 204 | A / B / C 三版主结果 |
| `csv/baseline_84pairs_backup.csv`、`ablation_A_84pairs_backup.csv`、`ablation_B_84pairs_backup.csv`、`ablation_C_84pairs_backup.csv` | 各 84 | 合并 204 对之前的 84 对记录，供口径对照 |
| `csv/baseline_mapdata.csv` | 120 | 基线在 MapData 120 对上的单独结果 |
| `csv/ablation_A_scale_scan.csv` | 84 | A 版尺度比扫描，用于确定金字塔截断阈值 |
| `csv/ablation_C_gpuoff_verify.csv` | 204 | 关闭 GPU 的复核结果 |
| `csv/mapdata_gt_check.csv` | 120 | 真值校验：RANSAC 匹配数、重投影误差、真值误差、方向判定 |

共 12 个 CSV、约 305 KB，UTF-8 无 BOM，逗号分隔，首行为表头。

## 公开版处理说明

原始记录中 `ref_file` / `sensed_file` 两列是本机的绝对路径。公开版把路径前缀（`<盘符>:\…\data\`）统一替换为相对路径 `data/`，并将反斜杠改为正斜杠。**除这两列的路径写法外，所有字段、数值与行序均与原始记录一致**，未做重算、删行或单位换算。

引用请注明本仓库与项目目录；论文全文、代码与影像数据不在本仓库内。
