# Day 2：真实混凝土裂缝筛查

本仓库是招商证券 AI 工程夏令营 Day 2 的独立提交。任务不是做“结构安全鉴定”，而是为设施维护团队提供照片初筛和人工复核优先级。

指定 Kaggle 数据已经完成真实检查：`Positive=20,000`、`Negative=20,000`。多数类基线和 SmallCNN 使用同一固定子集、同一划分与同一指标完成比较，所有提交数字都能由本页命令重新生成。

## 小组

- 张清波，U202412700（组长）
- 汤家齐，U202412691
- 吕绍康，U202412686

## 一眼看懂结果

固定条件：`seed=2026`，每类抽取 600 张，共 1,200 张；75/25 分层划分为 900 张训练、300 张测试；输入统一为 `3×64×64`。

| 方法 | Accuracy | Crack precision | Crack recall | FP | FN |
|---|---:|---:|---:|---:|---:|
| 多数类基线（并列时选 crack） | 50.0% | 50.0% | 100.0% | 150 | 0 |
| SmallCNN（2 epochs） | 81.3% | 89.2% | 71.3% | 13 | 43 |

混淆矩阵的行是真值 `[no_crack, crack]`，列是预测 `[no_crack, crack]`：

- 基线：`[[0,150],[0,150]]`
- SmallCNN：`[[137,13],[43,107]]`

最重要的结论不是“CNN 有 81.3% 准确率”，而是：CNN 显著减少误报并提高总体分类表现，但仍漏掉 43/150 张真实裂缝，不能直接用于免检或安全放行。

## 最容易答错的技术点

训练子集严格平衡，因此“多数类”实际发生并列。`Counter.most_common(1)` 按首次出现顺序打破平局，本次固定顺序选中 `crack`，于是基线把 300 张测试图全部判为裂缝。

这解释了一个看似反常的结果：

- 基线的 crack recall 是 100%，不是因为它理解了裂缝，而是因为它从不预测 `no_crack`。
- 它同时产生 150 个误报，所以高召回不等于整体系统更好。
- CNN 把 FP 从 150 降到 13，却产生 43 个 FN；这是误报与漏检之间的真实取舍。

## 仓库内容

| 路径 | 用途 |
|---|---|
| `train.py` | 数据契约检查、平衡拆分、基线、训练、指标与错误样本输出 |
| `models.py` | 两段 Conv-ReLU-MaxPool 的 SmallCNN |
| `tests/test_models.py` | 模型 shape 和拆分逻辑测试 |
| `report.md` | 数据来源、真实结果、失败案例、限制和证据边界 |
| `presentation.pptx` | 4 页真实结果答辩 PPT |
| `defense-kit/` | 逐页演讲稿、内容精讲、全面答辩知识点手册 |
| `submission.json` | 提交元数据和复现命令 |
| `team.md` | 小组成员 |

`data/raw/`、`runs/`、缓存和排版验收中间文件不会提交到 Git。

## 环境与安装

建议在独立虚拟环境中执行：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

主要依赖为 PyTorch、TorchVision、Pillow 和 Matplotlib。

## 准备真实数据

数据来源：

[Kaggle Surface Crack Detection](https://www.kaggle.com/datasets/arunrk7/surface-crack-detection)

解压后保持以下结构，不改类别名，不生成替代图片：

```text
data/
└── raw/
    ├── Negative/   # 20,000 images
    └── Positive/   # 20,000 images
```

先运行数据契约检查：

```powershell
python train.py --check-data
```

只有出现 `REAL DATA CHECK PASSED`，且两个类别都为 20,000 张，才继续模型路线。检查失败时应修复解压层级、来源或文件损坏问题，不能用生成图片补数量。

## 复现实验

先验证逻辑接口：

```powershell
python -m unittest discover -s tests -v
```

预期为 3 个测试全部通过。注意：单元测试只证明 shape 和拆分逻辑，不证明真实图像性能。

随后在同一目录运行：

```powershell
python train.py --model baseline
python train.py --model cnn --epochs 2
```

本地运行会在 `runs/` 中生成基线/CNN JSON 和错误图拼图。该目录被 Git 忽略，但它是核对报告数字的直接证据。

## 真实失败案例

`data/raw/Positive/09570.jpg` 的真值是 `crack`，模型预测为 `no_crack`。图中可以观察到深色、连续、弯折且肉眼明显的裂纹。

这张图能证明当前模型会漏掉明显裂缝；但单张图不能证明漏检由欠训练、纹理、位置或模型容量造成。原因必须通过受控实验验证，不能凭观察编造。

## SmallCNN shape 路线

```text
3×64×64
→ Conv 3→8 + ReLU + MaxPool
→ 8×32×32
→ Conv 8→16 + ReLU + MaxPool
→ 16×16×16
→ Flatten 4096
→ Linear 2 logits
```

`CrossEntropyLoss` 直接接收 logits；`argmax` 只实现固定决策，并没有针对漏检成本调阈值。

## 与仓库“学生学习教练”的差异

仓库中的 `student-learning-coach` 主要保护学习过程所有权：一次推动一个可观察动作，不替学生写讲稿或答案，证据不足时停止。

本提交额外完成了答辩表达层：

- 把概念、公式、代码 shape、真实 JSON、混淆矩阵和失败图串成一条可追问证据链。
- 明确解释多数类并列为何导致“50% accuracy + 100% recall + 150 FP”，而不是只机械报分。
- 提供逐页讲稿、40+ 刁钻问答、变式题、口误纠正和临场救急句。
- 保留教练的证据标准：不伪造运行结果，不把单张图写成因果，不把课堂筛查器夸大成工程安全系统。

这些差异来自三方对照：教练 `SKILL.md` 的职责边界、`train.py` 的实际实现、以及真实运行输出；不是为了刻意“写得不一样”而制造结论。

## 限制与下一步

- 数据集可能由大图切成相邻图块，随机拆分存在近重复泄漏风险。
- 当前结论只覆盖固定 1,200 张子集和两轮训练。
- 测试集不能反复用于挑 epoch 或阈值，否则会变成调参数据。
- 真实部署仍缺少按场地独立验证、拍摄规范、阈值校准、漂移监控和责任流程。

下一项最有价值的实验不是直接换更大模型，而是先按组建立训练/验证/测试协议，在验证集上预先用 crack recall 和人工复核量选择 epoch、类别权重或阈值，最后只使用一次保留测试集确认。

## 产品边界

输出只用于安排人工复核。它不能替代现场检查、工程师判断或安全决策。
