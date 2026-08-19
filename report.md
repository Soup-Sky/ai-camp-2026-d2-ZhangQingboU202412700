# Day 2 证据报告

## 1. 本日问题

- 里程碑：D2
- 学生或小组：张清波组
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
- 实际检查结果：**BLOCKED**。当前提交副本没有完整原始图像目录；未写入任何生成图像或伪造计数。
- 小测试限制：`tests/test_models.py` 的张量和拆分测试只证明逻辑接口，不证明真实图像评估。

## 3. 可复现运行

```powershell
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
python train.py --check-data
python train.py --model baseline
python train.py --model cnn --epochs 2
```

逻辑测试实际结果：3/3 通过。真实数据检查必须在 Kaggle 原始目录可用后运行；未生成 `runs/*.json` 作为主结果，也未编造准确率或召回率。

## 4. 基线与候选

### 简单基线

- 方法：训练子集多数类预测
- 为什么足够简单：不依赖图像纹理，提供“只看类别比例”的最低参照
- 结果：待真实数据检查通过后记录 `runs/baseline.json`

### 候选方法

- 核心改动：`models.py` 完成给定的 `Conv2d → ReLU → MaxPool2d` 两段和二分类线性层
- 保持不变：固定 seed、平衡子集、75/25 划分、64×64 输入和混淆统计
- 结果：待真实数据检查通过后记录 `runs/cnn.json`

## 5. 一个真实失败案例

- 当前状态：BLOCKED。没有真实图像就不能填写样本位置、真值或观察，不能用合成图像冒充。
- 取得数据后的最小动作：从 `runs/cnn.json` 的 `first_errors` 选择一张真实 false negative，用图像路径、真值、预测、可观察特征和下一项检查填回本节。

## 6. 智能体与学生工作边界

- 智能体协助解释代码结构、SmallCNN 局部实现建议和测试检查。
- 学生仍需核对真实数据来源、运行输出、失败图像、报告数字和 Git diff；不能把本报告中的 BLOCKED 改成“通过”来提交。

## 7. 结论与限制

目前能支持的最小结论是：给定 SmallCNN 的结构实现正确，逻辑接口测试通过。当前不能支持任何真实裂缝识别性能结论，因为指定数据未在本机完成检查。下一项最小检查是取得同一 Kaggle 来源的完整目录并运行 `--check-data`。即使真实评估通过，输出仍只能安排人工复核。相关图像泄漏、拍摄环境变化和漏检成本需要在答辩中明确。

## 8. 提交复核

- [x] 源码和逻辑测试通过
- [ ] 真实数据检查与主程序重新运行
- [ ] 报告数字与保存输出一致（待真实运行后）
- [x] PPT 已生成，主结果位置明确标为待真实数据
- [x] 无密钥、大数据、私人信息或生成替代数据
- [x] 已建立 Day 2 独立 GitHub 仓库
