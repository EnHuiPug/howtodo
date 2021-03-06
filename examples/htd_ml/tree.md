# Machine Learning



## 决策树

流程：特征选择 $\Rightarrow$ 决策树生成 $\Rightarrow$ 决策树剪枝

利用信息增益、信息增益比这类指标作为指导筛选特征

期望叶节点的成员有更高的纯度，再决定属性划分时就有不同的划分标准。

- 信息增益 information gain （ID3）
- 信息增益率 gain ratio （C4.5）
- 基尼指数 Gini Index（CART）

以上指标可以用来度量分割后的成员的纯度或不确定性(熵)



一些计算指标

熵 $H(X)=-\sum_{i=0}^{n}{p(X=x_i)logp(X=x_i)}$

条件熵$H(Y|X)=\sum_{i=1}^{n}p(X=x_i)H(Y|X=x_i)$

对于一个数据集合D来说，样本中有k个类别。每个类别的经验概率是$\frac{|C_k|}{|D|}$, $|C_k|$ 代表k类的样本个数 ，$|D|$ 代表数据集合的样本总数。

对于数据集合D的经验熵就是：

$H(D)=-\sum_{k=1}^{K}\frac{|C_k|}{|D|}log(\frac{|C_k|}{|D|})$

- **ID3**

  特征选择标准

  - 假设取特征A作为划分特征。可以对数据集合D求(决策树学习中的信息增益等价于数据集中类与特征的互信息)

  - $g(D,A)=H(D)-H(D|A)$

  - 数据集中有K个类别，特征A有$\{a_1, a_2, ..., a_n\}$种取值，则可以根据A特征的取值将数据进一步划分为$\{D_1,D_2,...,D_n\}$，在$D_i$中属于$C_k$的样本集为$D_{ik}$，则有

  - $H(D|A)=\sum_{i=1}^{n}\frac{|D_i|}{|D|}H(Di|A=a_i)=-\sum_{i=1}^{n}\frac{|Di|}{|D|}\sum_{k=1}^{K}\frac{|D_{ik}|}{|D_i|}log(\frac{|D_{ik}|}{|D_i|})$

  - 对于所有的特征都可以计算个$g(D|A)$，选个信息增益最大的特征作为当前步骤的划分特征

  生成流程：

  - 在决策树各个节点上利用信息增益准则选择特征，递归的构建决策树。具体是：从根节点开始，对节点计算所有可能特征的信息增益，选择信息增益最大的特征作为该节点的特征，由该特征的不同取值建立子节点，在对子节点递归调用以上过程，直到信息增益很小或者没有特征可用停止。

- **C4.5**

  特征选择标准

  - $g_R(D,A)=\frac{g(D,A)}{H_A(D)}$

  - 这个算法算是对ID3的优化。再考虑用特征A作为分割特征的同时，还要考虑A取值的情况。$H_A(D)=-\sum_{i=0}^{n}\frac{|D_i|}{|D|}log(\frac{|D_i|}{|D|})$ , $|D_i|$代表特征A取$a_i$的样本数。

  生成流程：

  - 和ID3类似

过拟合的问题：上述两种算法对于训练集可能表现不错，但泛化个能不好。决策树过于复杂，很适配训练集的结果。

这个时候需要对决策树进行简化，**剪枝（pruning）**

最小化决策树的loss function

loss怎么定义？

假设决策树T有$|T|$个叶节点，每个叶节点t又有$N_t$个样本，其中$N_{tk}$为该叶节点上属于k类的样本数。$k=1,2,...,K$，定义$H_t(T)$为叶节点t上的经验熵，$\alpha\geq0$，则决策树的loss可以定义成：

$C_{\alpha}(T)=\sum_{t=i}^{|T|}N_tH_t(T)+\alpha|T|$

经验熵： $H_t(T)=-\sum_{k=1}^{K}\frac{N_{tk}}{N_t}log{\frac{N_{tk}}{N_t}}$

令：$C(T)=\sum_{t=i}^{|T|}N_tH_t(T)=-\sum_{t=i}^{|T|}\sum_{k=1}^{K}N_{tk}log{\frac{N_{tk}}{N_t}}$

损失函数可以化简为： $C_{\alpha}(T)=C(T)+\alpha|T|$

$C(T)$：可以表示模型对训练数据的拟合效果，越小越好，但是模型的复杂度会升高。

$\alpha|T|$：代表模型的复杂度，叶节点分的越细，越能过拟合训练数据，复杂度就会升高。

剪枝就是在$\alpha$确定的情况下，选择损失函数最小的模型。

- 剪枝过程：
  - 输入：生成的决策树T和$\alpha$；输出：剪枝后的子树$T_{\alpha}$
    1. 计算每个节点的经验熵
    2. 递归的从叶节点向上回溯；设原来树为$T$，子节点归并到上一父节点后的树是$T^{'}$。如果这两棵树的损失函数有 $C_{\alpha}(T^{'})\leq C_{\alpha}(T)$，则进行剪枝，退回到父节点。
    3. 重复2，直到不能进行下去为止。

- **CART**
  - 思路：对$P(Y|X)$建模，每个节点对特征做二分（选取特征A的某个取值$a_i$，等价于递归地二分每个特征，将数据空间划分为有限个单元，在这些单元上确定预测的概率分布。
  - 步骤：
    1. 生成：训练集生产，生成的树要尽量大（什么叫大？怎么保证大？均分某个特征的空间，保证左右两子节点样本数均衡？）
    2. 剪枝：用验证集数据进行剪枝，保证损失函数最小。
  - CART生成
    - 筛选特征准则，对于回归树用Square Error指标，对于分类树用Gini index
      - 回归树：rv： X， Y， Y连续。对于训练集$D=\{(x_1,y_1),(x_2,y_2),...,(x_N,y_N)\}$

$Gini(p)=\sum_{k=1}^{K}p_k(1-p_k)=1-\sum_{k=1}^{K}p_k^2$

- $p_k$ 样本属于k类别的概率， $(1-p_k)$ 样本不是k类的概率
- 这个经验概率$p_k$可以用频率来计算，这样就有$Gini(D)=1-\sum_{k=1}^{K}(\frac{|C_k|}{|D|})^2$
- 由于Gini考虑的是对给定特征给定样本是否分类正确，即2项分类。所以如果有一个特征A，有$a_1,a_2,a_3$，三种取值。所以有三个划分选择。$Gini_{index}(D,A)=\sum_{i=1}^{3}\frac{|D_{a_i}|}{|D|}Gini({D_{a_i}})$



## Boosting

- 提升方法代表**AdaBoost**
- **Boosting Tree**
- **Gradient Boosting Decision Tree**
- **LightGBM**
- **XGBoost**
- **CatBoost**

