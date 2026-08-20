# Day 2 证据报告

## 1. 本日问题

- 里程碑：day-02
- 学生或小组：张清波组（张清波 U202412700、汤家齐 U202412691、吕绍康 U202412686）
- 使用者：需要先筛选大量照片、再安排人工复核的设施维护团队
- 真实输入：Kaggle Concrete Crack Images 的 Positive / Negative 图像
- 需要的输出：初步筛查标签与可打开的错误图像列表
- 与使用者最相关的错误：真实裂缝被判为 no_crack（false negative）
- 本日产品边界：只用于安排人工复核，不能替代现场检查、工程师判断或安全决策

## 2. 真实数据或真实课程输入

- 所有者/发布者：Kaggle 数据集发布者 arunrk7
- 标题：Surface Crack Detection
- 原始 URL：https://www.kaggle.com/datasets/arunrk7/surface-crack-detection
- 预期文件与结构：`data/raw/**/Positive` 与 `data/raw/**/Negative`，各 20,000 张图像
- 检查命令：`python train.py --check-data`
- 实际检查结果：**PASSED**。`Negative=20,000`、`Positive=20,000`，类别目录和后缀检查通过；没有使用生成图像替代评估数据。
- 小测试限制：`tests/test_models.py` 的张量和拆分测试只证明逻辑接口，不证明真实图像评估。

## 3. 可复现运行

```powershell
cd G:\AI-SUMMER\ai-summer-camp-2026\student-work\day-02-concrete
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
python train.py --check-data
python train.py --model baseline
python train.py --model cnn --epochs 2
```

逻辑测试实际结果：3/3 通过。真实数据检查、基线与 CNN 均已运行，原始证据写入 `runs/baseline.json`、`runs/cnn.json` 和错误拼图；这些运行产物不提交，但可由上述命令重建。

## 4. 基线与候选

### 简单基线

- 方法：训练子集多数类预测
- 为什么足够简单：不依赖图像纹理，提供“只看类别比例”最低参照
- 实际行为：训练子集严格平衡，所谓“多数类”并列；在固定 seed 和当前顺序下，`Counter.most_common(1)` 选择 crack，因此所有测试图都判为 crack。
- 结果：accuracy=50.0%，crack precision=50.0%，crack recall=100.0%；混淆矩阵（行是真值 no_crack/crack，列是预测 no_crack/crack）为 `[[0,150],[0,150]]`。它没有漏检，但把 150 张无裂缝图全部送去复核。

### 候选方法

- 核心改动：`models.py` 完成给定的 `Conv2d → ReLU → MaxPool2d` 两段和二分类线性层
- 保持不变：固定 seed、平衡子集、75/25 划分、64×64 输入和混淆统计
- 结果：accuracy=81.3%，crack precision=89.2%，crack recall=71.3%；混淆矩阵为 `[[137,13],[43,107]]`，共 43 个 false negative。
- 训练证据：两轮平均训练损失从 0.6818 降至 0.6186。损失下降只说明训练目标在改善，不证明模型已经适合安全筛查。
- 公平比较：两者使用相同 seed=2026、每类 600 张、900/300 的 75/25 划分和同一标签映射。
- 关键取舍：CNN 将 accuracy 提高 31.3 个百分点、减少大量误报，却把基线的 crack recall 从 100% 降至 71.3%。因此它在总体分类上更强，但不能说在“尽量不漏裂缝”的筛查目标上全面优于基线。

## 5. 一个真实失败案例

- 真实假阴性：`data/raw/Positive/09570.jpg`，真值 crack，预测 no_crack。
- 可观察特征：64×64 图像中存在从左上向右下延伸的深色、连续、弯折裂纹，和浅灰混凝土背景对比明显。这不是“裂纹太细所以肉眼看不到”的案例。
- 能说明：两轮训练的 SmallCNN 即使面对肉眼明显裂纹也可能漏检，当前模型不能直接作为免检依据。
- 不能说明：单张图不能证明漏检的因果原因。欠训练、抽样、纹理、位置或模型容量都只是待验证假设。
- 下一项最小检查：在训练段内部增加验证集并预先规定 recall 优先的选择规则，再比较更多 epoch、类别加权或决策阈值；保留测试集只做一次最终确认。

## 6. 智能体与学生工作边界

- 智能体帮助追踪代码、生成学习与答辩材料，并指出多数类并列、指标冲突和证据边界。
- 学生需能亲自解释数据检查、shape 流、混淆矩阵、真实失败图像和为什么不能把筛查输出当作结构安全结论。

## 7. 结论与限制

在固定真实子集上，SmallCNN 的总体准确率和精确率明显优于全判裂缝基线，但 crack recall 只有 71.3%，漏掉 43/150 张裂缝。最小诚实结论是：该模型可以作为课堂级候选分类器，尚不能作为实际免检系统。真实部署前还必须处理相邻图块泄漏、独立场地验证、阈值/校准、拍摄条件变化和人工复核责任。

## 8. 提交复核

- [x] 源码和逻辑测试通过
- [x] 真实数据检查与基线/CNN 主程序运行
- [x] 报告数字与保存输出一致
- [x] PPT、讲稿和知识手册更新为真实结果，并完成逐页视觉检查
- [x] 无密钥、大数据、私人信息或生成替代数据
- [x] 更新独立 GitHub 仓库并核对远程内容
