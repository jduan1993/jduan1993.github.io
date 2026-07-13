---
title: 金融风控模型
---

## 1. One-Hot Encoding 和 WoE (Weight of Evidence)

### One-Hot Encoding（独热编码）

#### 定义  
One-Hot Encoding 是一种将分类变量（categorical variable）转换成数值变量的方式。它将每个类别表示为一个 二进制向量，其中只有该类别对应的位置是 1，其他位置都是 0。

#### 例子
假设你有一个“颜色”特征，有三个类别：红色、绿色、蓝色。
| 颜色 | One-Hot Encoding |
| -- | ---------------- |
| 红色 | [1, 0, 0]        |
| 绿色 | [0, 1, 0]        |
| 蓝色 | [0, 0, 1]        |

#### 优点
* 不引入大小关系（不像直接用 1,2,3 表示，会有误导模型认为红色 < 绿色 < 蓝色）。
* 适合大多数机器学习模型（如线性回归、树模型也可以，但可能更浪费内存）。

#### 缺点
* 类别很多时，会导致维度急剧增加（“维度灾难”）。

### WoE (Weight of Evidence, 信息权重)

#### 定义
[WoE](#woe-转换的作用) 是金融风控中常用的特征转换方法，主要用于 二分类问题（比如坏账/好账）。它衡量某个类别对目标变量的“预测能力”，通过对类别与事件发生概率的对数比值进行转换。

#### 公式
{{< katex display=true >}}
WoE_i = \ln\frac{P({Good}_i)}{P({Bad}_i)}
{{< /katex >}}

其中：  
好样本比例 = 该类别/分箱的好样本数 ÷ 总好样本数  
坏样本比例 = 该类别/分箱的坏样本数 ÷ 总坏样本数

#### 例子
| 年龄区间  | 好客户 | 坏客户 | WoE               |
| ----- | --- | --- | ----------------- |
| 20-30 | 50  | 10  | ln(50/总好 ÷ 10/总坏) |
| 31-40 | 40  | 20  | ln(40/总好 ÷ 20/总坏) |
| 41-50 | 30  | 30  | ln(30/总好 ÷ 30/总坏) |

#### 优点
* [将类别变量转换为连续变量，适合逻辑回归。](#逻辑回归)
* WoE 与目标变量的单调性可以帮助构建更稳定的模型。
* 可以用于 IV (Information Value) 计算，衡量变量的重要性。

#### 缺点
* 主要适用于二分类问题。
* 分箱方法需要经验或算法支持（比如等频、决策树分箱）。

### 总结对比
| 特征编码方法           | 用途           | 特点                   | 局限性          |
| ---------------- | ------------ | -------------------- | ------------ |
| One-Hot Encoding | 将分类变量转为二进制向量 | 简单直观，无顺序假设           | 类别多时维度大      |
| WoE              | 金融/二分类特征转换   | 连续变量，便于逻辑回归建模，可计算 IV | 仅适用于二分类，需要分箱 |

## DPD

#### 定义
DPD = Days Past Due，中文通常叫“逾期天数”。
* 它表示 贷款或信用账户已经逾期未还款的天数。
* 常用来衡量客户的还款行为和信用风险。

#### 举例说明
假设贷款到期日是 10 月 1 日：
| 日期        | 实际还款 | 逾期天数 (DPD) |
| --------- | ---- | ---------- |
| 10 月 1 日  | 已还款  | 0          |
| 10 月 5 日  | 未还款  | 4          |
| 10 月 12 日 | 未还款  | 11         |

* DPD = 0 → 正常还款
* DPD = 1~29 → 轻微逾期
* DPD = 30~59 → 中度逾期
* DPD ≥ 90 → 高度逾期 / 不良贷款

> 注意：不同银行或金融机构的分界标准可能略有差异。

#### DPD 在风控中的用途
1. 信用评分建模
    * 逾期天数越多 → 风险越高 → 信用评分越低。
2. 贷款分类
    * 根据 DPD 可以将贷款分为正常、关注、次级、可疑、损失五类。
3. 催收策略
    * 不同 DPD 阶段，采取不同催收手段（短信提醒、电话催收、法律追讨）。
4. 违约率计算
    * 通常用 DPD ≥ 90 天来定义“违约”（default）。

## 随机森林

#### 定义
随机森林是一种 **集成学习**（Ensemble Learning）方法，通过组合多个决策树来进行分类或回归。
* 核心思想：“多颗树一起投票，比单颗树更稳健”
* 分类任务：用多数投票决定最终类别
* 回归任务：取所有树预测值的平均

#### 工作原理
随机森林主要有两个随机性来源，所以叫 **随机森林**：

1. Bagging（Bootstrap Aggregating）
    * 对训练数据做 **有放回抽样**，生成多份不同的训练子集
    * 每个子集训练一棵决策树
    * 这样每棵树都略有差异，降低过拟合

2. 特征随机选择
    * 每棵树在分裂节点时，不用考虑所有特征，而是 **随机选择一部分特征**
    * 增加树之间的差异性，提升模型的泛化能力

#### 举例
假设我们要预测是否会违约（是/否），有 3 棵树：
| 树编号 | 预测结果 |
| --- | ---- |
| 树1  | 是    |
| 树2  | 否    |
| 树3  | 是    |
* 最终随机森林结果 = 多数投票 = “是”

#### 优点
1. 抗过拟合：通过多棵树投票，单棵树的噪声影响被削弱
2. 高准确率：在很多实际问题中表现很好
3. 可以处理高维特征：特征多也能稳定工作
4. 可以评估特征重要性：比如金融风控中评估哪些指标最关键

#### 缺点
1. 模型复杂：不像单颗决策树容易可视化
2. 训练和预测慢：尤其特征和树很多时
3. 对噪声数据敏感（少量异常点影响不大，但数据质量差会有影响）

## 训练集和验证集

### 训练集（Training Set）

#### 定义
训练集是用于 **训练模型** 的数据。模型会根据训练集的数据来 **学习规律**，调整内部参数（如逻辑回归的系数、随机森林的树结构）。

#### 作用
* 拟合模型
* 找到特征和目标变量之间的关系
* 最小化训练误差（loss）

#### 注意点
* 如果只关注训练集的表现，模型可能 **过拟合**，在新数据上表现差。

### 验证集（Validation Set）

#### 定义
验证集是从训练数据中独立划分出来的一部分数据，不参与模型训练，用来 **调参和评估模型效果**。

#### 作用
1. 评估模型性能（accuracy、AUC、MSE等）
2. 调整超参数（如树的数量、正则化系数等）
3. 防止过拟合：如果模型在训练集表现很好，但在验证集表现差，说明模型可能过拟合

### 举例
假设你做二分类模型（贷款违约预测）：
| 数据集     | 用途              |
| ------- | --------------- |
| 训练集     | 学习客户特征与违约的关系    |
| 验证集     | 调整模型参数、测试泛化能力   |
| 测试集（可选） | 最终评估模型在全新数据上的表现 |

### 使用方法
1. 划分数据
    * 通常训练集 : 验证集 = 70% : 30% 或 80% : 20%
    * 也可以使用 **交叉验证**（K-Fold）替代单独验证集
2. 训练模型
    * 用训练集拟合模型，学习参数
3. 验证模型
    * 用验证集计算指标，调整模型超参数
    * 重复训练-验证直到指标满意
4. 最终测试（如果有测试集）
    * 测试集仅用于 模型最终性能评估，不参与任何训练或调参

## ROC 曲线及曲线下面积（AUC）

### ROC 曲线（Receiver Operating Characteristic）
ROC曲线是用来评估二分类模型性能的图形工具。
* 横轴：假正例率（FPR）
{{< katex display=true >}}
FPR = \frac{假阳性数（FP）}{实际负样本总数（FP+TN）}
{{< /katex >}}
* 纵轴：真正例率（TPR）/召回率
{{< katex display=true >}}
TPR = \frac{真正例数（TP）}{实际正样本总数（TP+FN）}
{{< /katex >}}
> 每个点对应一个 **分类阈值**（threshold）。  
> ROC 曲线显示模型在不同阈值下的 **灵敏度 vs 假警报率** 的平衡。

### AUC（Area Under Curve）
AUC = ROC 曲线下面积
* 数值范围：0 ~ 1
* 含义：模型随机挑选一个正样本和一个负样本，正样本得分高于负样本的概率
* 解释：
    * 0.5：模型没有区分能力，相当于随机猜
    * 0.7~0.8：一般水平
    * 0.8~0.9：良好
    * \>0.9：优秀

### 举例说明
假设你做信用卡违约预测：
| 样本 | 真实标签 | 模型预测概率 |
| -- | ---- | ------ |
| A  | 违约   | 0.9    |
| B  | 未违约  | 0.4    |
| C  | 违约   | 0.7    |
| D  | 未违约  | 0.3    |
* ROC 曲线会根据不同阈值（如 0.5、0.6、0.7）绘制 TPR/FPR
* 曲线下面积 AUC 越大 → 模型区分能力越强

## Brier Score

#### 定义
Brier Score 用于衡量一个模型预测的**概率**与**真实结果**之间的差异。
数学定义为：
{{< katex display=true >}}
\text{Brier Score} = \frac{1}{N} \sum_{i=1}^{N} ({p}_i - {y}_i)^2
{{< /katex >}}
其中：
* {{< katex >}}N{{< /katex >}}：样本总数
* {{< katex >}}p_i{{< /katex >}}：模型预测样本为正类（例如 1）的概率
* {{< katex >}}y_i{{< /katex >}}：样本的真实标签（0 或 1）

#### 直观理解
1. Brier Score 就是预测概率与真实标签之间的 **均方误差** (MSE)
2. 它衡量的是模型的 **校准度**（calibration） 和 **准确性**（accuracy）

#### 取值范围与含义
* 取值范围：
{{< katex display=true >}}0 \leq \text{Brier Score} \leq 1{{< /katex >}}
* 越小越好：
    * 0 表示完美预测（预测 1 的都是真的 1，预测 0 的都是真的 0）
    * 0.25 左右通常对应“随便猜测”的水平（例如预测概率都为 0.5）
    * 1 表示非常差的预测（完全反向）

#### 举例
假设模型预测如下：
| 样本 | 实际标签 (y_i) | 预测概率 (p_i) | 误差平方 ((p_i - y_i)^2) |
| -- | ---------- | ---------- | -------------------- |
| 1  | 1          | 0.9        | (0.9 - 1)² = 0.01    |
| 2  | 0          | 0.2        | (0.2 - 0)² = 0.04    |
| 3  | 1          | 0.7        | (0.7 - 1)² = 0.09    |
| 4  | 0          | 0.3        | (0.3 - 0)² = 0.09    |

平均：
{{< katex display=true >}}\text{Brier Score} = \frac{0.01+0.04+0.09+0.09}{4} = 0.0575{{< /katex >}}
→ Brier Score = 0.0575（说明模型相当不错）

#### 与其他指标的区别
| 指标              | 关注点        | 越大/越小越好 | 是否考虑概率置信度     |
| --------------- | ---------- | ------- | ------------- |
| **Accuracy**    | 分类对错       | 越大越好    | ❌ 不考虑概率       |
| **AUC**         | 排序能力       | 越大越好    | ✅ 部分考虑        |
| **Log Loss**    | 概率预测       | 越小越好    | ✅ 强调高置信错误惩罚更大 |
| **Brier Score** | 概率校准 + 准确度 | 越小越好    | ✅ 更平滑的误差衡量    |

#### 拆解
Brier Score 还能分解为三部分（Brier decomposition）：

{{< katex display=true >}}BS = \text{Uncertainty} − \text{Resolution} + \text{Reliability}{{< /katex >}}

* Uncertainty：数据集本身的不确定性（例如正负样本比例）
* Resolution：模型区分不同情况的能力
* Reliability：模型预测概率与真实频率之间的匹配程度（校准性）

→ 所以 Brier Score 不仅看准确性，还能体现模型的概率是否可靠。

## KS 统计量（Kolmogorov-Smirnov Statistic）和 PSI (Population Stability Index)

### KS（Kolmogorov-Smirnov Statistic）
* 全称：Kolmogorov-Smirnov 统计量
* 用途：衡量 **两个分布的差异**，在信用评分或分类模型中，用于判断模型对正负样本的区分能力。
* 计算方式：
    1. 将样本按照模型预测分数排序
    2. 计算累计好坏样本分布（CDF，Cumulative Distribution Function）
    3. KS = 最大的正样本累计百分比与负样本累计百分比的差
* 解释：
    * KS 值越大 → 模型区分能力越强
    * 常见区间：
        * KS < 0.2 → 模型弱
        * KS ≈ 0.4–0.5 → 模型优秀
* 图形化理解：可以画两条曲线（好客户 vs 坏客户累计分布），最大垂直距离就是 KS 值。

### PSI（Population Stability Index）
* 全称：Population Stability Index
* 用途：衡量 **样本分布随时间的变化**，主要用来检测模型是否发生了“数据漂移”，即评分分布是否稳定。
* 计算方式：
    1. 将评分分成若干区间（bins）
    2. 统计每个区间中基准期（expected）和当前期（actual）的样本比例
    3. PSI 公式：
{{< katex display=true >}}
PSI = \sum(\text{actual\_pct} - \text{expected\_pct}) \times \ln(\frac{\text{actual\_pct}}{\text{expected\_pct}})
{{< /katex >}}
* 解释：
    * PSI < 0.1 → 样本稳定
    * 0.1 ≤ PSI < 0.25 → 轻微变化
    * PSI ≥ 0.25 → 样本分布发生较大变化，模型可能需要重新校准

## 风险评分卡（Scorecard）

对于逻辑回归模型，通常将预测的违约概率转化为一个易于理解的风险评分。
{{< katex display=true >}}
\text{Score} = A - B \cdot \ln(\frac{P(\text{Default})}{1 - P(\text{Default})})
{{< /katex >}}
* {{< katex >}}P(\text{Default}){{< /katex >}} 是模型预测的违约概率。
* {{< katex >}}A{{< /katex >}} 和 {{< katex >}}B{{< /katex >}} 是缩放参数，通常由业务部门设定（例如：设定 {{< katex >}}B{{< /katex >}} 使得每 {{< katex >}}X{{< /katex >}} 分对应 {{< katex >}}\text{Odds}{{< /katex >}} (好坏比) 翻倍）。
* 切点设定 (Cut-off)： 确定一个分数阈值，低于该阈值拒绝申请。

## Python 代码实现骨架
以下代码展示了使用 Python 的 `pandas`, `numpy`, `scikit-learn` 库进行信用风险模型构建的核心步骤。
```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import roc_auc_score, confusion_matrix
from sklearn.preprocessing import StandardScaler
from imblearn.over_sampling import SMOTE # 用于处理不平衡数据

# 假设数据已经加载，且'target'是目标变量 (0:好客户, 1:坏客户)
# =========================================================================
# 1. 模拟数据加载（实际中是读取CSV或数据库）
# =========================================================================
np.random.seed(42)
data_size = 10000
df = pd.DataFrame({
    'income': np.random.lognormal(mean=9, sigma=0.5, size=data_size),
    'loan_amount': np.random.normal(loc=15000, scale=8000, size=data_size),
    'age': np.random.randint(20, 60, size=data_size),
    'utilization': np.random.beta(a=5, b=15, size=data_size),
    'credit_history_len': np.random.gamma(shape=5, scale=1, size=data_size)
})

# 模拟目标变量 (违约率假设为 5%)
df['target'] = (np.random.rand(data_size) < 0.05 + 0.1 * df['utilization']).astype(int)
# 修正，确保有足够的坏样本
df.loc[df['utilization'] > 0.8, 'target'] = 1
print(f"原始坏样本比例: {df['target'].mean():.4f}")

# =========================================================================
# 2. 数据准备与特征工程 (简化版)
# =========================================================================
X = df.drop('target', axis=1)
y = df['target']

# 简单的特征处理：创建债务收入比
X['dti'] = X['loan_amount'] / X['income']
X = X.drop(['loan_amount', 'income'], axis=1) # 移除原始变量

# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42, stratify=y)

# 标准化特征（有助于逻辑回归收敛）
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)


# =========================================================================
# 3. 样本不均衡处理 (使用 SMOTE 过采样)
# =========================================================================
smote = SMOTE(random_state=42)
X_train_smote, y_train_smote = smote.fit_resample(X_train_scaled, y_train)
print(f"SMOTE 后坏样本比例: {y_train_smote.mean():.4f}")


# =========================================================================
# 4. 模型选择与训练 (逻辑回归)
# =========================================================================
log_reg = LogisticRegression(solver='liblinear', C=1.0, random_state=42)
log_reg.fit(X_train_smote, y_train_smote)


# =========================================================================
# 5. 模型评估
# =========================================================================

# 预测测试集概率
y_pred_proba = log_reg.predict_proba(X_test_scaled)[:, 1]

# 计算 AUC (Area Under the Curve)
auc = roc_auc_score(y_test, y_pred_proba)
print(f"\n模型评估结果:")
print(f"测试集 AUC: {auc:.4f}")

# 转化为 Gini 系数
gini = 2 * auc - 1
print(f"测试集 Gini: {gini:.4f}")

# =========================================================================
# 6. 转化为风险评分卡 (Scorecard Transformation)
# =========================================================================

# 逻辑回归系数
coefficients = log_reg.coef_[0]
features = X_train.columns

print("\n模型系数 (用于评分卡计算):")
for feature, coef in zip(features, coefficients):
    print(f"  {feature}: {coef:.4f}")

# 假设设定：
# Point-to-Double-Odds (PDO): 50 分
# Base Odds (好坏比): 20:1
# Base Score: 600

# 计算 B 值 (Scaling Factor)
P = 50 # PDO
theta0 = np.log(2) # log(2)
B = P / theta0

# 计算 A 值 (Offset)
Base_Odds = 20
A = 600 + B * np.log(Base_Odds)

print(f"\n评分卡参数: A={A:.2f}, B={B:.2f}")

def calculate_score(proba):
    """根据概率计算评分卡分数"""
    odds = (1 - proba) / proba  # 好坏比 (Odds)
    score = A - B * np.log(1 / odds) # ln(Odds) = ln((1-P)/P)
    return score

# 计算测试集平均分数
test_scores = calculate_score(y_pred_proba)
print(f"测试集平均评分: {test_scores.mean():.2f}")

# 找出拒绝/接受的切点 (假设分数低于 550 为拒绝)
cut_off_score = 550
bad_count_at_cutoff = y_test[test_scores < cut_off_score].sum()
total_count_at_cutoff = (test_scores < cut_off_score).sum()
print(f"拒绝分数线: {cut_off_score}")
print(f"被拒绝客户数: {total_count_at_cutoff}")
if total_count_at_cutoff > 0:
    print(f"被拒绝客户中的坏客户比例 (Bad Rate): {bad_count_at_cutoff / total_count_at_cutoff:.2%}")
```

## 附录

### 逻辑回归

#### 逻辑回归模型公式（二分类）：
{{< katex display=true >}}
\text{logit}(p) = \ln\frac{p}{1-p} = \beta_0 + \beta_1x_1 + \beta_2x_2 + ... + \beta_nx_n
{{< /katex >}}

* {{< katex >}}p{{< /katex >}}是事件发生的概率（比如贷款违约）。
* {{< katex >}}x_i{{< /katex >}}是输入特征。
* {{< katex >}}\beta_i{{< /katex >}}是回归系数。

#### 类别变量的挑战
| 颜色 |
| -- |
| 红色 |
| 绿色 |
| 蓝色 |

如果直接用 1、2、3 编码：
* 逻辑回归会认为“红色 < 绿色 < 蓝色”有数值大小意义，这是错误的假设。
* 会导致模型误差或偏差。

如果用 One-Hot Encoding：
* 每个类别是一个二进制变量，逻辑回归可以处理。
* 但是类别太多时会产生大量虚拟变量，影响模型复杂度。

#### WoE 转换的作用

1. 将每个类别替换为一个 连续数值：
{{< katex display=true >}}
WoE_i = \ln\frac{P(好)}{P(坏)}
{{< /katex >}}
2. 这个连续值本质上是类别对事件发生概率的 “信息量”。
3. 好处：
* 逻辑回归用连续变量更自然，满足“线性关系”假设：WoE 值与 logit(p) 趋势上单调。
* 特征变得有序、可度量，不再是离散且无顺序的类别。

#### 举例说明
假设贷款违约数据中“年龄分箱” WoE：
| 年龄区间  | WoE  |
| ----- | ---- |
| 20-30 | -0.8 |
| 31-40 | -0.2 |
| 41-50 | 0.3  |
| 51-60 | 0.7  |

逻辑回归用这个连续 WoE 值：
{{< katex display=true >}}
\text{logit}(p) = \beta_0 + \beta_1 \times WoE
{{< /katex >}}

* WoE 越大 → 越倾向“坏客户”，模型可以直接拟合。
* 连续值 → 线性关系更容易建模，不需要多维虚拟变量。

### 利用验证集调整参数
