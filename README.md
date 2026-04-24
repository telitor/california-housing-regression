# 加州房价 · 价格预测

> Linear Regression 基线模型 → 特征工程 → 系统性消融实验

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)

---

## 项目简介

本项目在 [California Housing Dataset](https://scikit-learn.org/stable/datasets/real_world.html#california-housing-dataset) 上对 Linear Regression 进行系统性基准测试，分为两个阶段：

1. **基线模型** (`baseline_model.py`) — 标准流水线：加载 → 过滤 → 划分 → 训练 → 评估 → 可视化
2. **消融实验** (`sklearn（plus）.py`) — 在 `StandardScaler` × `PolynomialFeatures(degree=2)` 两个维度上进行 2×2 因子实验，每种配置均使用 5-fold cross-validation

---

## 实验结果

| 配置 | Scaler | Degree | MSE ↓ | R² ↑ | CV Score |
|------|--------|--------|-------|------|----------|
| A | ✗ | 1 | — | — | — |
| B | ✓ | 1 | — | — | — |
| C | ✗ | 2 | — | — | — |
| D | ✓ | 2 | — | — | — |

> 运行 `sklearn（plus）.py` 以填充表格数据。

---

## 可视化结果

### 基线模型 — 真实值 vs. 预测值

散点图对比真实房价（x 轴）与预测房价（y 轴）。对角线代表完美预测——点云越紧密，模型性能越好。

<img src="true_vs_pred.png" width="500"/>

---

### 消融实验 — 四配置对比图

每个子图对应一种实验配置，通过对比四幅图可以观察 scaling 与 polynomial feature expansion 各自及组合的效果。

<img src="true_vs_pred（plus）.png" width="780"/>

**图表布局说明：**

| | Degree = 1 | Degree = 2 |
|---|---|---|
| **无 Scaler** | 左上 | 左下 |
| **有 Scaler** | 右上 | 右下 |

> Degree=2 明显使点云向对角线收拢，尤其在中间价格区间（1.5–3.5）效果最为显著。单独使用 Scaler 对 Linear Regression 的提升有限——其价值在与 polynomial features 组合时才得到充分体现。

---

## 项目结构

```
.
├── baseline_model.py          # 基线流水线：数据 → 训练 → 评估 → 绘图
├── sklearn（plus）.py          # 消融实验：4 种配置 × (MSE, R², CV)
├── true_vs_pred.png           # 基线模型散点图
├── true_vs_pred（plus）.png    # 消融实验 2×2 子图
└── README.md
```

---

## 快速开始

```bash
# 安装依赖
pip install scikit-learn numpy matplotlib

# 运行基线模型（生成 true_vs_pred.png）
python baseline_model.py

# 运行消融实验（生成 2×2 对比图）
python "sklearn（plus）.py"
```

---

## 流水线

```
fetch_california_housing()
        │
        ▼
  过滤：y < 5.0              ← 去除封顶值/异常值
        │
        ▼
  train_test_split            ← train_size=0.2, random_state=42
        │
        ├──[plus 版本]──► PolynomialFeatures(degree=d)
        │
        ├──[plus 版本]──► StandardScaler()
        │
        ▼
  LinearRegression.fit()
        │
        ▼
  .predict()  ──►  MSE, R², cross_val_score(cv=5)
        │
        ▼
  散点图（真实值 vs. 预测值）
```

---

## 关键设计决策

**`train_size=0.2`** — 刻意使用较小的训练集以压力测试模型的泛化能力。生产环境应使用更大的划分比例；此处为实验性设定。

**价格上限过滤（`y < 5.0`）** — 数据集中价格被封顶在 5.0，导致该值附近出现人为的数据堆积。移除这些样本可降低标签噪声，防止模型将封顶值当作特征来学习。

**消融维度**
- *StandardScaler* — Linear Regression 理论上对量纲不敏感，但 polynomial features 引入的高次项在归一化输入下可避免数值不稳定。
- *PolynomialFeatures(degree=2)* — 引入交互项（如 `income × rooms`），使线性模型能够捕捉非线性关系，代价是特征维度大幅膨胀。

---

## 指标说明

| 指标 | 公式 | 含义 |
|------|------|------|
| **MSE** | `mean((y - ŷ)²)` | 平均平方误差；对大误差惩罚更重。越低越好。 |
| **R²** | `1 - SS_res / SS_tot` | 方差解释比例。1.0 = 完美拟合；0 = 仅预测均值。 |
| **CV Score** | 5-fold cross-validated R² | 估计泛化能力。方差过大说明存在过拟合。 |

---

## 备注

- 全程设置 `random_state=42` 以保证可复现性。
- 消融实验中的 cross-validation 仅在**训练集**上运行——模型选择阶段不接触测试集。
- 刻意不使用 degree=3 及以上，以避免在 `train_size=0.2` 的设定下出现严重过拟合。
