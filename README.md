# Day 2：筛查真实混凝土裂缝图像

本仓库为 Day 2 独立提交。目标：为设施维护团队制作一个初步图像筛查器，在真实裂缝图像上比较多数类基线与小型 CNN，并重点检查漏检裂缝。

**任务类型：** 图像二分类。输入是混凝土表面照片，输出是 `no_crack` / `crack`。

## 小组成员

- 张清波，U202412700（组长）
- 汤家齐，U202412691
- 吕绍康，U202412686

## 仓库文件

- `train.py`：真实数据检查、平衡划分、多数类基线、SmallCNN 训练与错误记录
- `models.py`：SmallCNN 结构
- `tests/test_models.py`：张量接口与拆分逻辑测试
- `report.md`：数据来源、方法、结果、失败与限制
- `presentation.pptx`：3 分钟答辩 PPT
- `submission.json`：提交清单
- `team.md`：成员名单

原始图像与运行产物（`runs/*.json`）不提交到 Git。

## 真实数据

- 所有者/发布者：Kaggle 用户 `arunrk7`
- 标题：Surface Crack Detection
- 页面：https://www.kaggle.com/datasets/arunrk7/surface-crack-detection
- 预期位置：`data/raw/Positive` 与 `data/raw/Negative`
- 预期规模：每类 20,000 张图像

只使用上述页面下载的真实图像。不要用 AI 生成图像替代；下载失败时联系教师取得同一来源缓存。

## 环境

本机验证环境为：

- Windows PowerShell
- Python 3.12
- 依赖见 `requirements.txt`（torch / torchvision / matplotlib）

其他版本范围内的环境也可能可运行，但请以你本机实际输出为准。

## 如何开始运行

以下命令都在仓库根目录执行，也就是能看到 `README.md` 与 `train.py` 的文件夹。

### 1. 取得代码

```powershell
git clone "https://github.com/Soup-Sky/ai-camp-2026-d2-ZhangQingboU202412700.git"
Set-Location "ai-camp-2026-d2-ZhangQingboU202412700"
```

应可看到：`Get-ChildItem` 列出 `train.py`、`models.py`、`tests`、`report.md`、`presentation.pptx`。

### 2. 安装依赖

```powershell
python -m pip install -r requirements.txt
```

### 3. 准备真实数据

从指定 Kaggle 页面下载并解压，然后放到：

```powershell
New-Item -ItemType Directory -Force data\raw
# 将 Positive / Negative 两个文件夹放到 data\raw\ 下
```

不要改类别文件夹名，不要提交原始大数据。

### 4. 检查真实数据

```powershell
python train.py --check-data
```

通过时应看到：

```text
REAL DATA CHECK PASSED
```

以及 Positive / Negative 各 20,000。失败则停止模型路线，检查当前目录、解压层级和来源。不要生成替代图像。

### 5. 运行测试

```powershell
python -m unittest discover -s tests -v
```

预期：

```text
Ran 3 tests
OK
```

注意：这些测试只证明模型接口与拆分逻辑，不证明真实图像评估。

### 6. 运行基线与候选

```powershell
python train.py --model baseline
python train.py --model cnn --epochs 2
```

应生成：

- `runs/baseline.json`
- `runs/cnn.json`
- 对应错误图像拼图（若有错误样本）

当前提交机若尚未放入完整 Kaggle 目录，`--check-data` 会失败；此时不应编造主结果数字。本仓库报告已按该边界如实记录。

## 方法摘要

- **基线：** 多数类预测（不看图像内容）
- **候选：** `models.py` 中的 SmallCNN（两段 Conv-ReLU-MaxPool + Linear）
- **主指标：** 裂缝召回率 `crack recall`，同时报告混淆矩阵与假阴性
- **边界：** 输出只用于安排人工复核，不能替代现场检查或工程师判断

## 限制

- 随机拆分高度相似的图像块可能造成泄漏，分数可能乐观
- 只报准确率会掩盖漏检风险
- 不能把筛查器结果写成结构安全结论
