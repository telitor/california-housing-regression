California Housing · 房价预测 Price Prediction

线性回归基线 → 特征工程 → 系统性消融实验
Linear Regression baseline → feature engineering → systematic ablation study

Show Image
Show Image
Show Image

项目概述 Overview
本项目在 加州房价数据集 上对线性回归进行基准测试，分为两个阶段：
A structured machine learning project that benchmarks Linear Regression on the California Housing Dataset. The project is split into two phases:

基线模型 baseline_model.py — 完整流水线：加载 → 过滤 → 拆分 → 训练 → 评估 → 可视化
Baseline (baseline_model.py) — clean pipeline: load → filter → split → train → evaluate → visualize
消融实验 sklearn（plus）.py — 跨 StandardScaler × PolynomialFeatures(degree=2) 的 2×2 因子实验，每个配置进行 5 折交叉验证
Ablation Study (sklearn（plus）.py) — 2×2 factorial experiment across StandardScaler × PolynomialFeatures(degree=2), with 5-fold cross-validation on each configuration


实验结果 Results
配置 Config标准化 Scaler多项式次数 DegreeMSE ↓R² ↑CV ScoreA✗1———B✓1———C✗2———D✓2———

运行 sklearn（plus）.py 以填充表格数据。
Run sklearn（plus）.py to populate the table.


可视化结果 Visualizations
基线模型 — 真实值 vs. 预测值 Baseline — True vs. Predicted
散点图对比真实房价（x 轴）与预测房价（y 轴）。对角线代表完美预测，点云越紧密则模型表现越好。
The scatter plot compares true house prices (x-axis) vs. predicted prices (y-axis). The diagonal represents a perfect predictor — tighter clustering means better performance.
<img src="true_vs_pred.png" width="500"/>

消融实验 — 四配置对比 Ablation Study — 4-Config Comparison
每个子图对应一种实验配置，通过对比四个面板可以观察标准化与多项式特征扩展的独立效果及叠加效果。
Each subplot corresponds to one experimental configuration. Comparing the four panels reveals the independent and combined effects of scaling and polynomial feature expansion.
<img src="true_vs_pred（plus）.png" width="780"/>
图表布局说明 Reading the grid:
一次多项式 Degree = 1二次多项式 Degree = 2无标准化 No Scaler左上 Top-left左下 Bottom-left有标准化 With Scaler右上 Top-right右下 Bottom-right

二次多项式特征（Degree=2）明显使点云向对角线收紧，尤其在中间价格区间（1.5–3.5）。标准化单独使用对线性回归影响有限，但与多项式特征结合时效果显著提升。
Degree=2 visibly tightens the point cloud around the diagonal, especially in the mid-price range (1.5–3.5). Scaler alone has minimal effect on Linear Regression — its benefit compounds when combined with polynomial features.


项目结构 Project Structure
.
├── baseline_model.py          # 基线流水线：数据 → 训练 → 评估 → 绘图 | Baseline pipeline
├── sklearn（plus）.py          # 消融实验：4 种配置 × (MSE, R², CV) | Ablation study
├── true_vs_pred.png           # 基线散点图 | Scatter plot from baseline
├── true_vs_pred（plus）.png    # 消融实验 2×2 子图 | 2×2 subplot from ablation study
└── README.md

快速开始 Quickstart
bash# 安装依赖 | Install dependencies
pip install scikit-learn numpy matplotlib

# 运行基线模型（生成 true_vs_pred.png）| Run baseline
python baseline_model.py

# 运行消融实验（生成 2×2 对比图）| Run ablation study
python "sklearn（plus）.py"

流水线 Pipeline
fetch_california_housing()
        │
        ▼
  过滤 Filter: y < 5.0        ← 移除封顶/异常值 removes capped/outlier values
        │
        ▼
  train_test_split             ← train_size=0.2, random_state=42
        │
        ├──[plus only]──► PolynomialFeatures(degree=d)
        │
        ├──[plus only]──► StandardScaler()
        │
        ▼
  LinearRegression.fit()
        │
        ▼
  .predict()  ──►  MSE, R², cross_val_score(cv=5)
        │
        ▼
  散点图（真实值 vs. 预测值）Scatter plot (true vs. predicted)

关键设计选择 Key Design Choices
train_size=0.2 — 有意使用较小的训练集来压测模型的泛化能力。生产模型应使用更大的训练比例，此处为实验目的。
Deliberately small training set to stress-test generalization. Production models should use a larger split; this is intentional for experimentation.
价格上限过滤 Price cap filter (y < 5.0) — 数据集将价格截断至 5.0，形成人为密集的上限区域。移除这些样本可降低标签噪声，防止模型将上限当作特征学习。
The dataset caps values at 5.0, creating artificial density at the ceiling. Removing these reduces label noise and prevents the model from learning the cap as a feature.
消融轴 Ablation axes

StandardScaler — 线性回归理论上对尺度不敏感，但多项式特征在归一化输入上可避免数值不稳定。
Linear Regression is scale-invariant in theory, but polynomial features benefit from normalized inputs to avoid numerical instability.
PolynomialFeatures(degree=2) — 引入交互项（如 income × rooms），使线性模型能够捕捉非线性关系，代价是特征维度急剧增加。
Introduces interaction terms (e.g., income × rooms), allowing the linear model to capture non-linear relationships at the cost of feature explosion.


评估指标参考 Metrics Reference
指标 Metric公式 Formula说明 InterpretationMSEmean((y - ŷ)²)平均平方误差，对大偏差惩罚更重，越低越好。 Average squared error; penalizes large mistakes heavily. Lower is better.R²1 - SS_res / SS_tot方差解释比例。1.0 = 完美；0 = 仅预测均值。 Proportion of variance explained. 1.0 = perfect; 0 = predicts the mean.CV Score5 折交叉验证 R²估计泛化能力，方差大则提示过拟合。 Estimates generalization. High variance here signals overfitting.

注意事项 Notes

全程使用 random_state=42 确保可复现性。 random_state=42 is set throughout for reproducibility.
消融实验中的交叉验证仅在训练集上运行——模型选择过程中不接触测试数据。
Cross-validation in the ablation study runs on the training split only — test data is never touched during model selection.
有意排除三次及以上多项式（degree=3+），以避免在 train_size=0.2 下出现严重过拟合。
Polynomial degree=3+ is intentionally excluded to avoid severe overfitting at train_size=0.2.
